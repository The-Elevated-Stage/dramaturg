# Staged Brainstorming Technique — Separation of Concerns Applied to LLM Interrogation

## Source Session
**Date:** 2026-04-17
**Context:** Dramaturg session designing Aletheia V2 (memory-system evolution). During Phase 3 Vision Expansion, Kyle requested a Gemini brainstorm on popular CC / agent memory systems to ensure V2 incorporates existing strengths and avoids known anti-patterns. Kyle explicitly specified a 3-stage structure rather than a single-shot call.

## Kyle's Original Rationale (Verbatim)

> "For this Gemini brainstorm, I would like you to run it in stages, where stage 1 is get a list of popular memory systems, stage 2 is get the variances and similarities in the projects with their pros/cons and strongest points, stage 3 is a more thorough brainstorm where we introduce our vision for Aletheia and use the context of the first 2 stages to help ensure we are hitting both pros/cons of existing systems to incorporate/avoid. My concern is that running a single phase will result in too broad of ideas and will not catch the anti-patterns. This should also help broaden the scope of what Gemini is looking at."

## The Insight

Kyle's 3-stage structure is **separation of concerns applied to LLM interrogation**. Each stage forces the model to perform one cognitive job well:

| Stage | Cognitive job | Tool |
|---|---|---|
| 1 | **Breadth** — inventory only, no scoring, no judgment | `gemini-search` (grounded in web data for currency) |
| 2 | **Structure** — comparative synthesis with explicit axes + anti-patterns + optional adopt-vs-build scoring | `gemini-query` (pro / high reasoning, Stage 1 data embedded) |
| 3 | **Application** — vision-contextualized brainstorm with Claude thoughts + Stages 1-2 as context | `gemini-brainstorm` (multiple rounds, `claudeThoughts` input) |

A single-shot prompt ("brainstorm about memory systems for Aletheia") collapses these three jobs into one call. Observed failure modes of collapsed prompts:
- Generic advice rather than specific patterns
- Anti-patterns missed because the model is simultaneously trying to be creative and critical
- **Vision contamination of the inventory step** — the model biases its "inventory" toward what fits the target design, losing genuine breadth
- No traceability between recommendations and source systems

Staged calls force each step to be honest:
- Stage 1 doesn't know about the target design yet, so the inventory is uncontaminated
- Stage 2 doesn't have to be creative, so it can be rigorously critical
- Stage 3 can reason creatively on a foundation of structured data it couldn't have built alone in one pass

## When to Use This Technique

Trigger conditions (any of):
- The Dramaturg session is about to request competitive-landscape research ("how do other systems handle X?")
- The user flags concern about single-shot brainstorms missing nuance
- The design has areas with **no clear precedent** — need to surface novel-territory risks specifically
- A target feature could plausibly already exist as a library — adopt-vs-build evaluation is in scope

Do NOT use for:
- Simple clarification questions Gemini could answer in one call
- Validation of a single specific claim ("is SQLite WAL actually safe across processes?")
- Small-scope design questions where inventory would be overkill

## How to Structure Each Stage

### Stage 1 — Breadth Inventory

- **Goal:** Comprehensive list with short per-item descriptions. No ranking, no recommendation.
- **Prompt shape:** Categories to cover + per-item fields (name, mechanism, use case, differentiator). Include *"Produce inventory only — do NOT rank or deeply analyze"* as an explicit constraint.
- **Tool:** `gemini-search` if currency matters (the set of systems changes fast); `gemini-query` if the world is stable.
- **Length target:** 20-30+ systems is fine. Breadth over curation.

### Stage 2 — Comparative Synthesis

- **Goal:** Axes-of-variance matrix + pros/cons per family + anti-pattern catalog + optional adopt-vs-build scoring with a requirements checklist.
- **Prompt shape:** Embed Stage 1 inventory (condensed — full output has too many citation artifacts; strip them). Specify explicit axes for comparison. Ask for anti-patterns as a dedicated section, not an afterthought. If adopt-vs-build is in scope, provide a scoring rubric (0 = missing, 1 = partial, 2 = fully meets) against a requirements checklist.
- **Tool:** `gemini-query` with `model=pro` and `thinkingLevel=high`. Long outputs are fine — depth matters here.
- **Verification:** Look for hedge words and low-confidence flags; Gemini marks them explicitly when asked. Flagged speculation is honest speculation; unflagged confidence may be hallucination.

