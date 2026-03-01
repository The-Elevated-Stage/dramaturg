# Dramaturg SKILL.md Lean Hub Refactor Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Refactor Dramaturg's SKILL.md from a 470-line heavy hub to a ~280-310 line lean hub with prompt-injection routing, moving protocol content to reference files while preserving strict Phase Registry routing discipline.

**Architecture:** Content migration from SKILL.md to reference files. Research protocol merges into renamed `research-protocol.md`. New `decision-journal.md` reference created. Phase Registry entries slimmed to role + mandatory gate + journal checkpoint + reference pointer. No behavioral changes — same routing, same mandatories, same workflow.

**Tech Stack:** Markdown (Tier 3 hybrid document structure)

**Design doc:** `docs/working/2026-03-01-skill-refactor-design.md`

**Source material:**
- `skill/SKILL.md` — current 470-line hub (content source for migration)
- `skill/references/research-strategy.md` — current 195-line file (absorbs research protocol, gets renamed)
- `skill/references/vision-loop.md` — 161 lines (absorbs framing paragraph)
- `skill/references/vision-expansion.md` — 226 lines (absorbs post-loop-back mandatory)
- `skill/references/approach-loop.md` — 360 lines (verify vision-change detection completeness)
- `skill/references/review-loop.md` — 232 lines (unchanged)
- `skill/references/support-phases.md` — 276 lines (unchanged)

---

### Task 1: Create `decision-journal.md` reference file

This is the one genuinely new file. Create it before modifying SKILL.md so the reference pointer has a target.

**Files:**
- Create: `skill/references/decision-journal.md`

**Step 1: Create the file**

Create `skill/references/decision-journal.md` with content migrated from SKILL.md lines 333-405 (the `decision-journal` section). Structure it as a Tier 3 reference file:

