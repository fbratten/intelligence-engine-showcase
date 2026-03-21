---
layout: default
title: Intelligence Engine
nav_order: 1
---

# Intelligence Engine

> Domain-agnostic intelligence engine with schema-driven knowledge graphs, hybrid search, and MCP server.

![Python 3.12+](https://img.shields.io/badge/python-3.12+-blue.svg)
![Version](https://img.shields.io/badge/version-0.21.0-green.svg)

**1261+ Tests** · **15 MCP Tools** · **33 REST Endpoints** · **8 Languages** · **2 Domains**

---

## Screenshots

| | |
|---|---|
| ![Graph](screenshots/04-graph-v21.png) | ![Cypher](screenshots/05-cypher-results.png) |
| **Knowledge Graph** — 5000 nodes, 7048 edges | **Cypher Console** — 50 rows in 9ms |
| ![Timeline](screenshots/06-dashboard-timeline.png) | ![Health](screenshots/08-dashboard-health.png) |
| **Dashboard Timeline** — indexing history | **Health Metrics** — dead code, cycles, hubs |
| ![Compare](screenshots/09-dashboard-compare.png) | ![AI Memory](screenshots/10-dashboard-aimem.png) |
| **Cross-Project Compare** — all indexed projects | **AI Memory** — KuzuDB + Minna browser |

---

## Architecture

```mermaid
graph LR
    A["Domain Schema<br/><small>YAML</small>"] --> B["Extractors<br/><small>Tree-sitter / Custom</small>"]
    B --> C["Knowledge Graph<br/><small>KuzuDB</small>"]
    C --> D["Hybrid Search<br/><small>BM25 + Semantic + Graph</small>"]
    D --> E["MCP / Web UI<br/><small>15 tools · 33 endpoints</small>"]
```

---

## Documentation

- [Architecture Deep-Dive](docs/architecture) — Domain schema system, pipeline, storage modes
- [Technical Overview](docs/technical-overview) — Parser, graph, search, AI, MCP, REST details
- [Architectural Decisions](docs/decisions) — Key design choices and rationale
- [Project Context](docs/project-context) — What it does, why it exists, status

---

## Interactive Demos

- [Which Search Strategy?](demos/search-picker/) — Choose the right search approach for your query

---

## Highlights

- Schema-driven domains: define entity types, relationships, and search profiles in YAML
- Code intelligence: 8 languages parsed via Tree-sitter (Python, JS, TS/TSX, Java, Go, HTML, CSS)
- Archaeology MVP: first non-code domain validates domain-agnostic architecture
- Hybrid search: 3-way RRF fusion (BM25 + semantic + graph)
- Graph visualization: React + Sigma.js force-directed explorer
- AI integration: summaries and Q&A via Claude, OpenAI, Gemini, Ollama
- MCP server: 15 tools for AI coding assistants
- Multi-project: shared DB mode with cross-project Cypher queries

---

[GitHub](https://github.com/fbratten/intelligence-engine-showcase)
