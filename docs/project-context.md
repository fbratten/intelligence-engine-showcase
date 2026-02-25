# Project Context — Intelligence Engine

> **Reference:** `[PROJ-EBQKC-TNHEV] Intelligence Engine @ intelligence-engine`

## What This Project Is

A **local code intelligence engine** powered by GitNexus patterns — AST-driven knowledge graphs, hybrid search (BM25 + Semantic + Cypher), and MCP server integration — running on WSL Ubuntu alongside the existing MCP server ecosystem.

## Problem Statement

The user manages 113+ projects across ``. Current tooling (PAAF, Smart Inventory MCP) can audit docs and scan for debt markers, but **cannot understand code structure** — what calls what, where dependencies flow, what the blast radius of a change is. This engine fills that gap.

## Core Capabilities

1. **Parse codebases** using Tree-sitter into ASTs (6 languages: Python, JS, TS/TSX, Java, Go)
2. **Build a knowledge graph** of code entities and relationships (KuzuDB + NetworkX dual-backend)
3. **Generate embeddings** for semantic search (all-MiniLM-L6-v2 + LanceDB)
4. **Provide hybrid search** combining BM25, semantic, and graph search with 3-way RRF fusion
5. **Expose via MCP** — 11 tools for Claude Code and other agents
6. **Web UI** — React/Sigma.js graph explorer with FastAPI backend (15 endpoints)
7. **Incremental indexing** — git diff + hash fallback, ~3-4s for small changes vs ~60s full
8. **AI-powered summaries** — 4 LLM providers (Claude, OpenAI, Gemini, Ollama) for entity summaries
9. **Run 100% locally** on WSL Ubuntu — no code leaves the machine

## Technology Stack

- **Language:** Python 3.12+ (backend), TypeScript (frontend)
- **AST Parser:** py-tree-sitter (6 language grammars)
- **Graph DB:** KuzuDB 0.11.3 (default) + NetworkX (fallback)
- **Vector Store:** LanceDB
- **Embeddings:** all-MiniLM-L6-v2 (384-dim, CPU-only)
- **Keyword Search:** BM25 (rank_bm25)
- **MCP Server:** FastMCP
- **Web:** FastAPI + React 18 + Sigma.js 3 + Vite 6 + Tailwind CSS v4
- **Package Manager:** uv (Python), npm (frontend)

## Key Constraints

- Personal-use tooling (not enterprise)
- Fail loudly, never silently
- No backward compatibility
- MVP-first, each phase building on the last
- Reuse existing project code where possible
- Local only — no data leaves the machine

## Implementation Phases (All Complete)

1. **Phase 1 (MVP-0):** AST Parsing & Entity Extraction (51 tests)
2. **Phase 2 (MVP-1):** Knowledge Graph Storage (40 tests)
3. **Phase 3 (MVP-2):** BM25 + Hybrid Search (43 tests)
4. **Phase 4 (MVP-3):** MCP Server — 10 tools (23 tests)
5. **Phase 5 (MVP-4):** Multi-Project Registry (35 tests)
6. **Phase 6 (MVP-5):** Semantic Embeddings (34 tests)
7. **Phase 6.5:** KuzuDB Migration (73 tests)
8. **Phase 7 (MVP-6):** Web UI — FastAPI + React/Sigma.js (25 tests)
9. **Phase 8:** Multi-Language Support — 6 languages (215 tests)
10. **Phase 9:** Incremental Indexing (46 tests)
11. **Phase 10:** Performance Dashboard — per-phase timing, health snapshots, 4-tab UI (24 tests)
12. **Phase 11:** AI-Powered Summaries + KuzuDB Lock Fix — 4 LLM providers, ie_summarize, SettingsDialog (38 tests)

**Total: 647 tests passing, 12 projects indexed.**

## KB Documents

All detailed reference material is in `KB/`:
- `CLAUDE.md` — Main instruction document
- `REF-gitnexus-architecture.md` — GitNexus architecture deep-dive
- `REF-existing-projects-audit.md` — Reusable code audit across 49+ projects
- `REF-implementation-phases.md` — Detailed step-by-step implementation
- `REF-mcp-integration-guide.md` — MCP server integration patterns
- `REF-technology-stack.md` — Technology choices and installation
- `memory-systems-guide.md` — Memory integration guide