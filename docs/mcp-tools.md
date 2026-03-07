---
title: "MCP Tools Reference"
nav_order: 3
---

# MCP Tools Reference

Intelligence Engine exposes 15 MCP tools for code intelligence, search, analysis, and AI-powered documentation. All tools are prefixed `ie_` and available via the `intelligence-engine` MCP server.

**Prerequisite:** Most tools require a project to be indexed first using `ie_index`.

---

## Tool Categories

| Category | Tools |
|----------|-------|
| **Indexing** | `ie_index`, `ie_status` |
| **Search** | `ie_query`, `ie_search_all` |
| **Graph Navigation** | `ie_context`, `ie_cypher` |
| **Quality & Health** | `ie_quality`, `ie_health` |
| **Change Analysis** | `ie_detect_changes` |
| **AI Summaries** | `ie_summarize`, `ie_batch_summarize` |
| **Documentation** | `ie_wiki` |
| **Cross-Project** | `ie_global_analysis` |
| **Memory** | `ie_memory` |

---

## 1. ie_index

Index a project codebase for code intelligence. Parses source code (Python, JavaScript, TypeScript, Java, Go), builds a knowledge graph, and creates search indices. Must be run before using other `ie_*` tools on a project.

Supports incremental indexing: when `mode='auto'` (default), re-indexes only changed files if a previous index exists.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project_path` | string | Yes | - | Path to project root directory |
| `mode` | string | No | `"auto"` | `"auto"` (incremental if possible), `"full"` (rebuild), `"incremental"` (fail if not possible) |
| `force` | boolean | No | `false` | Re-index even if already indexed |

### Example Usage

```json
{
  "project_path": "/home/simon/projects/paaf",
  "mode": "auto"
}
```

### Example Output

```
Indexed project 'paaf': 457 entities, 1823 relationships
Files: 42 Python files parsed
Time: 12.3s
```

---

## 2. ie_status

Check the index status of projects. Shows which projects are indexed, their entity counts, and when they were last indexed.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | No | `null` | Project name. If omitted, shows all indexed projects. |

### Example Usage

```json
{}
```

```json
{
  "project": "paaf"
}
```

### Example Output

```
Indexed Projects:
  paaf         457 entities   2026-03-05 14:30
  openclaw     312 entities   2026-03-04 09:15
  agent-comm    89 entities   2026-03-03 11:42
```

---

## 3. ie_query

Search a project's codebase using hybrid BM25 + graph search. Returns ranked results with file:line citations and graph context (callers/callees).

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |
| `query` | string | Yes | - | Search query text (max 500 chars) |
| `method` | string | No | `"hybrid"` | `"hybrid"` (BM25+semantic+graph), `"bm25"` (keyword only), `"semantic"` (vector only), `"graph"` (BM25+graph, no semantic) |
| `top_k` | integer | No | `10` | Number of results to return |

### Example Usage

```json
{
  "project": "paaf",
  "query": "authentication middleware",
  "method": "hybrid",
  "top_k": 5
}
```

### Example Output

```
Results for "authentication middleware" in paaf (hybrid):

1. authenticate_request (function)        score: 0.92
   src/middleware/auth.py:45
   Callers: app_factory, register_routes
   Callees: verify_token, get_user_session

2. AuthMiddleware (class)                 score: 0.87
   src/middleware/auth.py:12
   Methods: __init__, __call__, authenticate_request
```

---

## 4. ie_search_all

Search across ALL indexed projects simultaneously. Returns results grouped by project with relevance scores. In shared DB mode, uses a single vector query (much faster).

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | string | Yes | - | Search query text (max 500 chars) |
| `top_k` | integer | No | `10` | Max results per project |

### Example Usage

```json
{
  "query": "database connection pool",
  "top_k": 5
}
```

### Example Output

```
Cross-project results for "database connection pool":

paaf (3 results):
  1. create_pool (function)     src/db/pool.py:22        score: 0.91
  2. DatabasePool (class)       src/db/pool.py:5         score: 0.85

openclaw (2 results):
  1. get_connection (function)  src/storage/db.ts:44     score: 0.78
  2. ConnectionManager (class)  src/storage/db.ts:10     score: 0.72