```markdown
<skill>

<sections>
- purpose
- entry-types
- special-entries
- checkpoint-triggers
- lifecycle
</sections>

<section id="purpose">
<core>
## Purpose

The decision journal is the Dramaturg's progressive external memory. It captures decisions, research findings, and vision context as the session progresses, ensuring no work is lost to context compaction or session interruption.

1. **Context preservation** — Settled decisions survive context compaction. Critical for the Review Loop which needs Approach Loop decisions that may have compacted.
2. **Session resilience** — If a session dies mid-Approach-Loop, the journal preserves all settled approaches and research findings. A new session reads the journal and picks up where it left off.
3. **Pipeline handoff** — Research-backed decisions carry forward to the Arranger. Entries marked VERIFIED let the Arranger skip re-verification. Entries marked PARTIAL flag what still needs checking.
4. **Audit trail** — Shows how the design evolved, not just the final answer.
</core>
</section>

<section id="entry-types">
<core>
## Entry Types

Entry templates are placed inline at their checkpoint trigger points across reference files. The three formats:

### User-Discussed Entries

Settled through conversation:
- Fields: Phase, Category (goal | use-case | decision), Decided, User verbatim, User context, Alternatives discussed, Status
- **Category tags matter:** Goals and use cases are inviolable for downstream skills — they represent user intent that survives implementation replanning. Decisions can be reworked.
- **goal** — What the user wants the feature to accomplish. Be verbose. Capture the full scope including specific behaviors and types.
- **use-case** — How the user plans to use the feature. Reveals the user's mental model.
- **decision** — A technical or design choice. Default category.

<mandatory>User input is captured verbatim in the "User verbatim" field — not paraphrased. Downstream skills need the user's actual language to understand intent.</mandatory>

### Research-Backed Entries

Validated through investigation:
- Fields: Phase, Question, Tools used, Findings, Decision, Arranger note (VERIFIED | PARTIAL), Status
- The **Arranger note** enables cross-skill optimization. VERIFIED = Arranger can skip re-audit. PARTIAL = flag what still needs verification. UNRESEARCHED = decision made without research backing (e.g., Gemini was unavailable); Arranger must verify independently.

### Tension Entries

Requirements that exist in productive conflict:
- Fields: Phase, Requirements in tension, Why they conflict, Current resolution approach, Status
- Tensions are not failures — "offline AND real-time" is solvable but represents a design tension that the Arranger should be aware of
- Status is "acknowledged" not "settled" — the tension persists even after a resolution approach is chosen
- The Arranger receives tensions as design context, not as problems to solve

<template follow="format">
## Tension: [Short description]
**Phase:** [Phase where tension was identified]
**Requirements in tension:**
- [Requirement A]
- [Requirement B]
**Why they conflict:** [Brief explanation of the productive conflict]
**Current resolution approach:** [How the design currently handles the tension, or "unresolved — Arranger decides"]
**Status:** acknowledged
</template>
</core>
</section>

<section id="special-entries">
<core>
## Special Entries

These entries are written at specific phase exits. Templates for each appear in the relevant reference files at the checkpoint trigger points.

- **Vision Baseline** (Phase 2 exit) — What/Why/How-used. The regression test anchor.
- **Vision Expansion** (Phase 3 exit) — Accepted, rejected, and tabled enrichments. Vision Baseline update status.
- **Topic Map** (Phase 4 exit) — Areas to explore, user-confirmed coverage.
- **Section Approval** (Phase 6) — Key decisions reflected, feedback incorporated.
</core>
</section>

<section id="checkpoint-triggers">
<core>
## Checkpoint Triggers

Journal writes happen at:
- Vision Loop exit: vision baseline entry
- Vision Expansion exit: vision expansion entry (accepted, rejected, tabled capabilities)
- Broad Design Scoping exit: topic map entry
- Every settled topic in Approach Loop (decision or research entry)
- Every section approval in Review Loop
- Before discussing results of significant autonomous research — journal first, then discuss
</core>
</section>

<section id="lifecycle">
<core>
## Lifecycle Rules

<mandatory>If a journal file already exists at the target path (from a previous session or concurrent work), archive or rename it before beginning a new session's journal.</mandatory>

<mandatory>The journal is append-only. Entries are never edited. If a later decision supersedes an earlier one, the new entry references the old one.</mandatory>

Consultation journals (from the Repetiteur) are distinct from Dramaturg journals. A future Repetiteur consultation can override a prior Repetiteur's decision, but cannot override a user decision found in Dramaturg journals without escalating to the user.
</core>
</section>

</skill>
```

**Step 2: Verify the new file**

Read back `skill/references/decision-journal.md` and verify:
- All 5 sections present and match `<sections>` index
- All three entry types have complete field lists
- Tension template is present
- All 6 checkpoint triggers listed
- Both lifecycle mandatories preserved (existing journal handling, append-only)
- Verbatim capture mandatory preserved

---

### Task 2: Rename `research-strategy.md` to `research-protocol.md` and absorb content

**Files:**
- Rename: `skill/references/research-strategy.md` → `skill/references/research-protocol.md`
- Modify: `skill/references/research-protocol.md` (prepend new sections)

**Step 1: Rename the file**

```bash
cd /home/kyle/claude/skills_staged/orchestra/dramaturg
git mv skill/references/research-strategy.md skill/references/research-protocol.md
```

**Step 2: Prepend trigger-rules and diversion-flow sections**

Edit `skill/references/research-protocol.md`. Update the `<sections>` index to add the two new sections at the top:

```
<sections>
- trigger-rules
- diversion-flow
- tool-hierarchy
- subagent-delegation
- conflict-resolution
- inconclusive-research
- gemini-failure
</sections>
```

Update the loading guide comment:

```
<!-- Loading guide:
  - §trigger-rules, §diversion-flow: Load on first research diversion
  - §tool-hierarchy, §subagent-delegation: Load on first research execution
  - §conflict-resolution: Load when encountering conflicting sources
  - §inconclusive-research: Load when research returns inconclusive
  - §gemini-failure: Load when Gemini fails mid-session
-->
```

