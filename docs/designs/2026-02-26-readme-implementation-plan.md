# Dramaturg README Redesign — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Rewrite Dramaturg's README to match the Lethe/Copyist format with rich theatrical intro, three-way comparison table, and full operational documentation.

**Architecture:** Single-file rewrite of `README.md`. All content sourced from existing `skill/SKILL.md` and `skill/references/*.md` — no new information, just restructured for the README audience. SKILL.md is authoritative; where docs disagree, SKILL.md wins.

**Design doc:** `docs/designs/2026-02-26-readme-redesign.md`

**Reference files for cross-checking:**
- `skill/SKILL.md` — authoritative runtime contract
- `skill/references/vision-expansion.md` — Phase 3 details
- `skill/references/approach-loop.md` — Phase 5 details
- `skill/references/research-strategy.md` — tool hierarchy
- `skill/references/support-phases.md` — Phases 1, 4, 7, 8

**Style references:**
- `/home/kyle/claude/skills_staged/kyle-skills/lethe/README.md` — intro depth, section structure
- `/home/kyle/claude/skills_staged/orchestra/copyist/README.md` — pipeline diagram, guardrails table, origin section

---

### Task 1: Write the Intro Hook

**Files:**
- Modify: `README.md` (full rewrite, starting from line 1)

**Step 1: Write title and opening paragraphs**

Replace the entire README with the intro hook. Title line with subtitle, two opening paragraphs using Approach B (Classical Evolved) theatrical metaphor:

```markdown
# Dramaturg: Research-Backed Design Exploration for Claude Code

In traditional theater, the dramaturg is the company's deep researcher — they dig into a play's historical context, interrogate its themes, and arm the director with the understanding needed to make informed creative decisions. But the best dramaturgs go further. In new-play development and the German theater tradition, the dramaturg doesn't just enrich what's already on the stage — they actively scout festivals, discover unknown works, and expand what the company even considers performing. They broaden the artistic vision.

The `dramaturg` skill brings both sides of this role to software design. It merges the natural, conversational flow of brainstorming with deep Gemini-powered research — then uses that foundation to proactively expand your design vision. Where brainstorming refines what you already have in mind and deep-plan validates and structures it, Dramaturg helps you discover what you didn't know you should be building.
```

**Step 2: Write three-way comparison table**

```markdown
## Dramaturg vs Brainstorming vs Deep-plan

| | Brainstorming | Deep-plan | Dramaturg |
|---|---|---|---|
| **Interaction** | Highly interactive, one question at a time | Structured stakeholder interviews | Natural discussion with live research woven in |
| **Research** | None — works from existing knowledge | Gemini-powered, drives the plan | Deep research integrated into conversation flow |
| **Vision scope** | Refines what you bring | Validates and structures what you bring | Actively expands — surfaces unexplored use-cases and features |
| **Output** | Design approval → writing-plans | Sectionized implementation plan | Design document → Arranger handoff |
| **Best for** | Quick designs with clear scope | Complex implementations needing thorough pre-analysis | Designs where the full scope isn't yet known |
```

**Step 3: Verify intro content**

Cross-check against:
- `skill/SKILL.md:45-46` — preamble description matches
- `skill/SKILL.md:69-76` — scope boundaries (brainstorming/deep-plan positioning) are consistent
- Design doc approach B description

**Step 4: Commit**

```bash
git add README.md
git commit -m "docs: rewrite README intro with theatrical metaphor and comparison table"
```

---

### Task 2: Write Pipeline, Usage, and How It Works Sections

**Files:**
- Modify: `README.md` (append after intro)

**Step 1: Write "Where Dramaturg Fits" section**

Source: `skill/SKILL.md:49-57` (pipeline position)

```markdown
## Where Dramaturg Fits

```
Dramaturg          Arranger           Conductor          Musician
(design vision) --> (plan phases) --> (orchestrate) ---> (perform)
                                         |
                                         └── Copyist extracts individual parts
