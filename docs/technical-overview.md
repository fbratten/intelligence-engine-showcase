---
layout: default
title: Technical Overview
---

# Technical Overview -- Intelligence Engine

## Parsing Layer

### Tree-sitter (Code Domains)

The engine uses py-tree-sitter to parse source code into ASTs and extract typed entities and relationships. Each language has a dedicated extractor that shares a common base class.

**Supported languages:** Python, JavaScript, TypeScript, TSX, Java, Go, HTML, CSS

**Entity extraction per language:**

| Language | Entities Extracted |
|----------|-------------------|
| Python | functions, classes, methods, modules, variables |
| JavaScript | functions, classes, methods, modules, variables |
| TypeScript | functions, classes, methods, modules, variables, interfaces |
| TSX | functions, classes, methods, modules, variables, interfaces |
| Java | functions, classes, methods, modules, interfaces |
| Go | functions, classes (structs), methods, modules, interfaces |
| HTML | components, templates, forms, sections |
| CSS | selectors, css_variables, keyframes, media_queries |

**Relationships detected:**

| Relationship | Meaning | Languages |
|-------------|---------|-----------|
| `CALLS` | Function/method invocation | All code languages |
| `IMPORTS` | Module/symbol import | All code languages |
| `EXTENDS` | Class/interface inheritance | All code languages |
| `DEFINES` | Module defines entity | All code languages |
| `METHOD_OF` | Method belongs to class | All code languages |
| `LINKS_STYLESHEET` | HTML links CSS file | HTML |
| `REFERENCES_SCRIPT` | HTML references JS file | HTML |
| `USES_VARIABLE` | CSS selector uses variable | CSS |

**Per-language cyclomatic complexity** is calculated during extraction for functions and methods.

### Custom Extractors (Non-Code Domains)

For non-code domains (e.g., archaeology), custom extractors read YAML, JSON, or other data formats and produce the same entity/relationship structure. The extractor module is specified in the domain schema YAML:

```yaml
extractors:
  type: custom
  module: intelligence_engine.extractors.archaeology_extractor
```

---

## Knowledge Graph

### KuzuDB (Primary Backend)

KuzuDB is an embedded graph database with Cypher query support. The engine creates domain-scoped tables:

- **Node tables:** `Entity_<domain>` with base columns (id, project, domain, entity_type, name, source_ref, line_start, line_end, summary, etc.) plus domain-specific properties
- **Edge tables:** `Rel_<domain>_<type>` for each relationship type defined in the domain schema

Base entity columns (always present):

```
id, project, domain, entity_type, name, source_ref,
line_start, line_end, summary, summary_provider,
qa_history, schema_version, content_hash, provenance,
confidence, created_at, updated_at
```

Key Cypher notes:
- Use `n.entity_type` (not `n.type`)
- Use `n.complexity` (not `n.cc`)
- Use `label(r)` for edge type (not `type(r)`)
- All queries are read-only in serving mode (write operations blocked by validation layer)

### NetworkX (Fallback Backend)

NetworkX provides an in-memory graph backend for environments where KuzuDB is not available. Both backends implement the same `GraphStore` interface with project-filtered iteration.

---

## Search Engine

### 3-Way RRF Fusion

The hybrid search engine combines three retrieval strategies using Reciprocal Rank Fusion:

| Strategy | Weight | Implementation | Strengths |
|----------|--------|---------------|-----------|
| **BM25** | 0.35 | rank_bm25 | Exact keyword matching, function names, identifiers |
| **Semantic** | 0.40 | all-MiniLM-L6-v2 + LanceDB | Conceptual queries, natural language, synonyms |
| **Graph** | 0.25 | 2-hop context expansion | Structural relationships, callers/callees, imports |

**RRF formula:** `score = sum(1 / (k + rank_i))` across strategies, where `k = 60`.

### Search Profiles

Each domain schema defines search profiles that control which fields are searched and how they're weighted:

```yaml
search_profiles:
  default:
    bm25_fields: [name, docstring, code_snippet, file]
    bm25_weights:
      name: 3
      docstring: 2
      code_snippet: 1
      file: 1
    embedding_fields: [name, entity_type, docstring, file]
```

### Semantic Search

- **Model:** all-MiniLM-L6-v2 (384 dimensions, ~80MB, CPU-only)
- **Vector store:** LanceDB (embedded, incremental add/delete)
- **Cross-project search:** In shared mode, a single query searches all project embeddings
- **Incremental updates:** Only changed entities are re-embedded (delete + add)

---

## AI Features

### LLM-Powered Summaries

Four provider integrations with lazy imports and graceful degradation:

| Provider | Model Examples | Notes |
|----------|---------------|-------|
| **Anthropic** | Claude Sonnet, Opus | Primary provider |
| **OpenAI** | GPT-4, GPT-3.5 | OpenAI API |
| **Google** | Gemini Pro, Flash | Google AI API |
| **Ollama** | Any local model | Local inference, no API key needed |

Summaries are persisted to the knowledge graph and survive re-indexing through the AI overlay preservation system.

### AI Q&A

Free-form questions about any entity in the graph. Template prompts provide common starting points (e.g., "What does this function do?", "What are the side effects?"). Q&A history is persisted per entity.