Insert two new sections before the existing `<section id="tool-hierarchy">`. Content migrated from SKILL.md lines 275-329:

```markdown
<section id="trigger-rules">
<core>
## Research Trigger Rules

Research happens when triggered by specific conditions during conversation. It does not happen on a schedule.

**When the Dramaturg recommends an approach:** Research FIRST. Validate the approach, THEN present with evidence. <mandatory>Never recommend based solely on training data for substantive technical decisions. Apply the priority chain: compatibility > reliability > efficiency > security > performance.</mandatory>

**When the user proposes something with clear intent:** Research immediately to validate feasibility. The intent is clear — do not slow the conversation by asking "should I research this?"

**When the user proposes something with ambiguous intent:** Clarify first. Fully understand the goal. Confirm understanding. THEN ask "should I research this approach?"

**When a protocol or pattern is not already implemented in the project:** Research through Gemini before incorporating it into the design. Even if the Dramaturg "knows" how something works from training data, validate via current research to catch version-specific gotchas and platform constraints.

**When NOT to research:**
- Trivial decisions that don't affect architecture
- Clarification exchanges still establishing intent
- Details within the project's reliable, existing patterns
- Minor adjustments to already-researched approaches
- When the user explicitly declines research — accept their informed judgment, journal as user-directed decision
</core>
</section>

<section id="diversion-flow">
<core>
## Research Diversion Flow

<mandatory>Research diversions use the decision journal for flow anchoring. Write the entry before research, update to "settled" after return. Research diversions return directly to the recorded step — no Phase Registry re-bootstrap needed.</mandatory>

**Step 1 — Record position in the decision journal:**

<template follow="format">
## Research Diversion (in progress)
**Position:** [reference-file.md] §[section-id] → [step]
**Topic:** [what was being discussed] — [context for why research is needed]
**Trigger:** [user-proposal-clear-intent | dramaturg-recommendation | unimplemented-pattern | user-proposal-ambiguous]
**Status:** in-progress
</template>

**Step 2 — Execute research:**

Tool selection (use the best tool for each job):
1. **Gemini MCP** — Primary for feasibility research, technical questions, approach analysis. Excels at synthesizing tradeoffs and suggesting alternatives.
2. **Brave-search** — Discovering what exists: new libraries, recent articles, community sentiment.
3. **Explore subagents** — Codebase understanding, architectural analysis. Use subagents to keep main context clean.
4. **WebFetch** — Reading specific known URLs.
5. **General-purpose subagents** — Multi-source synthesis, speculative research. Isolates verbose or speculative exploration from the main conversation context — if the path is a dead end, the main session loses nothing.

For detailed tool rationale, subagent delegation patterns, and research depth guidance, see §tool-hierarchy and §subagent-delegation.

**Research should over-investigate, not under-investigate.** Go deep — check edge cases, discover limitations, find alternatives, form recommendations. Shallow research ("X exists and seems to work") is less valuable than thorough research ("X exists, works for Y, has Z limitation, and A might be better because B").

**Provide explicit recommendations**, not just findings. "Here are three options, and I recommend X because Y" is more useful than "here are three options." <mandatory>Recommendations must follow the priority chain: compatibility > reliability > efficiency > security > performance.</mandatory>

**Gemini alternatives:** Gemini naturally suggests alternatives — this is valuable. Briefly assess each against the current context. Deep-dive only on the ones that look genuinely promising or challenge an assumption. Not all alternatives need full investigation.

**Step 3 — Return to recorded step:**

Read the most recent "Research Diversion (in progress)" entry in the decision journal. Return directly to the recorded position in the reference file. Update the journal entry's status from "in-progress" to "settled" and add a **Findings** field with the research results. Resume the discussion with research findings.

<mandatory>Update the journal entry status after returning. In-progress entries must not accumulate — they indicate either active research or a stale diversion from a session interruption. If a session starts with an existing in-progress research diversion entry, re-execute the research from the recorded context.</mandatory>
</core>
</section>
```