```

The Dramaturg produces a design document that the Arranger consumes to create an executable implementation plan. Research-backed decisions marked VERIFIED in the decision journal let the Arranger skip redundant feasibility work. The user controls the pipeline — no automatic handoff after the design doc is complete.
```

**Step 2: Write "Usage" section**

Source: `skill/SKILL.md:47-48` (Gemini check), `skill/SKILL.md:6` (version)

```markdown
## Usage

```
/dramaturg
```

Dramaturg verifies Gemini MCP availability on launch — the skill will not proceed without it. It then explores the current project (Context Grounding), draws out your design vision through open-ended conversation, proactively expands it with research-backed suggestions, and works through each design topic with deep investigation before compiling a final design document.

For resumed sessions, Dramaturg bootstraps from its decision journal — reading all settled decisions, identifying the current phase, and picking up where the previous session left off.
```

**Step 3: Write "How It Works" section**

Source: `skill/SKILL.md:82-99` (phase registry)

```markdown
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
```

**Step 4: Verify against SKILL.md**

Cross-check:
- Phase names and numbering match `skill/SKILL.md:89-99`
- Pipeline description matches `skill/SKILL.md:49-57`
- Gemini requirement matches `skill/SKILL.md:25`
- Session bootstrap matches `skill/SKILL.md:459-464`

**Step 5: Commit**

```bash
git add README.md
git commit -m "docs: add pipeline, usage, and how-it-works sections"
```

---

### Task 3: Write Vision Expansion and Research Protocol Sections

**Files:**
- Modify: `README.md` (append after How It Works)

**Step 1: Write "Vision Expansion" section**

Source: `skill/references/vision-expansion.md` §enrichment-sources and §three-pass-presentation

```markdown
## Vision Expansion

Vision Expansion (Phase 3) is what makes Dramaturg unique. After the Vision Loop establishes what the user wants, the Dramaturg shifts from elicitation to enrichment — actively contributing design ideas rather than just capturing them.

Three sources generate enrichment suggestions:

1. **Dramaturg ideation** — interaction patterns the user hasn't mentioned, cross-feature implications from Context Grounding, edge cases and boundary conditions, domain-specific oversights
2. **Gemini creative ideation** — a structured prompt that builds on the Dramaturg's own ideas, bringing external perspective and possibilities
3. **Merge and curate** — deduplicate across both sources, then tier into strong recommendations, worth-asking, and remainder based on relevance to the user's stated goals

Suggestions are presented in three passes (strong → worth-asking → remainder) using AskUserQuestion for clean yes/no decisions with context. Brief discussion is allowed, but when it drifts into architectural detail, items are tabled for the Approach Loop.

Vision Expansion is mandatory after every Vision Loop pass, including loop-backs from later phases — the vision shift may have created new enrichment opportunities.
```

**Step 2: Write "Research Protocol" section**

Source: `skill/SKILL.md:267-330` (research protocol), `skill/references/research-strategy.md` §tool-hierarchy

```markdown
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
```

**Step 3: Verify against sources**

Cross-check:
- Enrichment sources match `skill/references/vision-expansion.md` §enrichment-sources (three sources, tiering)
- Three-pass presentation matches §three-pass-presentation
- Mandatory re-run matches `skill/SKILL.md:148`
- Research triggers match `skill/SKILL.md:276-290`
- Tool hierarchy matches `skill/references/research-strategy.md`
- Priority chain matches `skill/SKILL.md:27`

**Step 4: Commit**

```bash
git add README.md
git commit -m "docs: add vision expansion and research protocol sections"
```

---

### Task 4: Write Decision Journal, Outputs, and Guardrails Sections

**Files:**
- Modify: `README.md` (append)

**Step 1: Write "Decision Journal" section**

Source: `skill/SKILL.md:333-404` (decision journal)

```markdown
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
```

**Step 2: Write "Outputs and Handoff" section**

Source: `skill/SKILL.md:59-63` (artifacts), `skill/SKILL.md:249-263` (final design doc)

