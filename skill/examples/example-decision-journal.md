<skill name="dramaturg-example-decision-journal" version="1.0">

<metadata>
type: example
parent-skill: dramaturg
tier: 3
</metadata>

<sections>
- scenario
- vision-baseline-entry
- vision-expansion-entry
- topic-map-entry
- decision-entry
- research-entry
- tension-entry
- section-approval-entry
</sections>

<section id="scenario">
<context>
# Example: Decision Journal Entries

This example shows each journal entry type the Dramaturg produces. Same scenario as the design document example: task management app, DAG dependency system.

The decision journal lives at `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md` and is consumed by the Arranger alongside the design document. Entries are append-only — never edited after writing.
</context>
</section>

<section id="vision-baseline-entry">
<context>
## Vision Baseline (Phase 2 Exit)

Written once after the Vision Loop gate passes. This is the regression test anchor for Phase 6.

```markdown
## Vision Baseline
**Phase:** Phase 2 — Vision Loop
**What:** A DAG-based task dependency system where tasks can block other tasks,
with graph visualization showing blocking chains, automatic unblock notifications,
and priority inheritance that surfaces blocking tasks as urgent when they block
high-priority work.
**Why:** Visibility into why tasks are blocked and what needs to happen to unblock
them. The user's current workflow requires manually tracing dependency chains across
20-30 interconnected tasks, which is error-prone and time-consuming.
**How used:** User creates dependencies when defining tasks ("this task depends on
these three tasks"). When viewing a blocked task, sees a graph showing the full
blocking chain. When a blocking task completes, dependent task owners get notified.
High-priority blocked tasks make their blockers feel urgent via priority inheritance.
**User confirmed:** yes
**Research during vision:** Verified DAG visualization feasibility for 20-30 node
graphs — confirmed viable with standard graph layout algorithms.
**Status:** settled
```
</context>
</section>

<section id="vision-expansion-entry">
<context>
## Vision Expansion (Phase 3 Exit)

Written after the enrichment discussion. Captures what was added and what the user accepted or declined.

```markdown
## Vision Expansion
**Phase:** Phase 3 — Vision Expansion
**Enrichments presented:** 8 suggestions (4 from own analysis, 4 from Gemini)
**Accepted:**
- Dependency health dashboard showing overall project bottleneck analysis
- "What-if" mode: simulate completing a task to see what unblocks
- Slack integration for unblock notifications (in addition to in-app)
**Declined:**
- Auto-scheduling based on dependency graph (user: "too much magic")
- Critical path highlighting (user: "nice but not essential for v1")
- Dependency templates for recurring project types (user: "maybe later")
**Deferred:** Cross-project dependencies (user interested but out of scope for
this design)
**Vision baseline still valid:** yes — enrichments extend the vision, no regression
**Status:** settled
```
</context>
</section>

<section id="topic-map-entry">
<context>
## Topic Map (Phase 4 Exit)

Written after Broad Design Scoping confirms coverage. This is the authoritative list for Phase 5.

```markdown
## Topic Map
**Phase:** Phase 4 — Broad Design Scoping
**Areas to explore in Phase 5:**
- Graph data model — how dependencies are stored and queried
- Dependency resolution — algorithm for determining blocked/unblocked status
- Priority inheritance — how priority propagates through the graph
- Cycle prevention — how circular dependencies are detected and prevented
- Visualization — graph rendering approach for 20-30 node DAGs
- Unblock notifications — trigger mechanism, delivery channels, batching
- "What-if" simulation — how to model hypothetical task completion
**User confirmed coverage:** yes
**Status:** confirmed
```
</context>
</section>

<section id="decision-entry">
<context>
## User-Discussed Decision Entry (Phase 5)

For topics settled through conversation. Note the verbatim capture and Category tag.

