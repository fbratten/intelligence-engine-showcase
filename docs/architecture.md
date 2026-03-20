---
layout: default
title: Architecture Deep-Dive
---

# Architecture -- Intelligence Engine

## Overview

Intelligence Engine is a domain-agnostic knowledge graph platform. It ingests structured data (source code, archaeological records, or any domain defined by a YAML schema), builds a typed knowledge graph, and exposes it through hybrid search, a REST API, an MCP server, and a web UI.

The core abstraction is the **domain schema** -- a single YAML file that defines everything the engine needs to handle a new knowledge domain without code changes.

---

## High-Level Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                      INTELLIGENCE ENGINE                            │
│                                                                     │
│  ┌──────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │ Domain       │    │ Extractors       │    │ Knowledge Graph  │  │
│  │ Schema       │───>│                  │───>│                  │  │
│  │              │    │ Tree-sitter      │    │ KuzuDB           │  │
│  │ code.yaml    │    │  (8 languages)   │    │  Entity_code     │  │
│  │ archaeology  │    │                  │    │  Entity_archae.. │  │
│  │ your-domain  │    │ Custom YAML/JSON │    │  Rel_code_*      │  │
│  │              │    │  extractors      │    │  Rel_archae.._*  │  │
│  └──────────────┘    └──────────────────┘    └────────┬─────────┘  │
│                                                       │             │
│  ┌──────────────┐    ┌──────────────────┐             │             │
│  │ Embedding    │    │ Search Engine    │<────────────┘             │
│  │ Pipeline     │───>│                  │                           │
│  │              │    │ BM25 (0.35)      │    ┌──────────────────┐  │
│  │ MiniLM-L6   │    │ Semantic (0.40)  │───>│ API Layer        │  │
│  │ LanceDB     │    │ Graph (0.25)     │    │                  │  │
│  │ 384-dim      │    │                  │    │ MCP: 15 tools    │  │
│  └──────────────┘    │ 3-way RRF       │    │ REST: 33 endpts  │  │
│                      └──────────────────┘    │ Web: React/Sigma │  │
│  ┌──────────────┐                            │                  │  │
│  │ Change       │    ┌──────────────────┐    │ AI: 4 LLM       │  │
│  │ Detector     │    │ Quality Engine   │    │   providers      │  │
│  │ git diff +   │    │ Complexity, docs │    └──────────────────┘  │
│  │ hash fallback│    │ coupling, scores │                           │
│  └──────────────┘    └──────────────────┘                           │
│                                                                     │
│  Graph: KuzuDB (default) + NetworkX (fallback)                     │
│  Vectors: LanceDB (all-MiniLM-L6-v2, 384-dim)                     │
│  Incremental: git diff primary, SHA-256 hash fallback              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Domain Schema System

Each domain is defined by a YAML file in `config/domains/`. A schema specifies:

```yaml
domain: archaeology
version: 1
display_name: "Archaeology"

identity_rules:
  uniqueness: per_project
  id_pattern: "{project}::{source_ref}::{name}"

entity_types:
  - name: find
    label: Find
    color: "#D4A574"
    category: artifact
  - name: site
    label: Site
    color: "#4A90D9"
    category: location
  # ...

relationship_types:
  - name: FOUND_AT
    source_types: [find]
    target_types: [site]
    cardinality: many_to_one
    edge_properties:
      context: string
      depth: double
  # ...

entity_properties:
  - name: find_category
    type: string
    required: false
    description: "Category of find (axe, jewelry, pottery, etc.)"
  # ...

search_profiles:
  default:
    bm25_fields: [name, entity_type, description, region]
    bm25_weights:
      name: 3
      description: 2
    embedding_fields: [name, entity_type, description]

display:
  default_label_field: name
  detail_fields: [name, entity_type, find_category, region, ...]
  snippet_field: description

health_checks:
  callable_types: []
  dependency_edge: null
  skip_types: [external]

extractors:
  type: custom                                          # or "tree-sitter"
  module: intelligence_engine.extractors.archaeology_extractor

embedding:
  model: "all-MiniLM-L6-v2"
```

