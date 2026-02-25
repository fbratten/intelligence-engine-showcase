# Decisions Log — Intelligence Engine

## ADR-001: Python over Node.js

**Decision:** Use Python 3.11+ as the primary language.

**Rationale:**
- Direct reuse of win_rag_mini embedding pipeline (BGE-M3 + LanceDB)
- Direct reuse of existing MCP server patterns (all Python-based)
- py-tree-sitter works natively — no WASM overhead
- KuzuDB has excellent Python bindings
- Consistent with the development environment

**Alternatives considered:** Node.js (GitNexus's choice — browser-based, not applicable here)

---

## ADR-002: NetworkX before KuzuDB

**Decision:** Start with NetworkX for graph storage, migrate to KuzuDB when schema stabilizes.

**Rationale:**
- Zero extra dependency for MVP
- Faster iteration while exploring entity/relationship schema
- KuzuDB migration path is clear (same Cypher query language)

**Alternatives considered:** KuzuDB from day 1, Neo4j

---

## ADR-003: Reuse-First Strategy

**Decision:** Audit and reuse code from existing projects before writing new code.

**Rationale:**
- 49+ projects scanned, 5 CRITICAL reuse sources identified
- BGE-M3 pipeline exists and is proven (win_rag_mini)
- MCP server patterns are standardized across ecosystem
- Executor hooks pattern (spine-integrations) matches our integration needs exactly

---

## ADR-004: SPINE-Managed Project

**Decision:** Structure project as SPINE-managed with full Minna Memory integration.

**Rationale:**
- Consistent with ecosystem conventions
- Cross-session memory for ADRs, error resolutions, patterns
- Tiered task classification ensures proper tool usage
- MCP server ecosystem provides extensive auxiliary capabilities