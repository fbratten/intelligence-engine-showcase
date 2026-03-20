# Intelligence Engine

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Python 3.12+](https://img.shields.io/badge/python-3.12+-blue.svg)
![Version](https://img.shields.io/badge/version-0.21.0-green.svg)

> Domain-agnostic intelligence engine with schema-driven knowledge graphs.

Build knowledge graphs from any structured domain using YAML schema definitions, then search, explore, and analyze them through hybrid search, a web UI, or an MCP server for AI assistants.

**Key Stats:** 1261+ tests | 15 MCP tools | 33 REST endpoints | 8 languages | 2 domains

---

## Architecture

```
Domain Schema     Extractors         Knowledge Graph     Hybrid Search       MCP / Web UI
  (YAML)            |                    |                    |                  |
  code.yaml    +--> Tree-sitter  --+--> KuzuDB          +--> BM25 (0.35)  +--> 15 MCP tools
  archaeology  |    (8 languages)  |    Entity_code      |    Semantic      |    (FastMCP)
  your-domain  |                   |    Entity_archae..  |    (0.40)        |
  ...          +--> Custom YAML  --+    Rel_code_*       |    Graph (0.25)  +--> 33 REST
               |    extractors     |    Rel_archae.._*   |                  |    endpoints
               |                   |                     |    3-way RRF     |    (FastAPI)
               +-------------------+    NetworkX         |    fusion        |
                                        (fallback)       +------------------+--> React 18
                                                                            |    Sigma.js
                                        LanceDB                            |    graph explorer
                                        (384-dim vectors)                  +----
```

Each domain is defined by a single YAML file that specifies entity types, relationship types, entity properties, search profiles, display configuration, and health checks. The engine creates domain-scoped database tables (`Entity_<domain>`, `Rel_<domain>_*`) and routes all operations through the `(project, domain)` key.

---

## Feature Highlights

- **Domain Generalization** -- Define any knowledge domain via a YAML schema. The engine handles table creation, indexing, search, and visualization automatically.
- **Archaeology MVP** -- First non-code domain: finds, sites, periods, materials, water bodies. Validates the entire domain-agnostic architecture.
- **Hybrid Search** -- 3-way Reciprocal Rank Fusion combining BM25 keyword search, semantic vector search (all-MiniLM-L6-v2), and graph context expansion.
- **Graph Visualization** -- React 18 + Sigma.js 3 force-directed graph explorer with filtering, clustering, and interactive entity detail.
- **AI Integration** -- LLM-powered entity summaries and Q&A (Claude, OpenAI, Gemini, Ollama). Persistent AI data survives re-indexing.
- **MCP Server** -- 15 tools exposing the full engine to AI coding assistants via the Model Context Protocol.
- **Multi-Project** -- Index and search across 100+ projects. Shared database mode enables cross-project Cypher queries and global analysis.

---

## Domains

### Code Intelligence (built-in)

Parses 8 languages via Tree-sitter:

| Language | Entity Types |
|----------|-------------|
| Python | function, class, method, module, variable |
| JavaScript | function, class, method, module, variable |
| TypeScript/TSX | function, class, method, module, variable, interface |
| Java | function, class, method, module, interface |
| Go | function, class, method, module, interface |
| HTML | component, template, form, section |
| CSS | selector, css_variable, keyframe, media_query |

Relationships: `CALLS`, `IMPORTS`, `EXTENDS`, `DEFINES`, `METHOD_OF`, `LINKS_STYLESHEET`, `REFERENCES_SCRIPT`, `USES_VARIABLE`

### Archaeology (first non-code domain)

Custom YAML/JSON extractor for archaeological data:

| Entity Type | Category | Example |
|-------------|----------|---------|
| find | artifact | Bronze axe, pottery shard |
| site | location | Burial mound, settlement |
| period | temporal | Bronze Age, Iron Age |
| material | classification | Bronze, flint, ceramic |
| water_body | geography | Lake, river |

Relationships: `FOUND_AT`, `DATED_TO`, `MADE_OF`, `NEAR_WATER`, `ASSOCIATED_WITH`

### Adding a New Domain

1. Create `config/domains/your-domain.yaml` defining entity types, relationships, properties, and search profiles
2. Write an extractor (or use Tree-sitter for code-like domains)
3. Index your data -- the engine creates the necessary database tables automatically

See [docs/architecture.md](docs/architecture) for the full domain schema specification.

---

## Status

**All phases complete + Domain Generalization.** v0.21.0, 1261+ tests passing.

| Phase | Feature | Status |
|-------|---------|--------|
| 1 | AST Parsing (Tree-sitter) | Complete |
| 2 | Knowledge Graph (NetworkX + KuzuDB) | Complete |
| 3 | Hybrid Search (BM25 + Graph RRF) | Complete |
| 4 | MCP Server (15 tools via FastMCP) | Complete |
| 5 | Multi-Project (registry, cross-project search) | Complete |
| 6 | Semantic Embeddings (sentence-transformers + LanceDB) | Complete |
| 6.5 | KuzuDB Migration (dual-backend, Cypher queries) | Complete |
| 7 | Visual UI (React + Sigma.js graph explorer) | Complete |
| 8 | Multi-Language Support (Python, JS, TS/TSX, Java, Go, HTML, CSS) | Complete |
| 9 | Incremental Indexing (git diff + hash fallback) | Complete |
| 10 | Performance Dashboard (timeline, phases, health, compare) | Complete |
| 11 | AI-Powered Summaries (Claude, OpenAI, Gemini, Ollama) | Complete |
| 12 | Graph Clustering (Louvain community detection) | Complete |
| -- | Batch Summaries, Code Quality Metrics, UI Polish | Complete |
| -- | AI Q&A (free-form questions, template prompts, history) | Complete |
| -- | Shared DB Architecture (multi-tenant, migration tool) | Complete |
| -- | Cross-Project Search + Global Graph Analysis | Complete |
| -- | AI Overlay Preservation (data survives re-index) | Complete |
| -- | Read-Only Serving + Input Sanitization | Complete |
| -- | AI Memory Tab (unified memory browser, export) | Complete |
| -- | HTML & CSS Language Support (8 new entity types) | Complete |
| -- | **Domain Generalization** (schema-driven, YAML config) | Complete |
| -- | **Archaeology MVP** (first non-code domain) | Complete |

---

## Quick Start

```bash
# Clone and set up
git clone <repo-url> intelligence-engine
cd intelligence-engine
python3 -m venv .venv
source .venv/bin/activate
uv pip install -e ".[dev]"

# Index a project (code domain, auto-detected)
python -m intelligence_engine index /path/to/your/project

# Search
python -m intelligence_engine search data/myproject/parse.json "find authentication"

# Start the web UI
python -m intelligence_engine serve --port 8420 &
cd src/intelligence_engine/web/frontend && npm install && npm run dev

# Open the URL shown by Vite in your browser
```

### Storage Modes

```bash
# Per-project (default): each project gets its own databases
python -m intelligence_engine index /path/to/project

# Shared mode: all projects in a single database (enables cross-project queries)
# Set storage.mode: shared in config/config.yaml, then:
python -m intelligence_engine migrate --to-shared
python -m intelligence_engine migrate --verify-only
```

---

## Web UI

React 18 + Sigma.js 3 graph explorer with 6 dashboard tabs:

- **Interactive graph** -- Force-directed visualization (ForceAtlas2) with node/edge type filtering
- **Hybrid search** -- BM25, semantic, graph, or combined search with dropdown mode picker
- **Entity detail** -- Source code preview, complexity badges, AI summaries, Q&A history
- **Cypher console** -- Direct KuzuDB queries with sticky headers and monospace output
- **Performance dashboard** -- Index timeline, per-phase timing, health snapshots, project comparison, quality metrics
- **AI Memory browser** -- Unified view of all AI-generated data with filters, search, and CSV/JSON export
- **Graph clustering** -- Louvain community detection, color by type or cluster
- **Code quality** -- Composite scores, complexity histograms, coupling analysis
- **AI Q&A** -- Free-form questions about any entity, with template prompts and persistent history
- **LLM settings** -- Multi-provider credential management (Claude, OpenAI, Gemini, Ollama)
- **Batch operations** -- Project indexing, batch summarization with progress tracking

---

## MCP Server

15 tools for AI assistants via the Model Context Protocol:

| Tool | Purpose |
|------|---------|
| `ie_index` | Index a project into the knowledge graph |
| `ie_query` | Search within a single project |
| `ie_search_all` | Cross-project semantic search |
| `ie_context` | Entity context (callers, callees, blast radius) |
| `ie_detect_changes` | Pre-change risk assessment via git diff |
| `ie_cypher` | Read-only Cypher queries on KuzuDB |
| `ie_wiki` | Generate documentation from the graph |
| `ie_status` | List all indexed projects |
| `ie_health` | Structural health (dead code, cycles, hubs) |
| `ie_quality` | Code quality metrics (complexity, docs, coupling) |
| `ie_summarize` | AI-powered entity summary (single) |
| `ie_batch_summarize` | AI-powered summaries (project-wide) |
| `ie_global_analysis` | Cross-project clustering + health (shared mode) |
| `ie_memory` | Unified AI memory browser |

The server includes self-describing resources (`ie://schema`, `ie://cypher-templates`, `ie://guide`) and pre-built prompt workflows (code review, capability audit, change risk assessment).

---

## Technology Stack

| Component | Technology |
|-----------|-----------|
| **Language** | Python 3.12 (backend), TypeScript (frontend) |
| **AST Parser** | Tree-sitter (8 language grammars) |
| **Graph DB** | KuzuDB (default) + NetworkX (fallback) |
| **Vector Store** | LanceDB (all-MiniLM-L6-v2, 384-dim) |
| **Keyword Search** | BM25 (rank_bm25) |
| **MCP Server** | FastMCP |
| **Web Backend** | FastAPI (33 REST endpoints) |
| **Web Frontend** | React 18 + Sigma.js 3 + Vite 6 + Tailwind CSS v4 |
| **AI Providers** | Claude, OpenAI, Gemini, Ollama |
| **Build** | Hatchling (Python), Vite (frontend) |
| **Package Mgmt** | uv (Python), npm (frontend) |
| **Testing** | pytest (1261+ tests) |

---

## Documentation

- **[Architecture Deep-Dive](docs/architecture)** -- Domain schema system, pipeline, storage modes
- **[Technical Overview](docs/technical-overview)** -- Parser, graph, search, AI, MCP, REST details
- **[Architectural Decisions](docs/decisions)** -- Key design choices and rationale
- **[Project Context](docs/project-context)** -- What it does, why it exists, status

---

## Interactive Demos

- **[Which Search Strategy?](demos/search-picker/)** -- Choose the right search approach for your query

---

## License

[MIT](LICENSE)
