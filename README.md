# Intelligence Engine — Triple-Search Knowledge Backbone

> Kuzu graph + LanceDB vectors + BM25 full-text search across 27 indexed projects and 597MB of code intelligence.

**734 Tests** | **27 Projects Indexed** | **597MB Data** | **15 MCP Tools**

---

## What It Does

Intelligence Engine is the **knowledge backbone** for a portfolio of 180+ projects. It indexes source code, documentation, and project structure into three complementary search backends — then exposes everything as MCP tools.

```
Source Code → [INDEX] → Kuzu Graph    → Cypher queries (relationships)
                      → LanceDB       → Vector search (semantic similarity)
                      → BM25 Index    → Full-text search (exact keywords)
                                ↓
                     ie_search_all → Triple search (all three at once)
```

---

## Interactive Demo

**[Which Search Strategy?](demos/search-picker/)** — Decision tree to pick the right search approach for your query.

---

## Search Strategies

| Strategy | Backend | Best For | Example |
|----------|---------|----------|---------|
| **Graph** | Kuzu (property graph) | Relationships, dependencies, "which projects use X" | `ie_cypher "MATCH (p:Project)-[:USES]->(m:MCP) RETURN p.name"` |
| **Vector** | LanceDB (embeddings) | Semantic similarity, "find code that does X" | `ie_search_all "error handling patterns"` |
| **BM25** | Full-text index | Exact names, identifiers, error messages | `ie_search_all "ContentPipelineExecutor"` |
| **Triple** | All three merged | Not sure what you're looking for | `ie_search_all "music generation"` |

---

## MCP Tools (15)

| Tool | Purpose |
|------|---------|
| `ie_index` | Index a project (code, docs, structure) |
| `ie_query` | Graph query with natural language |
| `ie_cypher` | Direct Cypher query against Kuzu graph |
| `ie_search_all` | Triple search (graph + vector + BM25) |
| `ie_context` | Get rich context for an entity |
| `ie_wiki` | Generate wiki page for any entity |
| `ie_summarize` | Summarize a project or entity |
| `ie_batch_summarize` | Batch summarization across projects |
| `ie_detect_changes` | Detect what changed since last index |
| `ie_status` | Index health and statistics |
| `ie_health` | System health check |
| `ie_quality` | Data quality assessment |

---

## What's Indexed

- **27 projects** — source code, documentation, configs
- **597MB** of structured knowledge
- **Relationships**: function calls, imports, MCP usage, project dependencies
- **Entities**: projects, files, functions, classes, MCP servers, tools

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│              INTELLIGENCE ENGINE                 │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │  Kuzu    │  │ LanceDB  │  │  BM25    │      │
│  │  Graph   │  │  Vector  │  │  Index   │      │
│  │ (Cypher) │  │ (Embed)  │  │ (FTS)    │      │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘      │
│       └──────────────┼──────────────┘            │
│                      ▼                           │
│              ┌──────────────┐                    │
│              │ MCP Server   │                    │
│              │ (15 tools)   │                    │
│              └──────────────┘                    │
└─────────────────────────────────────────────────┘
```

---

## Documentation

- [Technical Overview](docs/technical-overview.md) — Full system details from CLAUDE.md
- [Architecture](docs/architecture.md) — Design decisions and search strategy
- [Decisions](docs/decisions.md) — Key architectural decisions
- [Project Context](docs/project-context.md) — Background and history

---

## Quality

- **734 tests** — comprehensive coverage
- **PAAF health score: 98/100** — highest in portfolio
- Triple search ensures no single point of failure for queries

---

## Related Projects

| Project | Role |
|---------|------|
| [SPINE](https://github.com/fbratten/spine) | Context engineering backbone |
| [portfolio-ops-hub](https://github.com/fbratten/portfolio-ops-hub) | Portfolio coordination (uses IE for knowledge) |

---

## Tech Stack

Python 3.11+ | Kuzu (graph DB) | LanceDB (vector DB) | BM25 | FastMCP | pytest

---

*Built with [showcase-mcp](https://github.com/fbratten/showcase-mcp)*
