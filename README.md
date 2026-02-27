# Dramaturg: Research-Backed Design Exploration for Claude Code

In traditional theater, the dramaturg is the company's deep researcher — they dig into a play's historical context, interrogate its themes, and arm the director with the understanding needed to make informed creative decisions. But the best dramaturgs go further. In new-play development and the German theater tradition, the dramaturg doesn't just enrich what's already on the stage — they actively scout festivals, discover unknown works, and expand what the company even considers performing. They broaden the artistic vision.

The `dramaturg` skill brings both sides of this role to software design. It merges the natural, conversational flow of brainstorming with deep Gemini-powered research — then uses that foundation to proactively expand your design vision. Where brainstorming refines what you already have in mind and deep-plan validates and structures it, Dramaturg helps you discover what you didn't know you should be building.

## Dramaturg vs Brainstorming vs Deep-plan

| | Brainstorming | Deep-plan | Dramaturg |
|---|---|---|---|
| **Interaction** | Highly interactive, one question at a time | Structured stakeholder interviews | Natural discussion with live research woven in |
| **Research** | None — works from existing knowledge | Gemini-powered, drives the plan | Deep research integrated into conversation flow |
| **Vision scope** | Refines what you bring | Validates and structures what you bring | Actively expands — surfaces unexplored use-cases and features |
| **Output** | Design approval → writing-plans | Sectionized implementation plan | Design document → Arranger handoff |
| **Best for** | Quick designs with clear scope | Complex implementations needing thorough pre-analysis | Designs where the full scope isn't yet known |

## Where Dramaturg Fits

```
Dramaturg          Arranger           Conductor          Musician
(design vision) --> (plan phases) --> (orchestrate) ---> (perform)
                                         |
                                         └── Copyist extracts individual parts
```

The Dramaturg produces a design document that the Arranger consumes to create an executable implementation plan. Research-backed decisions marked VERIFIED in the decision journal let the Arranger skip redundant feasibility work. The user controls the pipeline — no automatic handoff after the design doc is complete.

## Usage

```
/dramaturg
```

Dramaturg verifies Gemini MCP availability on launch — the skill will not proceed without it. It then explores the current project (Context Grounding), draws out your design vision through open-ended conversation, proactively expands it with research-backed suggestions, and works through each design topic with deep investigation before compiling a final design document.

For resumed sessions, Dramaturg bootstraps from its decision journal — reading all settled decisions, identifying the current phase, and picking up where the previous session left off.

## How It Works

Dramaturg works through eight phases, each with a specific role. Phases are a structural backbone — transitions feel natural in conversation, not like formal gate checks. Loop-backs between phases route through the Phase Registry in SKILL.md, which re-bootstraps the Dramaturg with fresh framing before entering the next phase.

| # | Phase | Role |
|---|-------|------|
| 1 | Context Grounding | Project exploration before design discussion |
| 2 | Vision Loop | Establish What / Why / How-used from user input |
| 3 | Vision Expansion | Proactive enrichment — capabilities, interactions, edge cases |
| 4 | Broad Design Scoping | High-level framing questions (max 6 across all areas) |
| 5 | Approach Loop | Research-backed topic exploration and settlement |
| 6 | Review Loop | Per-section detailed design review |
| 7 | Reconciliation | Cross-section consistency check |
| 8 | Final Design Doc | Compile and output |

The Approach Loop (Phase 5) is where the Dramaturg spends the most time. A single topic's discussion may last 30-60+ minutes of back-and-forth research and exploration. Thoroughness here is the skill's primary value.

## Vision Expansion

Vision Expansion (Phase 3) is what makes Dramaturg unique. After the Vision Loop establishes what the user wants, the Dramaturg shifts from elicitation to enrichment — actively contributing design ideas rather than just capturing them.

Three sources generate enrichment suggestions:

1. **Dramaturg ideation** — interaction patterns the user hasn't mentioned, cross-feature implications from Context Grounding, edge cases and boundary conditions, domain-specific oversights
2. **Gemini creative ideation** — a structured prompt that builds on the Dramaturg's own ideas, bringing external perspective and possibilities
3. **Merge and curate** — deduplicate across both sources, then tier into strong recommendations, worth-asking, and remainder based on relevance to the user's stated goals

Suggestions are presented in three passes (strong → worth-asking → remainder) using AskUserQuestion for clean yes/no decisions with context. Brief discussion is allowed, but when it drifts into architectural detail, items are tabled for the Approach Loop.

Vision Expansion is mandatory after every Vision Loop pass, including loop-backs from later phases — the vision shift may have created new enrichment opportunities.

## Research Protocol

Gemini MCP is the Dramaturg's primary research tool and a hard requirement — the skill will not start without it. If Gemini fails mid-session, the Dramaturg pauses and asks: "Gemini is unavailable — want to wait, or continue without research on this topic?"

Research happens when triggered by specific conditions, not on a schedule:

