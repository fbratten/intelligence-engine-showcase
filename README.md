# Intelligence Engine

> Triple-search knowledge backbone — Kuzu graph + LanceDB vectors + BM25 full-text across 27 indexed projects

## What is Intelligence Engine?

A **local code intelligence engine** — AST-driven knowledge graphs, hybrid search (BM25 + Semantic + Graph), and both MCP server and web UI. Runs 100% locally on WSL Ubuntu.

```
Tree-sitter AST  →  Knowledge Graph  →  Hybrid Search  →  MCP Server / Web UI
  (6 languages)      (KuzuDB + Cypher)   (BM25+Vector+    (12 tools + React)
                                          Graph RRF)
```

## Key Features

- **12 MCP Tools** for AI assistant integration
- **23 REST Endpoints** for web UI and API access
- **734 Tests** passing across 12 phases
- **6 Languages** supported (Python, JavaScript, TypeScript/TSX, Java, Go)
- **Triple Search** — BM25 full-text + LanceDB semantic vectors + Kuzu graph traversal
- **Reciprocal Rank Fusion** for combining search strategies

## Architecture

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Parser** | Tree-sitter | AST extraction for 6 languages |
| **Graph** | KuzuDB (+ NetworkX fallback) | Knowledge graph with Cypher queries |
| **Search** | BM25 + LanceDB + Graph RRF | Hybrid search with rank fusion |
| **MCP** | FastMCP | 12 tools for AI assistants |
| **Web** | FastAPI + React 18 + Sigma.js | REST API + interactive graph explorer |
| **LLM** | Claude, OpenAI, Gemini, Ollama | AI-powered entity summaries |

## Search Strategies

| Strategy | Best For | How It Works |
|----------|----------|--------------|
| **BM25** | Exact keyword matching | TF-IDF token scoring |
| **Semantic** | Natural language queries | sentence-transformers + LanceDB |
| **Graph** | Structural queries | Kuzu graph traversal + Cypher |
| **Triple** | Best overall results | RRF fusion of all three |

## Web UI

React 18 + Sigma.js 3 graph explorer with:
- Interactive force-directed graph visualization (ForceAtlas2)
- Hybrid search with strategy selector dropdown
- Entity detail with source code preview and complexity badges
- Cypher console with monospace table output
- Performance dashboard (timeline, phases, health, compare)
- AI-powered entity summaries (multi-provider: Claude, OpenAI, Gemini, Ollama)
- Batch summarization with progress tracking
- Code quality metrics (composite scores, complexity histograms, coupling)
- Graph clustering with community detection
- Dark-themed UI with custom scrollbars

## MCP Tools (12)

| Tool | Purpose |
|------|---------|
| `ie_index` | Index a project into the knowledge graph |
| `ie_query` | Natural language search across indexed projects |
| `ie_context` | Get rich context for a specific entity |
| `ie_detect_changes` | Find what changed since last index |
| `ie_wiki` | Generate wiki-style documentation |
| `ie_status` | Project indexing status and statistics |
| `ie_search_all` | Cross-project search across all indexed repos |
| `ie_health` | System health check |
| `ie_cypher` | Execute raw Cypher queries on the graph |
| `ie_summarize` | AI-powered entity summarization |
| `ie_batch_summarize` | Bulk summarization with progress tracking |
| `ie_quality` | Code quality metrics and scores |

## Interactive Demo

**[Which Search Strategy?](demos/search-picker/index.html)** — Interactive decision tree that helps you choose between BM25, semantic, graph, and triple search based on your query type.

## Documentation

| Document | Description |
|----------|-------------|
| [Technical Overview](docs/technical-overview.md) | Full technical reference with architecture and conventions |
| [Architecture](docs/architecture.md) | Component details, data flow, and design patterns |
| [Decisions](docs/decisions.md) | Architectural decision records |
| [Project Context](docs/project-context.md) | What the project is, problem statement, and approach |

## Implementation Phases

| Phase | Feature | Status |
|-------|---------|--------|
| 1 | AST Parsing (Tree-sitter) | Complete |
| 2 | Knowledge Graph (NetworkX + KuzuDB) | Complete |
| 3 | Hybrid Search (BM25 + Graph RRF) | Complete |
| 4 | MCP Server (12 tools via FastMCP) | Complete |
| 5 | Multi-Project (registry, cross-project search) | Complete |
| 6 | Semantic Embeddings (sentence-transformers + LanceDB) | Complete |
| 6.5 | KuzuDB Migration (dual-backend, Cypher queries) | Complete |
| 7 | Visual UI (React + Sigma.js graph explorer) | Complete |
| 8 | Multi-Language Support (6 languages) | Complete |
| 9 | Incremental Indexing (git diff + hash fallback) | Complete |
| 10 | Performance Dashboard | Complete |
| 11 | AI-Powered Summaries (multi-provider LLM) | Complete |
| 12 | Graph Clustering (Louvain community detection) | Complete |

## Technology Stack

- **Language:** Python 3.11+ (backend), TypeScript (frontend)
- **Graph Database:** KuzuDB with Cypher query support
- **Vector Store:** LanceDB with sentence-transformers
- **Full-Text:** BM25 (rank_bm25)
- **Web Backend:** FastAPI (23 endpoints)
- **Web Frontend:** React 18 + Sigma.js 3 + Vite
- **Parser:** Tree-sitter (6 language grammars)
- **Testing:** pytest (734 tests)

---

## Acknowledgements

Thanks to [GitNexus](https://github.com/nicholasgriffintn/GitNexus) by Nicholas Griffin for pointing me to [KuzuDB](https://kuzudb.com/) — which saved me from jumping through hoops trying to get "true" Cypher query support working with other graph databases — and to [Sigma.js](https://www.sigmajs.org/) for interactive graph visualization. Both technologies have since become staples across several of my projects.

---

*This showcase was generated from the source project using [showcase-mcp](https://github.com/fbratten/showcase-mcp). Source code is proprietary.*