### AI Overlay Preservation

When projects are re-indexed or migrated between storage modes, AI-generated data (summaries, Q&A history) is extracted before the rebuild and restored afterward. This prevents loss of accumulated AI insights.

### Memory Aggregation

A unified `MemoryRecord` format combines:
- Per-entity AI data from the knowledge graph (summaries, Q&A)
- Cross-session memory from external sources

Exposed through the REST API, MCP tool (`ie_memory`), and a dedicated browser tab.

---

## MCP Server

15 tools exposed via FastMCP for AI coding assistants:

```
ie_index            Index a project into the knowledge graph
ie_query            Search within a single project
ie_search_all       Cross-project semantic search
ie_context          Entity context (callers, callees, blast radius)
ie_detect_changes   Pre-change risk assessment via git diff
ie_cypher           Read-only Cypher queries on KuzuDB
ie_wiki             Generate documentation from the graph
ie_status           List all indexed projects
ie_health           Structural health (dead code, cycles, hubs)
ie_quality          Code quality metrics (complexity, docs, coupling)
ie_summarize        AI-powered entity summary (single)
ie_batch_summarize  AI-powered summaries (project-wide)
ie_global_analysis  Cross-project clustering + health (shared mode)
ie_memory           Unified AI memory browser
```

### Self-Description

The server includes:
- **Resources:** `ie://schema` (full graph schema), `ie://cypher-templates` (33 query templates), `ie://guide` (onboarding guide), `ie://rest-api` (endpoint reference)
- **Prompts:** Pre-built workflows for code review, understanding code, change risk assessment, capability audit, and AI data enrichment

---

## REST API

33 FastAPI endpoints organized by function:

| Category | Endpoints | Purpose |
|----------|-----------|---------|
| **Project** | `/api/projects`, `/api/index`, `/api/status` | Project management, indexing |
| **Search** | `/api/search`, `/api/search-all` | Single-project and cross-project search |
| **Graph** | `/api/graph`, `/api/context`, `/api/cypher` | Graph data, entity context, Cypher |
| **AI** | `/api/summarize`, `/api/ask`, `/api/batch-summarize` | LLM summaries, Q&A |
| **Quality** | `/api/quality`, `/api/health` | Code metrics, structural health |
| **Memory** | `/api/memory`, `/api/memory/stats`, `/api/memory/export` | AI memory browser |
| **Global** | `/api/global/clusters`, `/api/global/health` | Cross-project analysis |
| **Config** | `/api/llm/settings`, `/api/domains` | LLM credentials, domain listing |

All endpoints include input validation. Cypher queries are validated to block write operations in serving mode.

---

## Frontend

### Technology

- **React 18** with functional components and hooks
- **Sigma.js 3** for force-directed graph visualization (ForceAtlas2 layout)
- **Tailwind CSS v4** for styling
- **Vite 6** for development and building
- **highlight.js** for syntax highlighting in entity detail

### Dashboard Tabs

The performance dashboard has 6 tabs:

1. **Timeline** -- Index performance over time
2. **Phases** -- Per-phase timing breakdown
3. **Health** -- Structural health snapshots
4. **Compare** -- Side-by-side project comparison
5. **Quality** -- Code quality metrics (complexity histograms, coupling, doc coverage)
6. **AI Memory** -- Unified browser for all AI-generated data

### Key UI Features

- Node/edge type filtering with checkbox controls
- Depth-based graph exploration (1-hop, 2-hop, full)
- Color by entity type or Louvain community cluster
- Cypher console with direct KuzuDB query execution
- Entity detail panel with source code, complexity badges, AI summaries
- Q&A history panel (collapsible, with Cypher integration)
- AI data filter (show only nodes with AI-generated data)
- Custom dark-themed scrollbars

---

## Incremental Indexing

Two detection strategies:

1. **git diff** (primary) -- Compares against the last indexed commit to find changed files
2. **SHA-256 hash** (fallback) -- Computes file hashes and compares against stored values (for non-git directories)

Performance characteristics:
- Full index: ~30-60s for a medium project (depends on file count and language mix)
- Incremental: ~2-4s for small changes (graph + BM25 rebuilt, semantic truly incremental)
- The indexer records per-phase timing for performance tracking

---

## Concurrency and Locking

- **Serving mode:** KuzuDB opens in `read_only=True` mode, allowing unlimited concurrent readers
- **Indexing:** Requires exclusive write access -- the serving process must be stopped first
- **Batch re-indexing:** A CLI script handles sequential re-indexing of multiple projects
- **Input sanitization:** All Cypher queries pass through validation that blocks write operations (DELETE, CREATE, SET, DROP, MERGE, ALTER)

---

## Configuration

All tunable parameters live in `config/config.yaml`:

```yaml
parser:
  languages: [python, javascript, typescript, tsx, java, go, html, css]
  ignore_patterns: [node_modules, __pycache__, .git, ...]

storage:
  mode: per-project    # or "shared"
  data_dir: data/

search:
  bm25_weight: 0.35
  semantic_weight: 0.40
  graph_weight: 0.25

memory:
  enabled: true
```

Domain schemas are in `config/domains/<domain>.yaml`.
