<skill name="dramaturg-example-design-document" version="1.0">

<metadata>
type: example
parent-skill: dramaturg
tier: 3
</metadata>

<sections>
- scenario
- goals-section
- document-body
- arranger-notes
</sections>

<section id="scenario">
<context>
# Example: Design Document Structure

This example shows the expected structure of a Dramaturg design document. The scenario is a task management app adding a DAG-based dependency system. The content is abbreviated — a real design doc would be more detailed in each section.

The design document lives at `docs/plans/designs/YYYY-MM-DD-<topic>-design.md` and is consumed by the Arranger to produce an implementation plan.
</context>
</section>

<section id="goals-section">
<context>
## Goals Section (Opening — Required)

The goals section is verbose and narrative — not a bullet list. It captures everything the user wants, in enough detail that the Arranger could reconstruct the user's intent without reading the rest of the document.

```markdown
# Task Dependency System Design

## Goals

The user wants to add a dependency system to their existing task management app,
where tasks can block other tasks in a directed acyclic graph (DAG) structure.
The core motivation is visibility — when a task is blocked, the user wants to
immediately see *why* it's blocked and *what* needs to happen to unblock it,
without manually tracing dependency chains.

The system should support both simple linear chains ("Task B depends on Task A")
and complex multi-dependency scenarios ("Task D depends on Tasks A, B, and C all
being complete"). The user specifically mentioned wanting to visualize these
dependencies as a graph, not just a flat list, because their projects frequently
have 20-30 interconnected tasks where the blocking relationships aren't obvious
from titles alone.

Notifications are part of the vision: when a blocking task is completed, the
owners of newly-unblocked tasks should be notified. The user described a scenario
where three team members are each waiting on different prerequisite tasks, and
completing one task should automatically notify the right people that their work
is now unblocked.

The user wants the dependency system to integrate with their existing priority
system — a blocked task's effective priority should reflect the priorities of the
tasks blocking it, so that high-priority blocked work surfaces the blocking tasks
as urgent. They described this as "priority inheritance" — if Task D is critical
and Task A blocks it, Task A should feel urgent even if it was originally marked
as low priority.
```

Key properties of this goals section:
- Rich with context, not terse
- Uses the user's own framing ("priority inheritance")
- Describes specific behaviors and scenarios
- Independent of technical decisions — no mention of graph libraries, database schemas, or API designs
- Self-contained — readable without the rest of the document
</context>
</section>

<section id="document-body">
<context>
## Document Body (Open-Ended)

No rigid template. Sections emerge from the design discussion. A typical structure:

```markdown
## Data Model

The dependency system uses a DAG represented as an adjacency list...
[Architecture-level description of the data model, not column names or types]

## Dependency Resolution

When a task is marked complete, the system traverses the dependency graph...
[How the system determines what becomes unblocked]

## Priority Inheritance

Effective priority propagates through the dependency graph...
[The algorithm at design level — "walks the graph upstream" not "recursive CTE query"]

## Visualization

The graph visualization presents the user's task dependencies as a node-edge diagram...
[What the user sees, interaction patterns, not framework choices]

## Notifications

Unblock notifications are triggered when dependency resolution frees a task...
[When notifications fire, who receives them, what information they contain]

## Cycle Prevention

The system must prevent circular dependencies at creation time...
[Design-level constraint — validated on edge creation, not implementation details]
```

Key properties:
- Sections map to design topics, not code modules
- Architecture-level language — "adjacency list" not "junction table with foreign keys"
- Captures *what* and *why*, not *how it's coded*
- Inferences are marked: "Based on the user's goal of immediate visibility, this implies real-time graph updates rather than polling"
</context>
</section>

<section id="arranger-notes">
<context>
## Arranger Notes Appendix (Closing — Required)

```markdown
## Arranger Notes

### New Protocols / Unimplemented Patterns
- DAG cycle detection — not currently in the codebase. Research entry "DAG Cycle
  Detection Approaches" found that Kahn's algorithm is the standard approach for
  this scale. (Journal: Research: DAG Cycle Detection)
- Priority inheritance propagation — novel pattern for this codebase. Recursive
  graph traversal with memoization. (Journal: Research: Priority Propagation)

### Open Questions
- Maximum graph depth — user said "20-30 tasks" but didn't specify max chain
  length. May need a depth limit for performance.
- Notification batching — if completing one task unblocks 5 others, should
  notifications be batched or individual? Not discussed during design.

### Key Design Decisions
- DAG over tree structure — supports multi-dependency scenarios the user described
  (Journal: Decision: Graph Structure)
- Priority inheritance uses max-of-dependents, not sum — prevents priority
  explosion in deep graphs (Journal: Decision: Priority Algorithm)
- Cycle prevention at creation time, not async validation — user wants immediate
  feedback (Journal: Decision: Cycle Prevention Timing)
```

Key properties:
- References specific journal entries by title
- Open questions are genuinely unresolved, not hand-waving
- Key decisions point to the architecturally significant choices, not all choices
</context>
</section>

</skill>