When the engine encounters a domain, it:

1. Loads the schema YAML
2. Creates `Entity_<domain>` and `Rel_<domain>_<type>` tables in KuzuDB
3. Routes indexing to the correct extractor (Tree-sitter or custom)
4. Configures search profiles for the domain's fields
5. Sets up display and health check rules

---

## Component Responsibilities

| Component | Responsibility |
|-----------|----------------|
| `src/parser/` | Tree-sitter AST parsing, 8-language entity extraction |
| `src/extractors/` | Custom domain extractors (archaeology, etc.) |
| `src/domain/` | Domain schema loading, validation, registry |
| `src/graph/` | Knowledge graph (KuzuDB + NetworkX dual-backend), Cypher queries |
| `src/search/` | BM25, semantic (LanceDB), graph search, 3-way RRF fusion |
| `src/llm/` | LLM provider abstraction (Claude/OpenAI/Gemini/Ollama), entity summarizer |
| `src/mcp/` | MCP server with 15 tools |
| `src/web/` | FastAPI backend (33 endpoints) + React/Sigma.js frontend |
| `src/registry.py` | Multi-project registry, cross-project search |
| `src/change_detector.py` | Incremental indexing: git diff + SHA-256 hash fallback |
| `src/indexer.py` | Shared pipeline (CLI/MCP/Web), mode=auto/full/incremental |
| `src/quality.py` | Code quality metrics (complexity, doc coverage, coupling) |
| `src/storage.py` | Storage mode detection (per-project / shared), path resolution |
| `src/migrate.py` | Migration tool: per-project <-> shared, verification |
| `src/ai_overlay.py` | AI data preservation across re-indexing |
| `src/memory_aggregator.py` | Unified memory records from graph + external sources |

---

## Storage Architecture

### Per-Project Mode (default)

Each project gets isolated databases:

```
data/
  myproject/
    parse.json          # Extracted entities and relationships
    kuzu_db/            # KuzuDB graph database
    lancedb/            # LanceDB vector store
    index_history.json  # Performance tracking
```

### Shared Mode (multi-tenant)

All projects share a single database with `project` column filtering:

```
data/
  _shared/
    kuzu_db/            # All projects in one graph
    lancedb/            # All embeddings in one vector store
  myproject/
    parse.json          # Still per-project (source data)
    index_history.json
```

Shared mode enables:
- Cross-project Cypher queries (find dependencies across projects)
- Single-query semantic search across all projects
- Global graph analysis (community detection across the full graph)

Switch between modes using `python -m intelligence_engine migrate`.

### Domain-Scoped Tables

KuzuDB tables are scoped by domain to prevent schema conflicts:

```
Entity_code             # 15 entity types, code-specific properties
Rel_code_CALLS          # Code relationship tables
Rel_code_IMPORTS
Rel_code_EXTENDS
...

Entity_archaeology      # 6 entity types, archaeology-specific properties
Rel_archaeology_FOUND_AT
Rel_archaeology_DATED_TO
...
```

This allows multiple domains to coexist in the same database without column or type collisions.

---

## Routing Key: (project, domain)

All operations in the engine are routed by the tuple `(project, domain)`:

- **Indexing:** Selects the correct extractor and target tables
- **Search:** Scopes results and applies domain-specific search profiles
- **API:** Every endpoint accepts `project` and `domain` parameters
- **MCP:** Tools route through the same key
- **Web UI:** Domain selector in the project dropdown

This design means a single project can contain entities from multiple domains (e.g., a project with both code and documentation entities).

---

## Adding a New Domain

1. **Define the schema** -- Create `config/domains/your-domain.yaml` with entity types, relationships, properties, search profiles, and display config
2. **Write an extractor** -- Either use `tree-sitter` (for code-like domains) or create a custom extractor module that reads your data format and returns entity/relationship dicts
3. **Index** -- Run the indexer with `--domain your-domain` and the engine creates all necessary tables
4. **Use** -- Search, query, visualize, and analyze through the same interfaces as any other domain

No core engine code needs to change. The schema YAML drives everything.