**Step 3: Remove the old reference pointer from the diversion-flow section**

The current SKILL.md research-protocol section has a `<reference path="references/research-strategy.md">` tag pointing to the old filename. In the new `diversion-flow` section above, we replaced that with a prose cross-reference: "For detailed tool rationale, subagent delegation patterns, and research depth guidance, see §tool-hierarchy and §subagent-delegation." This is an internal cross-reference within the same file now, not a reference tag to a separate file. Verify no `<reference path="references/research-strategy.md">` tag exists in the new file.

**Step 4: Verify the renamed and expanded file**

Read back `skill/references/research-protocol.md` and verify:
- `<sections>` index lists all 7 sections in order
- Loading guide comment updated
- `trigger-rules` section has all 5 trigger scenarios + "when NOT to research"
- `diversion-flow` section has 3-step flow with journal template
- No references to `research-strategy.md` (old filename)
- Existing sections (`tool-hierarchy` through `gemini-failure`) unchanged

---

### Task 3: Migrate framing content into phase reference files

**Files:**
- Modify: `skill/references/vision-loop.md` (add framing context)
- Modify: `skill/references/vision-expansion.md` (add post-loop-back mandatory)
- Modify: `skill/references/approach-loop.md` (add thoroughness framing)
- Verify: `skill/references/approach-loop.md` (confirm vision-change detection completeness)

**Step 1: Add framing context to vision-loop.md**

The Phase Registry currently describes the Vision Loop as "brainstorming-style dialogue: open-ended, one question at a time, drawing the user out" (SKILL.md line 116). This framing context should live in the reference file.

Edit `skill/references/vision-loop.md`. In the `question-strategy` section, add a framing paragraph before the existing "The Vision Loop draws out..." line (currently line 13). Insert after the `<core>` tag on line 10:

```markdown
## Question Strategy

The Vision Loop is brainstorming-style dialogue: open-ended, one question at a time, drawing the user out. The goal is to understand what the user wants, why they want it, and how they'll use it — entirely from the user's own words. The Dramaturg contributes design ideas later (Phase 3); here, it listens.

The Vision Loop draws out What/Why/How-used entirely from the user's own words. Each dimension requires a different questioning approach because users volunteer them differently.
```

This replaces the current line 13 (`The Vision Loop draws out What/Why/How-used entirely from the user's own words. Each dimension requires a different questioning approach because users volunteer them differently.`) with a version that includes the migrated framing before the existing opening.

**Step 2: Add post-loop-back mandatory to vision-expansion.md**

The Phase Registry states: "Vision Expansion is mandatory after every Vision Loop pass, including loop-backs from later phases. The vision shift may have created new enrichment opportunities." (SKILL.md line 148). This is already in `vision-expansion.md` at the `loop-back-integration` section (verify by reading that section). If it's already there, this step is a no-op — just confirm and move on.

Read `skill/references/vision-expansion.md` and check whether the `loop-back-integration` section already contains the mandatory about re-running after every Vision Loop pass. If it does, no change needed. If not, add:

```markdown
<mandatory>Vision Expansion is mandatory after every Vision Loop pass, including loop-backs from later phases. The vision shift may have created new enrichment opportunities.</mandatory>
```

at the beginning of the `loop-back-integration` section.

**Step 3: Add thoroughness framing to approach-loop.md**

The Phase Registry describes the Approach Loop's time investment (SKILL.md lines 177-179): "This phase is where the Dramaturg spends the most time. A single topic's discussion may last 30-60+ minutes of back-and-forth research and exploration. Thoroughness here is the skill's primary value — never rush or compress this phase." This framing should live in the reference file.