| Trigger | Behavior |
|---------|----------|
| Dramaturg recommends an approach | Research FIRST, then present with evidence |
| User proposes with clear intent | Research immediately to validate feasibility |
| User proposes with ambiguous intent | Clarify intent first, then ask about research |
| Unimplemented protocol or pattern | Research through Gemini before incorporating |
| Trivial decisions, clarification exchanges | No research needed |

Tool hierarchy for research:
1. **Gemini MCP** — primary for feasibility, technical questions, approach analysis
2. **Brave-search** — discovering what exists (libraries, articles, community sentiment)
3. **Explore subagents** — codebase understanding, architectural analysis
4. **WebFetch** — reading specific known URLs
5. **General-purpose subagents** — multi-source synthesis, speculative research

Research should over-investigate, not under-investigate. The Dramaturg provides explicit recommendations, not just findings — following the priority chain: compatibility > reliability > efficiency > security > performance.

## Decision Journal

The decision journal is the Dramaturg's progressive external memory — an append-only log at `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md` that captures decisions, research findings, and vision context as the session progresses.

**Why it exists:**
- **Context preservation** — settled decisions survive context compaction
- **Session resilience** — if a session dies mid-Approach-Loop, a new session reads the journal and picks up where it left off
- **Pipeline handoff** — entries marked VERIFIED let the Arranger skip re-verification; PARTIAL flags what still needs checking
- **Audit trail** — shows how the design evolved, not just the final answer

**Entry types:**
- **User-discussed** — settled through conversation. Categorized as goal, use-case, or decision. User input captured verbatim, not paraphrased.
- **Research-backed** — validated through investigation. Includes tools used, findings, and an Arranger note (VERIFIED / PARTIAL / UNRESEARCHED).
- **Tension** — productive conflicts between requirements (e.g., "offline AND real-time"). Status is "acknowledged" not "settled" — the Arranger receives tensions as design context.

Journal entries are written at every phase exit checkpoint: Vision Baseline (Phase 2), Vision Expansion results (Phase 3), Topic Map (Phase 4), every settled topic (Phase 5), and every approved section (Phase 6).

## Outputs and Handoff

The Dramaturg produces two artifacts:

1. **Design document** — `docs/plans/designs/YYYY-MM-DD-<topic>-design.md`. An open-ended narrative capturing the design vision. Required elements: a Goals section (opening) with verbose, context-rich statements of the user's goals, and an Arranger Notes appendix (closing) flagging new protocols, open questions, and key design decisions for the planner.

2. **Decision journal** — `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md`. The structured decision trail described above. Archived to `docs/archive/` after the design doc is finalized.

After compilation, the Dramaturg informs the user: "The design is complete. When you're ready, invoke the Arranger to create an implementation plan from this design."

## Guardrails

| Rule | Detail |
|------|--------|
| One question at a time | Never multiple open-ended questions in a single message. Tight, bounded questions may be batched |
| One section at a time | Review Loop presents one design section per message |
| Scope protection | Redirects implementation details (file paths, function signatures, package versions) to architecture-level language |
| Priority chain | compatibility > reliability > efficiency > security > performance |
| No output truncation | Thorough reasoning is the skill's value — never compress to fit artificial limits |
| Mandatory Vision Expansion | Non-skippable after every Vision Loop pass, including loop-backs from later phases |
| Vision Loop gate | Strict: must answer What / Why / How-used from user input alone before proceeding |
| Research before recommendation | Never recommend based solely on training data for substantive technical decisions |

## Session Management

The Approach Loop (Phase 5) is the most context-intensive phase. When context pressure builds, the Dramaturg evaluates a session split at the Phase 5→6 boundary — completing approach exploration in one session, then starting the Review Loop fresh in a new session.

When starting a new session (fresh or post-split), the Dramaturg bootstraps from the decision journal:

1. Read the journal — the authoritative record of all settled decisions
2. Identify the current phase from the most recent entry type
3. Verify completeness — all expected entries exist for completed phases
4. Summarize for the user — what's settled and where the session picks up

If the journal is missing or corrupted, the Dramaturg reconstructs from conversation context if available. If both are unavailable, it starts fresh from the earliest unverifiable phase.

## Project Structure

```
dramaturg/
├── skill/
│   ├── SKILL.md                          # Skill definition (entry point)
│   └── references/
│       ├── support-phases.md             # Phases 1, 4, 7, 8
│       ├── vision-loop.md               # Phase 2
│       ├── vision-expansion.md          # Phase 3 (enrichment and proactive ideation)
│       ├── approach-loop.md             # Phase 5 (core research-discuss-settle loop)
│       ├── review-loop.md              # Phase 6
│       └── research-strategy.md        # Research tool hierarchy and delegation
└── docs/
    ├── archive/                         # Historical docs
    ├── designs/                         # Skill design specifications
    └── working/                         # Active design work
```

## Requirements

- **Gemini MCP** (mandatory — skill will not start without it)
- **Claude Code** with skill/plugin support

## Origin

Part of [The Elevated Stage](https://github.com/The-Elevated-Stage) orchestration system. Design doc: `docs/designs/2026-02-17-dramaturg-skill-design.md`.
