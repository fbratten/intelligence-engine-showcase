---
title: Real-World Use Cases
nav_order: 1
---

# Real-World Use Cases

Intelligence Engine goes beyond simple code search. These are production-tested patterns from a 54-project codebase.

---

## Multi-Agent Development Cycle

**Scenario:** Three AI agents (Claude Code, Nelly, Agatha) work on the same codebase. Each needs to understand what the others changed without direct communication.

**How IE solves it:**

1. **Post-commit indexing** -- After each push, `ie_index` updates the knowledge graph incrementally (git diff-based, only re-parses changed files)
2. **Pre-coding search** -- Before writing new code, an agent runs `ie_query` to find related implementations: "What patterns exist for message routing?" returns ranked results across all modules
3. **Blast radius check** -- Before refactoring a shared utility, `ie_context` reveals every downstream caller: "If I change `Coordinator.route()`, which 47 functions break?"
4. **Health gate** -- After a sprint, `ie_health` catches dead code introduced by parallel work: "Agent A deleted the caller but Agent B didn't know -- 12 functions are now unreachable"

**Cypher query -- Find entities modified by multiple agents:**

```cypher
MATCH (n:Entity)
WHERE n.entity_type = 'function'
  AND n.file CONTAINS 'bridges/'
RETURN n.name, n.file, n.complexity
ORDER BY n.complexity DESC
LIMIT 20
```

**Key insight:** The knowledge graph becomes the shared context that replaces the need for agents to communicate about code structure. Each agent queries the same truth.

---

## Test Coverage Gap Discovery

**Scenario:** A project has 87% line coverage according to pytest-cov, but critical business logic paths are untested. Traditional coverage tools can't see structural gaps.

**How IE finds what coverage tools miss:**

1. **Find untested high-complexity functions:**

```cypher
MATCH (f:Entity)
WHERE f.entity_type = 'function'
  AND f.complexity > 5
  AND NOT EXISTS {
    MATCH (t:Entity)-[:CALLS]->(f)
    WHERE t.file CONTAINS 'test'
  }
RETURN f.name, f.file, f.complexity
ORDER BY f.complexity DESC
```

This returns functions with cyclomatic complexity > 5 that have **no callers from test files**. Line coverage might say these functions are "covered" because they're called indirectly through a wrapper -- but no test directly exercises their branch logic.

2. **Find classes with no test counterpart:**

```cypher
MATCH (c:Entity)
WHERE c.entity_type = 'class'
  AND NOT EXISTS {
    MATCH (t:Entity)
    WHERE t.entity_type = 'class'
      AND t.name STARTS WITH 'Test'
      AND t.name CONTAINS c.name
  }
RETURN c.name, c.file
```

3. **Identify orphan branches** using `ie_health`:

```
Dead Code: 239 entities
  - 142 functions never called from test or production code
  - 97 variables assigned but never read
```

**Key insight:** IE understands *structural* relationships between production code and test code. A function that's reachable via 5 layers of indirection might have 100% line coverage but 0% direct test coverage. IE reveals the direct-call gap.

---

## Post-Merge Impact Analysis

**Scenario:** A large PR merges 14 files touching the transport layer. QA needs to know which downstream systems are affected before signing off.

**How IE calculates blast radius:**

1. **Run change detection:**

```bash
# Via MCP
ie_detect_changes --project agent-comm
```

Returns:
```
Changed files: 14
New entities: 3
Modified entities: 22
Deleted entities: 5

Blast radius:
  Direct callers affected:  47
  2-hop callers affected:   189
  Test files to re-run:     12
```

2. **Trace specific impact chains:**

```cypher
MATCH path = (changed:Entity)-[:CALLS*1..3]->(downstream:Entity)
WHERE changed.name = 'HTTPTransport'
RETURN [n IN nodes(path) | n.name] AS call_chain,
       length(path) AS depth
ORDER BY depth
LIMIT 50
```

3. **Find affected test files:**