Edit `skill/references/approach-loop.md`. In the `topic-selection` section, add a framing paragraph after the `<core>` tag on line 19, before the existing mandatory on line 21. Insert:

```markdown
## Topic Selection

This is the core of the Dramaturg skill — where the big architectural decisions happen, informed by active research. A single topic's discussion may last 30-60+ minutes of back-and-forth research and exploration. Thoroughness here is the skill's primary value — never rush or compress this phase.
```

This replaces the current line 19 heading (`## Topic Selection`) with a version that includes the migrated thoroughness framing before the existing mandatory about reading the Topic Map.

**Step 4: Verify approach-loop.md vision-change detection**

The Phase Registry has detailed vision-change detection content (SKILL.md lines 195): scope contraction, goal shift, new use cases. Verify that `skill/references/approach-loop.md` section `vision-change-detection` already contains all of this:
- Scope contraction example ("actually I just need X, not Y")
- Goal shift example ("this is really about X, not Y")
- New use cases that invalidate settled topics
- Routing instruction to Phase Registry → Vision Loop
- Mandatory Vision Expansion pass-through on return path

Read `approach-loop.md` lines 123-138. All items are already present based on earlier grep results. This step is verification only — no edit expected.

**Step 5: Verify all three files**

Read the modified sections of each file and confirm:
- `vision-loop.md` has the framing paragraph integrated naturally (not duplicated with existing content)
- `vision-expansion.md` has the post-loop-back mandatory (either already existed or was added)
- `approach-loop.md` has thoroughness framing prepended to topic-selection opening
- `approach-loop.md` has complete vision-change detection (confirmed, no edit needed)

---

### Task 4: Rewrite SKILL.md Phase Registry and protocol sections

This is the core task — slimming SKILL.md. All target reference files now have their absorbed content.

**Files:**
- Modify: `skill/SKILL.md` (major rewrite of lines 79-405)

**Step 1: Rewrite the Phase Registry (lines 79-265)**

Replace the entire `phase-registry` section content. Keep the opening paragraph, the mandatory about routing through the registry, the guidance about repeated loop-backs, and the routing table. Replace all per-phase entries with the lean pattern.

The new Phase Registry section should be:

```markdown
<section id="phase-registry">
<core>
## Phase Registry

These are the phases of the Dramaturg's workflow. When a reference file says "Proceed to Phase Registry → [Phase Name]," return to this section, find the phase, re-read its framing and constraints, then follow the reference pointer. Every phase transition passes through this registry.

<mandatory>All loop-backs between phases pass through this registry. Reference files never route directly to other reference files. When transitioning: re-read the Phase Registry entry for the target phase in SKILL.md — this is a literal instruction (use the Read tool on SKILL.md) because the framing and constraints may have compacted from context. The registry re-bootstraps the Dramaturg with fresh framing and constraints before entering the next phase.</mandatory>

<guidance>If the same loop-back occurs twice (e.g., Review Loop routes back to Vision Loop a second time), the underlying issue may not be addressable through the normal loop-back path. Surface the pattern to the user: "We've looped back to [phase] twice now. There may be a deeper issue worth discussing directly."</guidance>

| # | Phase | Role | Reference |
|---|-------|------|-----------|
| 1 | **Context Grounding** | Project exploration before design discussion | `support-phases.md` §context-grounding |
| 2 | **Vision Loop** | Establish What/Why/How-used from user input | `vision-loop.md` |
| 3 | **Vision Expansion** | Proactive enrichment — capabilities, interactions, edge cases | `vision-expansion.md` |
| 4 | **Broad Design Scoping** | High-level framing questions | `support-phases.md` §broad-design-scoping |
| 5 | **Approach Loop** | Research-backed topic exploration and settlement | `approach-loop.md` |
| 6 | **Review Loop** | Per-section detailed design review | `review-loop.md` |
| 7 | **Reconciliation** | Cross-section consistency check | `support-phases.md` §reconciliation |
| 8 | **Final Design Doc** | Compile and output | `support-phases.md` §final-design-doc |

---

### Context Grounding

**Role:** Understand the current project before discussing what to build on top of it.

**Journal checkpoint:** None — Context Grounding does not produce a journal entry.

<reference path="references/support-phases.md" load="required">
Read §context-grounding only. Explore subagent launch patterns for project understanding.
</reference>

---

### Vision Loop

**Role:** Understand what the user wants, why they want it, and how they'll use it — entirely from the user's own words.

<mandatory>Transition gate (strict): Move to Vision Expansion ONLY when the Dramaturg can answer all three of these from user input alone:
1. **What** does the user want to build?
2. **Why** do they want to build it? (the motivation, the problem it solves)
3. **How** will they use it? (concrete use cases, user scenarios)
These answers must come strictly from what the user has said. Do not infer the "why" from project context or fill in use cases from training data. If any answer is missing, keep asking.</mandatory>

**Journal checkpoint:** Write the Vision Baseline entry on exit.

<reference path="references/vision-loop.md" load="required">
Question strategy, gate verification, research during vision gathering, vision baseline journal template, loop-back conditions.
</reference>

---

### Vision Expansion

**Role:** Proactively enrich the user's vision with capabilities, interactions, edge cases, and cross-feature implications they haven't considered.

<mandatory>Vision Expansion is mandatory after every Vision Loop pass, including loop-backs from later phases.</mandatory>

**Journal checkpoint:** Write the Vision Expansion entry on exit.

<reference path="references/vision-expansion.md" load="required">
Enrichment sources, three-pass presentation flow, AskUserQuestion pattern, Gemini prompt design, discussion boundary, vision baseline re-verification, journal template.
</reference>

---

### Broad Design Scoping

**Role:** Establish a high-level map of the design territory before diving into detailed per-section work.

<mandatory>Transition gate: Before moving to Approach Loop, explicitly list the topics to explore in Phase 5 and ask the user to confirm the list is complete.</mandatory>

**Journal checkpoint:** Write the Topic Map entry on exit.

<reference path="references/support-phases.md" load="required">
Read §broad-design-scoping only. Question framing, constraints (max 6 questions, max 2 per area), topic map construction.
</reference>

---

### Approach Loop

**Role:** Explore, research, and settle on the right approach for each major design topic through fluid, research-backed conversation. This is the core of the skill.

<mandatory>Research FIRST, recommend AFTER. Never recommend based solely on training data for substantive technical decisions.</mandatory>

<mandatory>Vision-change detection: If the user's stated goal, scope, or use cases change fundamentally during approach exploration, pause forward progress and confirm with the user. If confirmed, route to Phase Registry → Vision Loop.</mandatory>

**Journal checkpoint:** Write a Decision or Research entry for every settled topic.

<reference path="references/approach-loop.md" load="required">
Topic selection, research-discuss-settle loop, tangent bounding, vision-change detection, infeasibility pivots, settlement confirmation, session split evaluation, ripple assessment protocol, journal templates.
</reference>

---

### Review Loop

**Role:** Present the detailed design as structured sections for focused review.

<mandatory>Present one section at a time. One section per message.</mandatory>

<mandatory>If user feedback constitutes a vision change, route to Phase Registry → Vision Loop. If it constitutes an approach change, route to Phase Registry → Approach Loop.</mandatory>

**Journal checkpoint:** Write a Section Approval entry for every approved section.

<reference path="references/review-loop.md" load="required">
Section presentation, feedback classification, vision regression test, approach change detection, security lens, section approval journal template.
</reference>

---

### Reconciliation

**Role:** Ensure all approved sections are internally consistent before compiling the final document.

<mandatory>If substantive inconsistencies are found, present affected sections for re-approval. Route to Phase Registry → Review Loop if a section needs rework beyond reconciliation-level fixes.</mandatory>

<mandatory>Implementation readiness gate: Before moving to Final Design Doc, evaluate — could the Arranger produce an executable implementation plan from this design without making design-level decisions?</mandatory>

<reference path="references/support-phases.md" load="required">
Read §reconciliation only. Severity classification, implementation readiness test.
</reference>

---

### Final Design Doc

**Role:** Compile all reconciled sections into a single design document at `docs/plans/designs/YYYY-MM-DD-<topic>-design.md`.

<reference path="references/support-phases.md" load="required">
Read §final-design-doc only. Goals section guidance, Arranger Notes template, journal archival.
</reference>

After the design document is finalized and the journal is archived, inform the user: "The design is complete. When you're ready, invoke the Arranger to create an implementation plan from this design."
</core>
</section>
```