```

---

## 5. ie_context

Get 360-degree context for a code entity -- callers, callees, and relationships. Shows who calls it, what it calls, and related entities.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |
| `entity_name` | string | Yes | - | Entity name to look up (e.g. `"scan_debt"`) |
| `depth` | integer | No | `1` | Depth for relationship traversal (1-3) |

### Example Usage

```json
{
  "project": "paaf",
  "entity_name": "scan_debt",
  "depth": 2
}
```

### Example Output

```
Entity: scan_debt (function)
File: src/analyzers/debt.py:89
Complexity: 12

Callers (who calls scan_debt):
  main                  src/cli.py:45
  run_analysis          src/runner.py:112

Callees (what scan_debt calls):
  parse_file            src/parsers/base.py:23
  calculate_score       src/analyzers/scoring.py:56
  format_results        src/output/formatter.py:78

Blast Radius: 5 entities affected by changes to scan_debt
```

---

## 6. ie_cypher

Execute a read-only Cypher query against a project's knowledge graph (KuzuDB backend). Write operations (CREATE, DELETE, SET, DROP, etc.) are blocked.

**CRITICAL:** Use `n.entity_type` not `n.type` for filtering entity types. Always include a `LIMIT` clause to avoid overwhelming results.

### Entity Table Columns

`id`, `project`, `name`, `entity_type`, `file`, `line_start`, `line_end`, `is_exported`, `docstring`, `params`, `decorators`, `bases`, `code_snippet`, `language`, `complexity`, `summary`, `summary_provider`, `qa_history`

### Edge Types

`CALLS`, `IMPORTS`, `EXTENDS`, `DEFINES`, `METHOD_OF`

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |
| `query` | string | Yes | - | Cypher query string (max 2000 chars) |

### Example Usage

```json
{
  "project": "paaf",
  "query": "MATCH (n:Entity) WHERE n.entity_type = 'class' RETURN n.name, n.file LIMIT 20"
}
```

### Example Output

```
| n.name           | n.file                    |
|------------------|---------------------------|
| DebtScanner      | src/analyzers/debt.py     |
| AuthMiddleware    | src/middleware/auth.py     |
| DatabasePool     | src/db/pool.py            |
```

---

## 7. ie_quality

Get code quality metrics for a project: composite score, complexity distribution, documentation coverage, function length stats, module coupling, and fan-in/out metrics.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |

### Example Usage

```json
{
  "project": "paaf"
}
```

### Example Output

```
Composite Score: 69.1/100

Complexity:
  Average: 1.6
  Max: 43 (main)
  High complexity (>10): 5 entities

Doc Coverage: 39.6% (181/457)

Function Length:
  Average: 12.3 lines
  Max: 156 lines (main)

Module Coupling:
  Average fan-out: 3.2
  Max fan-out: 18 (cli.py)
  Average fan-in: 2.1
  Max fan-in: 24 (utils.py)
```

---

## 8. ie_health

Get code health metrics from graph analysis: dead code count, circular dependencies, and hub concentration.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |

### Example Usage

```json
{
  "project": "paaf"
}
```

### Example Output

```
Dead Code: 239 entities (no callers, not exported)
Cycles: 0 circular dependency chains
Hub Concentration: 12.0% (top 5 entities account for 12% of all edges)

Top Hubs:
  main              18 connections
  utils.format      14 connections
  db.get_connection  12 connections

Dead Code Samples:
  _legacy_parse     src/parsers/old.py:34
  unused_helper     src/utils/misc.py:89
```

---

## 9. ie_detect_changes

Analyze the blast radius of changed files to assess risk. Identifies entities in the changed files and traces their callers/dependents.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |
| `files` | string[] | Yes | - | List of changed file paths (relative to project root) |

### Example Usage

```json
{
  "project": "paaf",
  "files": ["src/db/pool.py", "src/middleware/auth.py"]
}
```

### Example Output

```
Changed files: 2
Entities in changed files: 8
Directly affected callers: 14
Total blast radius: 22 entities

High-risk changes:
  DatabasePool.get_connection    12 callers affected
  authenticate_request            6 callers affected

Affected files:
  src/cli.py
  src/runner.py
  src/api/routes.py
```

---

## 10. ie_summarize

Generate an AI-powered natural language summary for a single code entity. Requires an LLM provider to be configured (Claude, OpenAI, Gemini, or Ollama).

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |
| `entity_name` | string | No | `null` | Entity name (alternative lookup) |
| `entity_id` | string | No | `null` | Full entity ID |
| `provider` | string | No | `null` | LLM provider (auto-detected if omitted) |
| `max_tokens` | integer | No | `300` | Max tokens for summary |

### Example Usage

```json
{
  "project": "paaf",
  "entity_name": "scan_debt"
}
```

### Example Output

```
scan_debt (function) - src/analyzers/debt.py:89