```cypher
MATCH (f:Entity)-[:CALLS*1..3]->(changed:Entity)
WHERE changed.name IN ['send_message', 'route_message', 'get_transport']
  AND f.file CONTAINS 'test'
RETURN DISTINCT f.file AS test_file
```

**Key insight:** What takes a human hours of "find references" clicking in an IDE, IE answers in milliseconds. The graph already has every call chain pre-computed.

---

## Cross-Project Dependency Audit

**Scenario:** Updating a shared library (`mem-system-lite-mcp` v2.0) that 8 projects depend on. Need to know exactly what will break.

**How IE maps the dependency surface:**

1. **Find all projects importing the library:**

```cypher
MATCH (n:Entity)
WHERE n.entity_type = 'module'
  AND n.name CONTAINS 'mem_system'
RETURN DISTINCT n.project, n.name, n.file
ORDER BY n.project
```

2. **Find specific API usage per project:**

```cypher
MATCH (caller:Entity)-[:CALLS]->(api:Entity)
WHERE api.file CONTAINS 'mem_system_lite'
RETURN caller.project, caller.name, api.name AS api_used
ORDER BY caller.project
```

3. **Use `ie_search_all` for semantic discovery:**

Search for "memory store recall" across all 54 projects to find projects that use memory patterns even without direct imports (e.g., they vendored an older version).

**Key insight:** No more grepping across dozens of repos. The shared knowledge graph already has the complete dependency picture for all 54 projects.

---

## Dead Code Cleanup Sprint

**Scenario:** Technical debt has accumulated over 6 months. The codebase has functions nobody calls, classes nobody instantiates, and imports nobody uses.

**How IE guides the cleanup:**

1. **Get the full dead code inventory:**

```bash
ie_health --project my-project
```

```
Dead Code: 239 entities
  Functions with 0 callers:   142
  Classes with 0 references:  23
  Modules never imported:     8
  Variables never read:       66
```

2. **Prioritize by complexity (remove complex dead code first):**

```cypher
MATCH (f:Entity)
WHERE f.entity_type = 'function'
  AND NOT EXISTS { MATCH ()-[:CALLS]->(f) }
RETURN f.name, f.file, f.complexity, f.lines
ORDER BY f.complexity DESC
LIMIT 20
```

3. **Verify before deleting** -- check that "dead" code isn't dynamically dispatched:

```cypher
MATCH (f:Entity)
WHERE f.name = 'handle_webhook'
  AND NOT EXISTS { MATCH ()-[:CALLS]->(f) }
RETURN f.name, f.docstring
```

If the docstring says "Called by Flask route decorator" -- it's not truly dead, just not statically reachable.

**Key insight:** IE gives you the hit list, but human judgment decides what's safe to remove. The combination of graph analysis + AI summarization (via `ie_summarize`) provides the context needed for each decision.

---

## Architecture Evolution Tracking

**Scenario:** Over 3 months, the codebase grew from 200 to 1200 entities. Did the architecture stay clean or did it accumulate coupling?

**How IE tracks structural evolution:**

1. **Hub concentration over time** -- `ie_health` shows whether code is centralizing around a few god objects:

```
Hub Concentration: 12.0%
  Top 5 hubs handle 12% of all connections
  main():        44 connections
  Coordinator:   38 connections
  create_app():  18 connections
```

2. **Community modularity** -- The UI's Community view shows whether modules are becoming more or less independent:

```
14 communities detected
Modularity: 0.741  (> 0.4 is good, > 0.7 is excellent)
```

3. **Quality score trending** via `ie_quality`:

```
Composite Quality Score: 69.1
  Avg Complexity:      1.6 (good)
  Doc Coverage:        39.6% (needs work)
  Dead Code Ratio:     62.1% (too high)
```

**Key insight:** Structural metrics complement functional testing. A project can have 100% test coverage and still be architecturally fragile if hub concentration is 40% or modularity drops below 0.3.