```markdown
## Decision: Graph Structure
**Phase:** Phase 5 — Approach Loop
**Category:** decision
**Decided:** Use a DAG (directed acyclic graph) over a simpler tree structure.
Tasks can have multiple dependencies and multiple dependents.
**User verbatim:** "I definitely need multi-dependency support. Like, a release
task might depend on both QA passing and docs being written. A tree wouldn't
let me express that."
**User context:** The user's projects frequently have convergence points where
multiple streams of work must complete before a milestone. Tree structures would
force artificial linearization.
**Alternatives discussed:** Tree structure (simpler but can't express convergence),
flat dependency list without graph semantics (no transitivity). Both rejected
because the user's core use case requires multi-parent relationships.
**Status:** settled
**Supersedes:** —
```

### Goal Entry Variant

When the user states a goal (not a technical decision), use Category: goal. Goals are inviolable for downstream skills.

```markdown
## Decision: Immediate Blocking Visibility
**Phase:** Phase 5 — Approach Loop
**Category:** goal
**Decided:** When viewing any blocked task, the user must immediately see the full
chain of what blocks it and what needs to happen to unblock it.
**User verbatim:** "I want to look at a blocked task and instantly know why it's
stuck and what I need to do about it. No clicking through three screens."
**User context:** Current workflow requires opening each blocking task individually
to understand the chain. With 20-30 tasks this takes significant time.
**Alternatives discussed:** N/A — this is a stated goal, not a technical choice.
**Status:** settled
**Supersedes:** —
```
</context>
</section>

<section id="research-entry">
<context>
## Research-Backed Entry (Phase 5)

For topics validated through Gemini or other research tools. Note the Arranger note field.

```markdown
## Research: DAG Cycle Detection
**Phase:** Phase 5 — Approach Loop
**Question:** What's the most practical approach for preventing circular
dependencies in a task DAG at creation time?
**Tools used:** gemini-search, gemini-query
**Findings:** Kahn's algorithm (topological sort) is the standard approach.
O(V+E) complexity, well-suited for graphs under 1000 nodes. For real-time
validation on edge creation, incremental cycle detection (DFS from the new
edge's target back to its source) is more efficient than full topological
sort — O(V+E) worst case but typically much faster for single-edge validation.
Libraries like networkx (Python) and graphlib (stdlib) provide built-in
implementations.
**Decision:** Use incremental DFS cycle detection on dependency creation.
Full topological sort reserved for batch validation (e.g., import).
**Arranger note:** VERIFIED — standard algorithm, multiple implementations
available. Arranger can skip re-audit of the algorithmic approach.
**Status:** settled
**Supersedes:** —
```

### PARTIAL Research Example

When research validates the approach but leaves gaps:

```markdown
**Arranger note:** PARTIAL — Arranger should verify: whether the chosen graph
library supports incremental edge addition without full rebuild. Research
confirmed the algorithm but not library-specific API availability.
```
</context>
</section>

<section id="tension-entry">
<context>
## Tension Entry

For acknowledged tradeoffs that don't have clean resolutions. These are informational — the Arranger knows about the tension.

```markdown
## Tension: Real-Time Graph Updates vs. Query Performance
**Phase:** Phase 5 — Approach Loop
**Requirements in tension:**
1. User wants to see dependency graph updates immediately when tasks change status
2. Priority inheritance requires traversing the full upstream graph on every status change
**Why they conflict:** Real-time graph traversal on every status change could
become expensive for deep graphs. Caching the traversal results introduces staleness.
**Current resolution approach:** Eager computation for graphs under 50 nodes
(covers the user's stated 20-30 task range). Lazy computation with cache
invalidation for larger graphs. The threshold is a design parameter the Arranger
can tune during implementation.
**Status:** acknowledged
```
</context>
</section>

<section id="section-approval-entry">
<context>
## Section Approval Entry (Phase 6)

Written for each design section approved during the Review Loop.

```markdown
## Section Approved: Data Model
**Phase:** Phase 6 — Review Loop
**Key decisions reflected:** Decision: Graph Structure, Research: DAG Cycle Detection
**User feedback incorporated:** Added explicit mention of soft-delete for
dependencies (user: "I might want to temporarily disable a dependency without
deleting it")
**Status:** settled
```
</context>
</section>

</skill>