Scans a Python project directory for technical debt indicators. Walks the
file tree, parses each Python file using AST, and evaluates complexity,
documentation coverage, and code smells. Returns a DebtReport with per-file
scores and an aggregate project health score.

Called by: main, run_analysis
Calls: parse_file, calculate_score, format_results
Complexity: 12
```

---

## 11. ie_batch_summarize

Generate AI summaries for all entities in a project. Processes entities sequentially with a configurable delay between LLM calls for rate limit protection. Skips external and module nodes.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |
| `entity_types` | string[] | No | `null` | Filter to specific types (e.g. `["function", "class"]`) |
| `provider` | string | No | `null` | LLM provider (auto-detected if omitted) |
| `max_tokens` | integer | No | `300` | Max tokens per summary |
| `delay_seconds` | number | No | `1` | Pause between LLM calls (rate limit protection) |
| `skip_existing` | boolean | No | `true` | Skip entities that already have summaries |

### Example Usage

```json
{
  "project": "paaf",
  "entity_types": ["function", "class"],
  "delay_seconds": 2,
  "skip_existing": true
}
```

### Example Output

```
Batch summarization for paaf:
  Total entities: 457
  Skipped (existing): 120
  Skipped (external/module): 89
  To summarize: 248

Progress: 248/248 complete
Errors: 0
Time: 8m 32s
```

---

## 12. ie_wiki

Generate markdown documentation from the knowledge graph. Can produce a full project overview, a single module breakdown, or a single entity page.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |
| `scope` | string | No | `"full"` | `"full"` (project overview), `"module"` (single file), `"function"` (single entity) |
| `entity_name` | string | No | `null` | Entity or module name (required for `module`/`function` scope) |

### Example Usage

```json
{
  "project": "paaf",
  "scope": "module",
  "entity_name": "src/analyzers/debt.py"
}
```

### Example Output

```markdown
# Module: src/analyzers/debt.py

## Overview
Technical debt analysis module containing the core scanning logic.

## Classes
- **DebtScanner** - Main scanner class for debt analysis

## Functions
- **scan_debt** - Scans project for technical debt indicators
- **calculate_score** - Computes debt score from raw metrics

## Dependencies
- Imports from: src/parsers/base, src/output/formatter
- Imported by: src/cli, src/runner
```

---

## 13. ie_global_analysis

Analyze the entire codebase across ALL projects in shared DB mode. Runs community detection and health analysis on the unified knowledge graph. Each community shows which projects its members come from.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `include_clusters` | boolean | No | `true` | Run community detection across all projects |
| `include_health` | boolean | No | `true` | Compute dead code, cycles, hubs across all projects |

### Example Usage

```json
{
  "include_clusters": true,
  "include_health": true
}
```

### Example Output

```
Global Analysis (3 projects, 858 entities):

Communities:
  Cluster 1: paaf (scan_debt, calculate_score), openclaw (analyze)
  Cluster 2: agent-comm (comm_send, comm_poll, comm_ack)
  Cluster 3: paaf (DatabasePool), openclaw (StorageManager)

Cross-Project Health:
  Total dead code: 312 entities
  Cross-project cycles: 0
  Shared patterns: 4 (logging, config, db, auth)
```

---

## 14. ie_memory

Unified memory browser aggregating data from two sources: KuzuDB (per-entity summaries and Q&A history) and Minna Memory (cross-session SQLite memories including session handovers, decisions, known issues).

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `project` | string | Yes | - | Project name (as indexed) |
| `entity_name` | string | No | `null` | Filter by entity name |
| `source` | string | No | `null` | Filter by source: `"kuzu"` or `"minna"` |
| `search` | string | No | `null` | Full-text search query (Minna FTS) |
| `attribute` | string | No | `null` | Filter by attribute type |
| `minna_scope` | string | No | `null` | `"project"` = filter to current project only, `null` = show all |
| `limit` | integer | No | `50` | Max records to return (1-500) |
| `include_stats` | boolean | No | `true` | Include summary statistics |

### Example Usage

```json
{
  "project": "paaf",
  "search": "authentication",
  "source": "minna"
}
```

```json
{
  "project": "paaf",
  "entity_name": "scan_debt",
  "source": "kuzu"
}
```

### Example Output

```
Memory for paaf (2 sources):