### Stage 3 — Vision-Contextualized Brainstorm

- **Goal:** Concrete architectural proposals for the target system, informed by Stages 1-2.
- **Prompt shape:** Full vision description + Stages 1-2 distilled outputs + specific deliverables (principle incorporation, anti-pattern avoidance verification, novel-risk brainstorm, gaps in the delta list, alternatives for risky choices, creative patterns the Dramaturg missed).
- **Tool:** `gemini-brainstorm` with `claudeThoughts` = substantive synthesis of stages 1-2 **plus tensions you want chewed on**. `maxRounds=4` for depth.
- **Key move:** Use `claudeThoughts` to push back on yourself. List tensions, hypotheses to test, questions you think you know the answer to. Gemini's value at this stage is disagreeing with or refining those hypotheses — not repeating them back to you.

## Recording the Result

The 3-stage brainstorm is a **research diversion within Vision Expansion**. It does not count as a Phase and does not get its own Phase Registry entry. Record it in the decision journal as a single consolidated Research entry after Stage 3 completes:

```markdown
## Research: Landscape Survey via Staged Brainstorm
**Phase:** Phase 3 — Vision Expansion
**Question:** [what the staged brainstorm was investigating]
**Tools used:** gemini-search (Stage 1), gemini-query pro/high (Stage 2), gemini-brainstorm maxRounds=4 (Stage 3)
**Findings:** [consolidated synthesis of all 3 stages — verdict, anti-patterns avoided, principles carried forward]
**Decision:** [adopt/build verdict + which principles are locked in]
**Arranger note:** VERIFIED (3-stage Gemini pipeline with grounded Stage 1) — or PARTIAL if anything remained speculative
```

One journal entry, not three. The stage-level outputs live in session context; only the consolidated synthesis and its decisions carry forward to Phase 5 and the Arranger.

## Why Document This for Future Dramaturg Sessions

The technique is nowhere in the current Dramaturg skill files. It emerged organically from Kyle's prior experience with single-shot brainstorm failures. Future Dramaturg sessions that encounter broad-landscape questions should reach for this pattern automatically rather than reinvent it — or worse, do a single-shot call and miss anti-patterns that matter.

**Candidate home if this pattern gets skill-ified:** `skills/dramaturg/references/research-protocol.md` could grow a "staged brainstorm" sub-section, or a dedicated `references/staged-brainstorming.md` could be added and referenced from Phase 3 Vision Expansion and the research-protocol's trigger rules.

## Related Patterns

- **Fresh Eyes Review** (`docs/working/2026-04-08-fresh-eyes-review-pattern.md`) — post-design flaw-finding via teammate given ONLY the design doc. Shares the core principle of separating cognitive jobs (designer vs reviewer). Staged brainstorming separates jobs *within a single session*; fresh-eyes review separates them *across sessions*. They compose well: stage 1-2-3 during Vision Expansion, fresh-eyes review at end of Review Loop.


---

## 2026-04-27 — Amendment & Integration Note

**Line 92 amendment:** the Related Patterns section says "fresh-eyes review separates them *across sessions*." That phrasing is a 200k-context-era artifact and is not how the fresh-eyes pattern actually works (see `2026-04-08-fresh-eyes-review-pattern.md` §"Why Teammate, Not Separate Session"). Both patterns separate cognitive jobs *within a single session* via teammates with their own context windows. They compose: stage 1-2-3 brainstorming during Vision Expansion or Approach Loop research; fresh-eyes review at end of Review Loop. No session split is required for either.

**Integration:** this technique has been folded into the Dramaturg skill at `skills/dramaturg/references/staged-brainstorming.md`, triggered from Phase 3 Vision Expansion (broad-landscape enrichment) and Phase 5 Approach Loop research diversions (broad-landscape research questions). The single-consolidated-journal-entry convention from this doc is preserved.
