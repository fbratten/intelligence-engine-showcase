# Project Context -- Intelligence Engine

## What It Does

Intelligence Engine is a domain-agnostic platform for building, searching, and visualizing knowledge graphs from structured data. It started as a code intelligence tool -- parsing source code into typed knowledge graphs -- and evolved into a general-purpose engine where any domain can be defined through a YAML schema.

**In concrete terms:**

1. **Define a domain** via a YAML schema (entity types, relationships, properties, search profiles)
2. **Index data** using Tree-sitter (for code) or custom extractors (for any other format)
3. **Query** through hybrid search (keyword + semantic + graph), Cypher queries, or the REST API
4. **Explore** via an interactive web UI with force-directed graph visualization
5. **Integrate** with AI assistants through 15 MCP tools
6. **Analyze** with AI-powered summaries, Q&A, code quality metrics, and community detection

Everything runs locally. No data leaves the machine.

## Why It Exists

Managing a portfolio of 100+ projects creates a specific set of problems:

- **"What calls what?"** -- Understanding call chains, dependency flow, and the blast radius of a change across a large codebase
- **"Where is this pattern used?"** -- Finding implementations across dozens of projects that share conventions but aren't formally linked
- **"What's the quality landscape?"** -- Measuring complexity, documentation coverage, and coupling across the entire portfolio
- **"What connects to what?"** -- Understanding structural relationships that grep and text search can't reveal

Existing tools (IDE search, grep, static analyzers) work well within a single project but don't scale to portfolio-level analysis. Intelligence Engine fills that gap by building a unified knowledge graph that spans all indexed projects.

The domain generalization step (v0.21.0) extended this beyond code. The same infrastructure now handles archaeological data -- and any future domain that can be described by entities, relationships, and properties.

## Current Status

**Version:** 0.21.0
**Tests:** 1261+ passing
**Maturity:** Production-ready for personal use

### What Works

- Full pipeline from parsing through search, visualization, and AI integration
- 8 programming languages parsed via Tree-sitter
- 1 non-code domain (archaeology) validated via custom extractor
- Shared database mode supporting 100+ projects with cross-project queries
- 15 MCP tools for AI assistant integration
- 33 REST endpoints
- React web UI with graph explorer, Cypher console, and 6-tab dashboard
- AI summaries and Q&A via 4 LLM providers
- Incremental indexing with git diff detection
- AI data preservation across re-indexing and migration

### Design Philosophy

- **MVP-first:** Built in 12 phases, each extending the previous
- **Schema-driven:** Domain knowledge lives in YAML, not code
- **Fail loudly:** No silent failures -- crashes are preferable to silent data corruption
- **Local-only:** No cloud dependencies for core functionality (LLM providers are optional)
- **Extensible:** New domains require only a YAML schema and an extractor

## The Domain Generalization Story

Intelligence Engine started as a code-only tool (v0.1 through v0.20). The entire architecture was built around code concepts: functions, classes, methods, modules, CALLS, IMPORTS, EXTENDS.

At v0.21.0, the architecture was generalized:

1. **Hardcoded entity types** became **schema-defined entity types** from YAML
2. **Single Entity table** became **domain-scoped tables** (`Entity_code`, `Entity_archaeology`)
3. **Tree-sitter-only parsing** became **pluggable extractors** (Tree-sitter or custom modules)
4. **Code-specific search fields** became **configurable search profiles** per domain
5. **Code-specific health checks** became **domain-aware health checks** with skip types and metric selection

The archaeology domain was chosen as the validation case because it's maximally different from code -- no AST, no Tree-sitter, completely different entity semantics. Its successful implementation confirms that the domain abstraction works.

## Technology Choices

| Decision | Choice | Reason |
|----------|--------|--------|
| Graph DB | KuzuDB | Embedded, Cypher support, excellent Python bindings |
| Vector DB | LanceDB | Embedded, fast, incremental operations |
| Embeddings | all-MiniLM-L6-v2 | 384-dim, CPU-friendly, good quality/size ratio |
| Parser | Tree-sitter | Native Python bindings, multi-language, fast |
| MCP | FastMCP | Python-native, simple tool definition |
| Web backend | FastAPI | Async, fast, good typing support |
| Web frontend | React + Sigma.js | Component model + purpose-built graph visualization |
| Search fusion | RRF | Simple, robust, no training data needed |

See [decisions.md](decisions.md) for the full architectural decision log.
