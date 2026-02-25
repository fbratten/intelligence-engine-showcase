# Architecture — Intelligence Engine

## High-Level Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    INTELLIGENCE ENGINE                         │
│                                                               │
│  ┌──────────┐    ┌──────────────┐    ┌───────────────┐       │
│  │ Tree-    │───>│ Knowledge    │───>│ Hybrid Search │       │
│  │ sitter   │    │ Graph (DB)   │    │ Engine        │       │
│  │ Parser   │    │              │    │               │       │
│  │          │    │ Nodes:       │    │ - BM25        │       │
│  │ 6 langs  │    │  Functions   │    │ - Semantic    │       │
│  │ Py/JS/TS │    │  Classes     │    │ - Graph Walk  │       │
│  │ Java/Go  │    │  Interfaces  │    │               │       │
│  │          │    │  Modules     │    │ 3-way RRF     │       │
│  │          │    │ Edges:       │    │  Fusion       │       │
│  │          │    │  CALLS       │    │  Scoring      │       │
│  │          │    │  IMPORTS     │    │               │       │
│  │          │    │  EXTENDS     │    │ Results:      │       │
│  │          │    │  DEFINES     │    │  Ranked,      │       │
│  │          │    │  METHOD_OF   │    │  Cited,       │       │
│  │          │    │              │    │  Contextual   │       │
│  └──────────┘    └──────────────┘    └───────┬───────┘       │
│                                              │                │
│  ┌──────────┐    ┌──────────────┐            │                │
│  │ Embedding│    │ MCP Server   │<───────────┘                │
│  │ Pipeline │    │ (FastMCP)    │                              │
│  │ MiniLM   │    │ 13 tools     │    ┌──────────────────┐    │
│  │ LanceDB  │    │              │───>│ Claude Code      │    │
│  └──────────┘    └──────────────┘    │ Other Agents      │    │
│                                      └──────────────────┘    │
│  ┌──────────┐    ┌──────────────┐                            │
│  │ Change   │    │ Web UI       │                            │
│  │ Detector │    │ FastAPI +    │                            │
│  │ git diff │    │ React/Sigma  │                            │
│  │ + hashes │    │ Graph Viz    │                            │
│  └──────────┘    └──────────────┘                            │
│                                                               │
│  Graph DB: KuzuDB (default) + NetworkX (fallback)            │
│  Vector DB: LanceDB (all-MiniLM-L6-v2, 384-dim)             │
│  Incremental: git diff primary, hash fallback                │
│  Local only — WSL Ubuntu                                      │
└──────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

| Component | Phase | Responsibility |
|-----------|-------|----------------|
| `src/parser/` | 1, 8 | Tree-sitter AST parsing, 6-language entity extraction (Py/JS/TS/Java/Go) |
| `src/graph/` | 2, 6.5 | Knowledge graph (KuzuDB + NetworkX dual-backend), Cypher queries |
| `src/search/` | 3, 6 | BM25, semantic (all-MiniLM-L6-v2 + LanceDB), graph search, 3-way RRF fusion |
| `src/llm/` | 11 | LLM provider abstraction (Claude/OpenAI/Gemini/Ollama), credentials, entity summarizer |
| `src/mcp/` | 4, 11 | MCP server with 13 tools for Claude Code |
| `src/registry.py` | 5 | Multi-project registry, cross-project search |
| `src/web/` | 7 | FastAPI backend (12 endpoints) + React/Sigma.js graph explorer |
| `src/change_detector.py` | 9 | Incremental indexing: git diff + SHA-256 hash fallback |
| `src/index_history.py` | 10 | Append-only index history per project (capped at 100 entries) |
| `src/indexer.py` | 4, 9, 10 | Shared pipeline (CLI/MCP/Web), mode=auto/full/incremental, per-phase timing, health snapshots |

## Key Design Decisions

- **KuzuDB default, NetworkX fallback:** Single Entity table + 5 rel tables, Cypher support
- **all-MiniLM-L6-v2 for embeddings:** 384-dim, CPU-only (GPU sm_61 incompatible), ~80MB
- **LanceDB for vectors:** Embedded, fast, incremental add/delete support
- **FastMCP for MCP server:** Consistent with all other MCP servers in ecosystem
- **Incremental indexing:** git diff primary, hash fallback; graph+BM25 rebuilt (~2s); semantic truly incremental (delete+add)
- **Config over code:** All tunable parameters in `config/config.yaml`
- **6 languages:** Python, JavaScript, TypeScript/TSX, Java, Go — per-language extractors with shared base
- **LLM providers:** Claude, OpenAI, Gemini, Ollama — lazy imports, graceful degradation, file+env credentials
- **KuzuDB lock management:** close() on backends, _evict_cache() helpers, retry with gc.collect()+backoff