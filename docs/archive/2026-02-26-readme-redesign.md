# Dramaturg README Redesign

**Date:** 2026-02-26
**Goal:** Rewrite Dramaturg's README to match the Lethe/Copyist README format — rich intro with theatrical metaphor, three-way comparison table, and full operational documentation.

## Approach

**Approach B: Classical Evolved** — Open with the traditional dramaturg role (deep researcher, probing questions), then pivot to the modern evolution (creative curator, vision expander). Maps the merge of brainstorming's interactivity with deep-plan's research, and the unique Vision Expansion capability.

## Intro Design

### Title

```
# Dramaturg: Research-Backed Design Exploration for Claude Code
```

### Opening Paragraphs

Two paragraphs, Lethe-depth:

1. **Theater metaphor** — Traditional dramaturg as deep researcher who arms the director with understanding. Pivot: best dramaturgs go further — scout festivals, discover unknown works, expand what the company considers performing. They broaden the artistic vision.

2. **Skill mapping** — Merges conversational flow of brainstorming with Gemini-powered research. Uses that foundation to proactively expand design vision. Where brainstorming refines what you already have and deep-plan validates and structures it, Dramaturg helps discover what you didn't know you should be building.

### Three-Way Comparison Table

| | Brainstorming | Deep-plan | Dramaturg |
|---|---|---|---|
| **Interaction** | Highly interactive, one question at a time | Structured stakeholder interviews | Natural discussion with live research woven in |
| **Research** | None — works from existing knowledge | Gemini-powered, drives the plan | Deep research integrated into conversation flow |
| **Vision scope** | Refines what you bring | Validates and structures what you bring | Actively expands — surfaces unexplored use-cases and features |
| **Output** | Design approval → writing-plans | Sectionized implementation plan | Design document → Arranger handoff |
| **Best for** | Quick designs with clear scope | Complex implementations needing thorough pre-analysis | Designs where the full scope isn't yet known |

## Section Structure

### 3. Where Dramaturg Fits

Pipeline diagram showing: Dramaturg → Arranger → Conductor → Musician, with Copyist extracting parts. Note that user controls the pipeline — no automatic handoff.

### 4. Usage

- `/dramaturg` invocation
- What happens: Gemini availability check, Context Grounding, then conversational design exploration through 8 phases
- Session resume via decision journal

### 5. How It Works

8-phase table:

| # | Phase | Role |
|---|-------|------|
| 1 | Context Grounding | Project exploration before design discussion |
| 2 | Vision Loop | Establish What/Why/How-used from user input |
| 3 | Vision Expansion | Proactive enrichment — capabilities, interactions, edge cases |
| 4 | Broad Design Scoping | High-level framing questions (max 6) |
| 5 | Approach Loop | Research-backed topic exploration and settlement |
| 6 | Review Loop | Per-section detailed design review |
| 7 | Reconciliation | Cross-section consistency check |
| 8 | Final Design Doc | Compile and output |

Brief paragraph: phases are a structural backbone, transitions feel natural in conversation (not "PHASE 2 COMPLETE"). Loop-backs between phases route through the Phase Registry.

### 6. Vision Expansion

Highlight the unique feature. Three enrichment sources:
1. Dramaturg ideation (interaction patterns, cross-feature implications, edge cases, domain oversights)
2. Gemini creative ideation (structured prompt building on Dramaturg's ideas)
3. Merge and curate (deduplicate, tier into strong recommendations / worth asking / remainder)

Three-pass presentation: strong recommendations first, then worth-asking, then remainder. Each presented via AskUserQuestion for clean yes/no decisions.

### 7. Research Protocol

- Gemini MCP is mandatory — skill will not proceed without it
- Fallback: if Gemini fails mid-session, pauses and asks user
- Tool hierarchy: Gemini → Brave-search → Explore subagents → WebFetch → General-purpose subagents
- Trigger rules: research first when recommending, validate immediately when user proposes with clear intent, clarify first when intent is ambiguous
- Research should over-investigate, not under-investigate

### 8. Decision Journal

Purpose: context preservation across compaction, session resilience, pipeline handoff (VERIFIED entries let Arranger skip re-verification), audit trail.

Entry types:
- User-discussed (goal / use-case / decision categories, verbatim user quotes)
- Research-backed (tools used, findings, VERIFIED/PARTIAL/UNRESEARCHED arranger notes)
- Tension (productive conflicts between requirements)

Checkpoint triggers at each phase exit.

### 9. Outputs and Handoff

Two artifacts:
1. Design document — `docs/plans/designs/YYYY-MM-DD-<topic>-design.md`. Open-ended narrative. Required: Goals section (opening), Arranger Notes appendix (closing).
2. Decision journal — `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md`. Append-only decision trail.

Journal archived to `docs/archive/` after design doc is finalized. Handoff: user invokes Arranger manually.

### 10. Guardrails

Table format:

| Rule | Detail |
|------|--------|
| One question at a time | Never multiple open-ended questions in a single message |
| One section at a time | Review Loop presents one design section per message |
| Scope protection | Redirects implementation details to architecture-level language |
| Priority chain | compatibility > reliability > efficiency > security > performance |
| No output truncation | Thorough reasoning is the skill's value — never compress to fit artificial limits |
| Mandatory Vision Expansion | Non-skippable after every Vision Loop pass, including loop-backs |
| Vision Loop gate | Strict transition: must answer What/Why/How-used from user input alone |

### 11. Session Management

- Session split at Phase 5→6 boundary when context pressure is high
- Bootstrap from decision journal: read journal, identify current phase, verify completeness, summarize for user
- Journal is authoritative record — survives context compaction and session interruption

### 12. Project Structure

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

### 13. Requirements

- Gemini MCP (mandatory — skill will not start without it)
- Claude Code with skill support

### 14. Origin

Part of The Elevated Stage orchestration system. Link to GitHub org. Design doc reference.

## Source Conflicts to Resolve During Implementation

From the deep-dive audit, two inconsistencies need resolution before or during README writing:

1. **Journal archival path** — SKILL.md says `docs/archive/` alongside design doc. Support-phases says `docs/archive/plans/designs/...` while design doc stays in `docs/plans/designs/`. Pick one.

2. **Artifact count** — SKILL.md preamble says two artifacts (design doc + journal, Arranger Notes is part of design doc). Support-phases context says three artifacts including Arranger Notes appendix. SKILL.md is authoritative — README should say two artifacts.

## Definition of Done

- README reflects all 8 phases including Vision Expansion
- README includes `/dramaturg` usage and Gemini prerequisite
- README includes artifact lifecycle and handoff behavior
- README project structure matches actual repository layout
- No contradictory statements vs SKILL.md and reference files
- Three-way comparison table positions Dramaturg against brainstorming and deep-plan
- Intro uses Approach B theatrical metaphor at Lethe-level depth