**Step 2: Rewrite the research-protocol section (lines 267-331)**

Replace the entire `research-protocol` section with the slim routing version:

```markdown
<section id="research-protocol">
<core>
## Research Protocol

Research happens when triggered by specific conditions during conversation, not on a schedule. The research-protocol reference defines when to research, how to execute the diversion flow, and how to return to the recorded step.

<mandatory>Research diversions use the decision journal for flow anchoring. Write the entry before research, update to "settled" after return. Research diversions return directly to the recorded step — no Phase Registry re-bootstrap needed.</mandatory>

<reference path="references/research-protocol.md" load="required">
Trigger rules, diversion flow, tool selection, journal anchoring, conflict resolution, Gemini failure handling.
</reference>
</core>
</section>
```

**Step 3: Rewrite the decision-journal section (lines 333-405)**

Replace the entire `decision-journal` section with the slim routing version:

```markdown
<section id="decision-journal">
<core>
## Decision Journal

The decision journal is the Dramaturg's progressive external memory — capturing decisions, research findings, and vision context as the session progresses.

**Location:** `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md`

<mandatory>The journal is append-only. Entries are never edited. Write journal entries at every checkpoint trigger — skipping journal entries is a task failure.</mandatory>

<reference path="references/decision-journal.md" load="required">
Entry types, field lists, templates, checkpoint triggers, special entries, lifecycle rules.
</reference>
</core>
</section>
```

**Step 4: Update the mandatory-rules section**

In the `mandatory-rules` section, update line 31 which references `§research-protocol`:

Current: `Research diversions route through §research-protocol with direct return to recorded step — no Phase Registry re-bootstrap needed`

This wording still works — §research-protocol is still a section in SKILL.md, it just now routes to the reference file. No change needed. Verify and move on.

Also check line 315 reference to `research-strategy.md` — this is in the old research-protocol section which is being fully replaced in Step 2, so it's handled.

**Step 5: Verify the complete SKILL.md**

Read the entire new SKILL.md and verify:
- `<sections>` index matches actual section IDs (mandatory-rules, preamble, phase-registry, research-protocol, decision-journal, conversation-style, session-bootstrap)
- All 8 phases present in the Phase Registry with routing table
- Each phase has: role, mandatory gate (where applicable), journal checkpoint, reference pointer
- No orphaned content (severity table, entry type field lists, research triggers, etc. — all migrated)
- conversation-style section unchanged
- session-bootstrap section unchanged
- No references to `research-strategy.md` (old filename)

Run: `wc -l skill/SKILL.md` — expect 280-310 lines.

---

### Task 5: Update all references to the renamed file

**Files:**
- Modify: `skill/SKILL.md` (if any remaining references to `research-strategy.md`)
- Verify: all reference files for stale `research-strategy.md` references

**Step 1: Search for stale references**

```bash
cd /home/kyle/claude/skills_staged/orchestra/dramaturg
grep -r "research-strategy" skill/
```

Expected: 0 hits after Task 4. If any remain, update them to `research-protocol.md`.

