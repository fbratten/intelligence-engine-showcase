---
layout: default
title: Architectural Decisions
---

# Architectural Decisions -- Intelligence Engine

## ADR-001: Python as Primary Language

**Decision:** Use Python 3.12+ as the primary backend language.

**Rationale:**
- py-tree-sitter works natively with no WASM overhead
- KuzuDB has excellent Python bindings
- LanceDB, sentence-transformers, and FastMCP are all Python-first
- FastAPI provides high-performance async web serving
- Consistent with the broader tooling ecosystem

**Alternatives considered:** Node.js (better for browser-based tools, but introduces WASM overhead for Tree-sitter and lacks native KuzuDB bindings).

---

## ADR-002: NetworkX First, Then KuzuDB

**Decision:** Start with NetworkX for graph storage, migrate to KuzuDB when the schema stabilized.

**Rationale:**
- Zero extra dependency for the initial MVP
- Faster iteration while exploring entity/relationship schemas
- KuzuDB migration path was clear (same Cypher query language)

**Outcome:** Both backends are maintained. KuzuDB is the default (Cypher support, persistence, domain-scoped tables). NetworkX remains as a fallback for environments where KuzuDB is unavailable.

---

## ADR-003: Option B -- Separate Entity Tables Per Domain

**Decision:** Create separate `Entity_<domain>` and `Rel_<domain>_*` tables for each domain, rather than a single `Entity` table with a `domain` column.

**Rationale:**
- **Schema safety** -- Domain-specific properties (e.g., `complexity` for code, `latitude` for archaeology) live in their own table columns without NULL pollution across domains
- **Query performance** -- Cypher queries on `Entity_code` scan only code entities, not the entire entity set
- **Independent evolution** -- Adding properties to one domain never affects another domain's table schema
- **Clear separation** -- Each domain's graph is structurally isolated, reducing accidental cross-domain queries

**Trade-offs:**
- Cross-domain queries require explicit JOINs or UNION across tables
- More tables in KuzuDB (one per domain + one per relationship type per domain)
- Schema migration requires per-domain handling

**Alternatives considered:**
- *Option A (single Entity table):* Simpler to implement but leads to wide, sparse tables as domains accumulate domain-specific columns
- *Option C (separate databases per domain):* Too isolated -- prevents cross-domain queries entirely

---

## ADR-004: (project, domain) as Routing Key

**Decision:** All operations are routed by the tuple `(project, domain)`.

**Rationale:**
- A single project can contain entities from multiple domains (e.g., code + documentation)
- Domain-specific extractors, search profiles, and health checks are selected by this key
- Shared mode uses `project` column filtering; domain selects the correct table set
- The web UI exposes this as a project dropdown with domain selector

---

## ADR-005: Schema Versioning from Day One

**Decision:** Every domain schema includes a `version` field. The indexer records `schema_version` in project metadata at index time.

**Rationale:**
- Enables detection of stale indexes (schema changed but project not re-indexed)
- Future: automatic re-indexing when schema version changes
- Future: schema migration tooling for breaking changes
- Low cost to implement, high value for maintainability

**Current behavior:** The indexer compares stored `schema_version` against the current schema. If they differ, the project is flagged as stale in `ie_status` output.

---

## ADR-006: 3-Way RRF Fusion for Search

**Decision:** Combine BM25, semantic, and graph search using Reciprocal Rank Fusion with fixed weights (0.35, 0.40, 0.25).

**Rationale:**
- BM25 excels at exact identifier matching (function names, class names)
- Semantic search handles natural language queries and conceptual similarity
- Graph search adds structural context (callers, imports, related entities)
- RRF is simple, robust, and doesn't require training data
- Fixed weights work well in practice; could be made configurable per domain via search profiles

**Formula:** `score = sum(1 / (k + rank_i))` across strategies, `k = 60`.

---

## ADR-007: Shared Database Mode as Optional

**Decision:** Support both per-project (isolated) and shared (multi-tenant) storage modes, with a migration tool to switch between them.

**Rationale:**
- Per-project mode is simpler and works for most use cases
- Shared mode enables cross-project Cypher queries and global analysis
- Making it optional avoids forcing the complexity of multi-tenant filtering on users who don't need it
- The migration tool handles bidirectional conversion with verification

**Shared mode implementation:**
- KuzuDB: single database with `project` column on all entity/relationship tables
- LanceDB: single vector store with `project` column
- BM25: remains per-project (in-memory, rebuilt on load)

---

## ADR-008: Archaeology as Validation Domain

**Decision:** Use archaeology as the first non-code domain to validate the domain generalization architecture.

**Rationale:**
- Maximally different from code -- no AST, no Tree-sitter, completely different entity and relationship semantics
- Requires custom extractor (YAML/JSON input), proving the extractor abstraction works
- Has rich relationship types (spatial, temporal, material) that exercise the schema system
- Real-world domain with genuine analytical value, not a toy example
- If archaeology works, any structured domain can be added

**Outcome:** Successfully validated:
- YAML schema definition drives table creation
- Custom extractor module loading works
- Domain-specific properties, relationships, and search profiles function correctly
- Web UI handles non-code entities without code-specific assumptions

---

## ADR-009: Read-Only Serving with Input Sanitization

**Decision:** The web server and MCP server open KuzuDB in `read_only=True` mode. All Cypher input is validated to block write operations.

**Rationale:**
- Multiple readers can coexist without lock contention
- Prevents accidental or malicious data modification through the API
- Write operations (indexing, migration) require stopping the server first -- an acceptable trade-off for personal use
- Cypher validation strips comments and strings before checking for write keywords (DELETE, CREATE, SET, DROP, MERGE, ALTER, COPY, REMOVE, DETACH)

---

## ADR-010: AI Data Preservation Across Re-Indexing

**Decision:** Extract AI-generated data (summaries, Q&A history) before re-indexing and restore it afterward.

**Rationale:**
- LLM summaries are expensive to regenerate (API cost + time)
- Q&A history represents accumulated knowledge that shouldn't be lost
- Re-indexing destroys and recreates graph entities, which would lose all AI annotations
- The extraction/restoration approach is simple and reliable -- no schema changes needed

**Implementation:** `ai_overlay.py` extracts AI data keyed by entity identity before rebuild, then matches and restores after the new graph is populated.