KuzuDB (3 records):
  scan_debt     summary    "Scans project for technical debt..."
  scan_debt     qa         Q: "What does scan_debt return?" A: "A DebtReport..."

Minna (2 records):
  paaf          decision   "Switched from SQLite to KuzuDB for graph storage"
  paaf          gotcha     "ie_cypher uses entity_type not type"

Stats: 5 total records (3 kuzu, 2 minna)
```

---

## Cypher Cheat Sheet

The `ie_cypher` tool accepts read-only Cypher queries against the KuzuDB knowledge graph. Below are 10 common query patterns.

**Remember:** Always use `n.entity_type` (not `n.type`), and always include a `LIMIT` clause.

### 1. Find All Classes

```cypher
MATCH (n:Entity)
WHERE n.entity_type = 'class'
RETURN n.name, n.file, n.line_start
LIMIT 50
```

### 2. Functions by Complexity (Highest First)

```cypher
MATCH (n:Entity)
WHERE n.entity_type = 'function' AND n.complexity > 5
RETURN n.name, n.complexity, n.file
ORDER BY n.complexity DESC
LIMIT 20
```

### 3. Callers of a Function

```cypher
MATCH (caller:Entity)-[:CALLS]->(target:Entity)
WHERE target.name = 'authenticate_request'
RETURN caller.name, caller.file
LIMIT 20
```

### 4. Class Hierarchy (Inheritance)

```cypher
MATCH (child:Entity)-[:EXTENDS]->(parent:Entity)
RETURN child.name AS subclass, parent.name AS superclass, child.file
LIMIT 30
```

### 5. Dead Code (No Callers, Not Exported)

```cypher
MATCH (n:Entity)
WHERE n.entity_type = 'function'
  AND n.is_exported = false
  AND NOT EXISTS { MATCH (caller:Entity)-[:CALLS]->(n) }
RETURN n.name, n.file, n.line_start
LIMIT 50
```

### 6. Hub Entities (Most Connections)

```cypher
MATCH (n:Entity)-[r]-(m:Entity)
WITH n, count(r) AS connections
WHERE connections > 5
RETURN n.name, n.entity_type, connections, n.file
ORDER BY connections DESC
LIMIT 20
```

### 7. Imports Between Modules

```cypher
MATCH (a:Entity)-[:IMPORTS]->(b:Entity)
RETURN a.file AS importer, b.name AS imported, b.file AS from_file
LIMIT 30
```

### 8. Count Entities by Type

```cypher
MATCH (n:Entity)
RETURN n.entity_type AS type, count(n) AS count
ORDER BY count DESC
LIMIT 20
```

### 9. Functions in a Specific File

```cypher
MATCH (n:Entity)
WHERE n.file = 'src/analyzers/debt.py'
  AND n.entity_type = 'function'
RETURN n.name, n.line_start, n.complexity
ORDER BY n.line_start
LIMIT 50
```

### 10. Cross-Project Search (Shared DB Mode)

```cypher
MATCH (n:Entity)
WHERE n.name CONTAINS 'auth'
RETURN n.project, n.name, n.entity_type, n.file
ORDER BY n.project
LIMIT 30
```

---

## Quick Reference

| Tool | One-liner | Key params |
|------|-----------|------------|
| `ie_index` | Index a project for code intelligence | `project_path`, `mode` |
| `ie_status` | Check which projects are indexed | `project` (optional) |
| `ie_query` | Search within a single project | `project`, `query`, `method` |
| `ie_search_all` | Search across all projects | `query`, `top_k` |
| `ie_context` | Entity context: callers, callees, blast radius | `project`, `entity_name`, `depth` |
| `ie_cypher` | Raw Cypher queries on knowledge graph | `project`, `query` |
| `ie_quality` | Code quality composite score | `project` |
| `ie_health` | Dead code, cycles, hub analysis | `project` |
| `ie_detect_changes` | Blast radius of file changes | `project`, `files` |
| `ie_summarize` | AI summary for one entity | `project`, `entity_name` |
| `ie_batch_summarize` | AI summaries for all entities | `project`, `entity_types` |
| `ie_wiki` | Generate docs from graph | `project`, `scope` |
| `ie_global_analysis` | Cross-project architecture analysis | `include_clusters`, `include_health` |
| `ie_memory` | Unified KuzuDB + Minna memory browser | `project`, `search`, `source` |