**Step 2: Search more broadly**

```bash
grep -r "research-strategy" docs/ README.md
```

Update any hits to `research-protocol.md`.

**Step 3: Verify reference pointer consistency**

Check that all `<reference path="...">` tags in SKILL.md point to files that exist:

```bash
grep -oP 'path="references/[^"]+' skill/SKILL.md | sort -u
```

Cross-check each path against:

```bash
ls skill/references/
```

All paths should resolve. Expected files:
- `references/support-phases.md`
- `references/vision-loop.md`
- `references/vision-expansion.md`
- `references/approach-loop.md`
- `references/review-loop.md`
- `references/research-protocol.md`
- `references/decision-journal.md`

**Step 4: Verify reference files' internal references**

Check that reference files pointing to the research reference use the new name:

```bash
grep -r "research-strategy\|research-protocol" skill/references/
```

The only hits should be in `research-protocol.md` itself (if it has any self-references). No other reference file should mention `research-strategy.md`.

Also check that `vision-loop.md` line 49 still says `§research-protocol` (which refers to the SKILL.md section, not the filename):

```bash
grep "research-protocol\|research-strategy" skill/references/vision-loop.md
```

This should show `§research-protocol` — which is correct, it routes through SKILL.md.

---

### Task 6: Full verification and commit

**Step 1: Run all verification checks from the design doc**

1. **SKILL.md line count:**
```bash
wc -l skill/SKILL.md
```
Expected: 280-310 lines.

2. **No orphaned content** — every mandatory from old SKILL.md exists somewhere:
```bash
# Check key mandatories are preserved
grep -c "mandatory" skill/SKILL.md
grep -c "mandatory" skill/references/decision-journal.md
grep -c "mandatory" skill/references/research-protocol.md
```

3. **No broken routing** — every "Phase Registry" reference in files resolves:
```bash
grep -r "Phase Registry" skill/references/ | grep -v "^Binary"
```
All should point to phases that exist in the registry.

4. **No stale filenames:**
```bash
grep -r "research-strategy" skill/ docs/ README.md
```
Expected: 0 hits.

5. **Sections index consistency** — for each modified file, verify `<sections>` matches `<section id=`:
```bash
for f in skill/SKILL.md skill/references/research-protocol.md skill/references/decision-journal.md; do
  echo "=== $f ==="
  echo "Index:"
  grep -A 20 "^<sections>" "$f" | grep "^- "
  echo "Actual:"
  grep -oP 'section id="\K[^"]+' "$f"
  echo
done
```

6. **Reference file count:**
```bash
ls skill/references/
```
Expected: 7 files (approach-loop, decision-journal, research-protocol, review-loop, support-phases, vision-expansion, vision-loop).

**Step 2: Review the diff**

```bash
cd /home/kyle/claude/skills_staged/orchestra/dramaturg
git diff --stat
git diff
```

Review the full diff. Verify:
- SKILL.md is substantially smaller
- `research-strategy.md` is gone, `research-protocol.md` exists
- `decision-journal.md` is new
- Phase reference files have minor additions only
- No content was dropped without being migrated

**Step 3: Commit**

```bash
cd /home/kyle/claude/skills_staged/orchestra/dramaturg
git add skill/SKILL.md skill/references/
git commit -m "refactor: slim SKILL.md to lean hub with prompt-injection routing

- Slim Phase Registry to role + mandatory gate + journal checkpoint per phase
- Move research protocol (trigger rules, diversion flow) to research-protocol.md
  (renamed from research-strategy.md, absorbs SKILL.md content)
- Create decision-journal.md reference (entry types, templates, triggers)
- Migrate framing content to phase reference files (vision-loop, vision-expansion)
- Preserve strict Phase Registry routing discipline (all transitions through registry)
- SKILL.md reduced from 470 to ~290 lines

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

**Step 4: Push**

```bash
git push origin main
```
