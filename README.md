# Intelligence Engine

> `[PROJ-EBQKC-TNHEV]` Local code intelligence engine powered by GitNexus patterns.

AST-driven knowledge graphs, hybrid search (BM25 + Semantic + Graph), and both MCP server and web UI. Runs 100% locally on WSL Ubuntu.

## Status

**All phases complete.** 734 tests passing.

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
| 8 | Multi-Language Support (Python, JS, TS/TSX, Java, Go) | Complete |
| 9 | Incremental Indexing (git diff + hash fallback) | Complete |
| 10 | Performance Dashboard (timeline, phases, health, compare) | Complete |
| 11 | AI-Powered Summaries (Claude, OpenAI, Gemini, Ollama) | Complete |
| 12 | Graph Clustering (Louvain community detection) | Complete |
| — | Batch Summaries (bulk LLM summarization) | Complete |
| — | Code Quality Metrics (cyclomatic complexity, quality scores) | Complete |
| — | UI Polish (scrollbars, visual hierarchy, table improvements) | Complete |

## Quick Start

```bash
cd intelligence-engine
source .venv/bin/activate

# Index a project
python -m intelligence_engine parse /path/to/project --output data/myproject/parse.json
python -m intelligence_engine load data/myproject/parse.json
python -m intelligence_engine index data/myproject/parse.json

# Search
python -m intelligence_engine search data/myproject/parse.json "find authentication"

# Start web UI (backend on :8420, frontend on :5173+)
python -m intelligence_engine serve &
cd src/intelligence_engine/web/frontend && npm run dev
```

## Architecture

```
Tree-sitter AST  -->  Knowledge Graph  -->  Hybrid Search  -->  MCP Server / Web UI
  (parser/)           (graph/)              (search/)           (mcp/ + web/)
  6 languages         NetworkX | KuzuDB     BM25 + Semantic     12 MCP tools
  complexity calc     Cypher queries        + Graph RRF         FastAPI + React
```

## Web UI

React 18 + Sigma.js 3 graph explorer with:
- Interactive force-directed graph visualization (ForceAtlas2)
- Hybrid search (BM25 / Semantic / Graph / combined) with polished dropdown
- Entity detail with source code preview, complexity badges, and visual hierarchy
- Cypher console with sticky headers and monospace table output
- Performance dashboard (timeline, phases, health, compare, quality)
- AI-powered entity summaries (Claude, OpenAI, Gemini, Ollama)
- Batch summarization with progress tracking
- Code quality metrics (composite scores, complexity histograms, coupling)
- Graph clustering with community detection (color by type or cluster)
- Project indexing and management from the browser
- Node/edge type filtering and depth-based exploration
- LLM settings dialog (multi-provider credential management)
- Custom dark-themed scrollbars and polished visual styling

## MCP Server

12 tools for AI assistants: `ie_index`, `ie_query`, `ie_context`, `ie_detect_changes`, `ie_wiki`, `ie_status`, `ie_search_all`, `ie_health`, `ie_cypher`, `ie_summarize`, `ie_batch_summarize`, `ie_quality`.

## Languages Supported

Python, JavaScript, TypeScript, TSX, Java, Go — all parsed via Tree-sitter with per-language cyclomatic complexity calculation.

## Documentation

- **[`docs/ui-guide.md`](docs/ui-guide.md)** — Complete UI walkthrough (all features, controls, tips)
- `CLAUDE.md` — SPINE-managed project context
- `NEXT.md` — Task tracking
- `CHANGELOG.md` — Version history
- `KB/CLAUDE.md` — Main instruction document
- `KB/REF-*.md` — Reference documents (architecture, phases, tech stack)

## License

Personal use. Not open-source.

## Interactive Demos

- **[Which Search Strategy?](demos/search-picker/)** — Choose the right search approach for your query

---

*This showcase was generated from the source project using [showcase-mcp](https://github.com/fbratten/showcase-mcp). Source code is proprietary.*