```markdown
## Outputs and Handoff

The Dramaturg produces two artifacts:

1. **Design document** — `docs/plans/designs/YYYY-MM-DD-<topic>-design.md`. An open-ended narrative capturing the design vision. Required elements: a Goals section (opening) with verbose, context-rich statements of the user's goals, and an Arranger Notes appendix (closing) flagging new protocols, open questions, and key design decisions for the planner.

2. **Decision journal** — `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md`. The structured decision trail described above. Archived to `docs/archive/` after the design doc is finalized.

After compilation, the Dramaturg informs the user: "The design is complete. When you're ready, invoke the Arranger to create an implementation plan from this design."
```

**Step 3: Write "Guardrails" section**

Source: `skill/SKILL.md:24-37` (mandatory rules), `skill/SKILL.md:407-451` (conversation style)

```markdown
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
```

**Step 4: Verify against SKILL.md**

Cross-check:
- Artifact count is two (SKILL.md:59-63 is authoritative — Arranger Notes is part of design doc, not separate)
- Journal path matches `skill/SKILL.md:339`
- Archival to `docs/archive/` matches `skill/SKILL.md:257`
- All guardrails traceable to mandatory rules in `skill/SKILL.md:24-37`

**Step 5: Commit**

```bash
git add README.md
git commit -m "docs: add decision journal, outputs, and guardrails sections"
```

---

### Task 5: Write Session Management, Project Structure, Requirements, and Origin

**Files:**
- Modify: `README.md` (append final sections)

**Step 1: Write "Session Management" section**

Source: `skill/SKILL.md:455-467` (session bootstrap), `skill/SKILL.md:179-181` (session split)

```markdown
## Session Management

The Approach Loop (Phase 5) is the most context-intensive phase. When context pressure builds, the Dramaturg evaluates a session split at the Phase 5→6 boundary — completing approach exploration in one session, then starting the Review Loop fresh in a new session.

When starting a new session (fresh or post-split), the Dramaturg bootstraps from the decision journal:

1. Read the journal — the authoritative record of all settled decisions
2. Identify the current phase from the most recent entry type
3. Verify completeness — all expected entries exist for completed phases
4. Summarize for the user — what's settled and where the session picks up

If the journal is missing or corrupted, the Dramaturg reconstructs from conversation context if available. If both are unavailable, it starts fresh from the earliest unverifiable phase.
```

**Step 2: Write "Project Structure" section**

Source: actual repo file listing (verified during exploration)

```markdown
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
```

**Step 3: Write "Requirements" and "Origin" sections**

```markdown
## Requirements

- **Gemini MCP** (mandatory — skill will not start without it)
- **Claude Code** with skill/plugin support

## Origin

Part of [The Elevated Stage](https://github.com/The-Elevated-Stage) orchestration system. Design doc: `docs/designs/2026-02-17-dramaturg-skill-design.md`.
```

**Step 4: Verify project structure against actual repo**

Run `find dramaturg/ -type f -not -path '*/\.*'` and confirm the tree matches.

**Step 5: Commit**

```bash
git add README.md
git commit -m "docs: add session management, project structure, requirements, and origin"
```

---

### Task 6: Full Cross-Check and Final Commit

**Files:**
- Review: `README.md` (complete)

**Step 1: Read the complete README end-to-end**

Verify flow, tone consistency, and no gaps between sections.

**Step 2: Cross-check every factual claim**

Check against these sources:
- `skill/SKILL.md` — phase names, mandatory rules, artifacts, paths, Gemini requirement
- `skill/references/vision-expansion.md` — enrichment sources, three-pass presentation, mandatory re-run
- `skill/references/research-strategy.md` — tool hierarchy
- `skill/references/support-phases.md` — Context Grounding, Broad Design Scoping, Reconciliation, Final Design Doc
- `skill/references/approach-loop.md` — session split evaluation
- Actual repo file listing — project structure accuracy

**Step 3: Verify no contradictions with current README's Authority section**

The authority hierarchy should be preserved:
- SKILL.md is the authoritative runtime document
- Design documents record rationale and decisions
- If they disagree, SKILL.md wins

**Step 4: Final commit if any corrections were made**

```bash
git add README.md
git commit -m "docs: final cross-check corrections for README redesign"
```
