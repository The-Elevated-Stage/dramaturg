<skill>

<sections>
- separation-of-concerns
- trigger-criteria
- when-to-use-narrative
- when-not-to-use
- stage-1-breadth
- stage-2-synthesis
- stage-3-application
- recording-the-result
- exit
</sections>

<!-- Loading guide:
  - §separation-of-concerns: Load on first encounter with broad-landscape research
  - §trigger-criteria: Load BEFORE deciding whether to escalate from single-shot Gemini — binary, auditable test
  - §when-to-use-narrative, §when-not-to-use: Companion narrative for §trigger-criteria; load mid-decision for the why behind the criteria
  - §stage-1-breadth, §stage-2-synthesis, §stage-3-application: Load in order as you execute each stage
  - §recording-the-result: Load after Stage 3 completes
  - §exit: Load when returning to the calling phase
-->

<section id="separation-of-concerns">
<core>
## Separation of Concerns

Single-shot brainstorm prompts collapse three cognitive jobs into one call and fail predictably. Staged brainstorming is separation of concerns applied to LLM interrogation — each stage forces the model to perform one cognitive job well.

| Stage | Cognitive job | Tool |
|---|---|---|
| 1 | **Breadth** — inventory only, no scoring, no judgment | `gemini-search` (grounded in web data for currency) |
| 2 | **Structure** — comparative synthesis with explicit axes, anti-patterns, optional adopt-vs-build scoring | `gemini-query` (`model=pro`, `thinkingLevel=high`) with Stage 1 inventory embedded |
| 3 | **Application** — vision-contextualized brainstorm with your synthesis plus tensions to chew on | `gemini-brainstorm` (`maxRounds=4`, `claudeThoughts` populated) |

### Why Single-Shot Prompts Fail

A single-shot prompt ("brainstorm about memory systems for our design") asks the model to be creative and critical simultaneously while inventorying the landscape and applying it to a target — four cognitive jobs in one call. Observed failure modes:

- **Generic advice** rather than specific patterns from real systems.
- **Anti-patterns missed** because the model is simultaneously trying to be creative and critical, and creativity wins.
- **Vision contamination of the inventory step** — the model biases its "inventory" toward what fits the target design, losing genuine breadth. This is the load-bearing failure mode the staged structure prevents.
- **No traceability** between recommendations and source systems — recommendations float free of evidence.

### Why Staged Calls Work

Each stage is honest because each stage has only one job:

- Stage 1 does not yet know about your target design, so the inventory cannot be contaminated by it.
- Stage 2 does not have to be creative, so it can be rigorously critical — anti-patterns get a dedicated section instead of an afterthought.
- Stage 3 reasons creatively on a foundation of structured data it could not have built alone in one pass.

The output of the three stages composed is materially better than any single-shot call can produce. The cost is three Gemini invocations instead of one — accept the cost when the trigger conditions in §trigger-criteria apply.
</core>
</section>

<section id="trigger-criteria">
<core>
## Trigger Criteria

The staged pipeline is recommended-not-mandatory: narrow-scope research does not benefit from the overhead, and the value scales with breadth. The *evaluation* of whether to invoke is mandatory; the *execution* is conditional on what the evaluation finds.

The criteria below are mechanically answerable from artifacts already in scope (the user's recent messages, the Topic Map, the Vision Baseline). No criterion depends on "if the question feels broad" or "Dramaturg's judgment" — that wiggle room lets single-shot Gemini win by default and the staged pattern erodes.

### Staged brainstorming SHOULD be invoked if any of the following criteria fire:

1. **User question explicitly references competitive landscape.** *Verify:* the user's most recent message contains any of: `how do other systems`, `competitive landscape`, `what's out there`, `survey`, `inventory of`, `adopt-vs-build`, `adopt vs build`, `existing libraries`, `existing solutions`, `prior art`, `state of the art`. One match is sufficient.
2. **Topic Map entry explicitly marks landscape research needed.** *Verify:* grep the journal's Topic Map entry for the marker `landscape-research-needed` or for a sub-topic listing 5 or more existing systems / libraries / approaches that the design must compare against.
3. **Question requires breadth-first inventory before application.** *Verify:* the question cannot be answered by validating a single specific claim (per `research-protocol.md` §when-not-to-use). Instead, an honest answer requires enumerating 5+ distinct existing approaches and synthesizing across them. If the answer would start with "There are several X..." rather than "X is/isn't...", this criterion fires.
4. **User has flagged single-shot brainstorm concern.** *Verify:* the user's recent messages contain explicit signal that single-shot Gemini will miss nuance — e.g., `single-shot might miss`, `worried we'll miss`, `surface anti-patterns`, `cover the field`. One match is sufficient.
5. **Adopt-vs-build evaluation in scope.** *Verify:* the design has flagged a feature that could plausibly already exist as a library/framework/service, AND the design needs a defensible verdict (build or adopt) before settlement. Topic Map or Vision Baseline mentions `adopt-vs-build`, `existing library`, `consider using`, or equivalent.

### Staged brainstorming MAY be skipped only if all of the following are true:

- The user's question is a single-claim validation (e.g., "Does X work with Y?") that one Gemini call answers — not a landscape question (criterion 1's keyword set absent, criterion 3's "There are several X..." shape absent).
- The Topic Map has no `landscape-research-needed` markers and no sub-topic enumerates 5+ existing approaches (per criterion 2).
- The user has not flagged single-shot concern (per criterion 4).
- Adopt-vs-build evaluation is not in scope — the design path is committed to building, or to a single named library (per criterion 5).

If any SHOULD-invoke criterion fires, you invoke the staged pipeline. The MAY-skip block is the *negation* of the SHOULD-invoke set, restated for legibility — it is not an independent gate. If you find yourself looking for reasons to skip when a SHOULD-invoke criterion fires, the gate has self-deceived.

<mandatory>The trigger evaluation is mandatory whenever a research diversion is initiated. Even when the decision is "skip — single-shot Gemini suffices," write a Trigger Decision journal entry per the research-diversion-flow in `research-protocol.md` §diversion-flow. The entry captures which criteria fired (or `—` if none), the decision, and the reasoning. This makes the choice between staged and single-shot auditable — without the entry, single-shot wins by default and the staged pattern's value erodes.</mandatory>
</core>
</section>

<section id="when-to-use-narrative">
<guidance>
## When to Use — Narrative Companion

This section is the conversational version of §trigger-criteria for a Dramaturg reading mid-diversion. The formal answer is binary and lives there; this is the why.

Trigger conditions (any of):

- **Competitive-landscape research** — "How do other systems in this category handle X?" The question implies inventorying existing systems before applying lessons. Corresponds to §trigger-criteria criterion 1 and criterion 3.
- **Single-shot concern** — you (or the user) flag concern that a single-shot brainstorm will miss nuance, anti-patterns, or specific failure modes. Trust the concern; reach for the staged pipeline. Corresponds to criterion 4.
- **No-clear-precedent design** — the design has areas without obvious prior art and you need to surface novel-territory risks specifically. Often co-occurs with criterion 2 (Topic Map flags landscape research needed) or criterion 3 (the question requires breadth-first inventory).
- **Adopt-vs-build evaluation** — a target feature could plausibly already exist as a library, framework, or service, and the design needs a defensible adopt-vs-build verdict. Corresponds to criterion 5.

Any one of these conditions is sufficient to invoke the staged pipeline. Multiple conditions reinforce the choice but do not require additional ceremony.

Cross-reference: every narrative trigger above corresponds to one or more SHOULD-invoke criteria in §trigger-criteria. The criteria are the binding answer — this section exists so the criteria are not a bare checklist disconnected from the reasoning behind them.
</guidance>
</section>

<section id="when-not-to-use">
<guidance>
## When Not to Use

Staged brainstorming has overhead. Three Gemini invocations plus consolidated journaling is the wrong shape for narrow-scope research.

Do not use the staged pipeline for:

- **Simple clarifications** Gemini can answer in one call ("Does Express 4.x support HTTP/2 natively?").
- **Single-claim validation** ("Is SQLite WAL actually safe across processes on this OS?"). Validate inline; staging adds no value.
- **Small-scope design questions** where inventory would be overkill. If the question is bounded enough that breadth-first inventory feels excessive, single-shot Gemini is the right tool.

The heuristic: if Stage 1 (breadth inventory) feels like overkill, you do not need Stages 2 or 3 either. Use the standard research-protocol single-shot path instead.
</guidance>
</section>

<section id="stage-1-breadth">
<core>
## Stage 1 — Breadth Inventory

**Goal:** Comprehensive list of relevant systems, libraries, or approaches with short per-item descriptions. No ranking, no recommendations, no scoring. Pure inventory.

**Tool:** `gemini-search` if currency matters (the set of systems in scope changes fast — new libraries, recent releases, evolving community sentiment). `gemini-query` if the world is stable enough that synthesis without grounded search is acceptable. Default to `gemini-search` for landscape questions.

**Length target:** 20-30+ items is fine. Breadth over curation. Trim later in Stage 2; do not pre-filter here.

### Prompt Shape

- Categories or families to cover (so the inventory does not miss a class entirely).
- Per-item fields to populate: name, mechanism, primary use case, differentiator from peers.
- Explicit constraint: *"Produce inventory only — do NOT rank, score, or deeply analyze the items. List comprehensively."*

<mandatory>Stage 1 must produce inventory only. Do not ask the model to rank, recommend, or pre-filter at this stage. The "produce inventory only" instruction is load-bearing — it is the constraint that prevents vision contamination of the inventory step. Skipping it collapses Stage 1 into a single-shot brainstorm and the entire pipeline loses its value.</mandatory>

The inventory is raw material for Stage 2. Citations and verbose explanations are fine here — Stage 2 will distill.
</core>
</section>

<section id="stage-2-synthesis">
<core>
## Stage 2 — Comparative Synthesis

**Goal:** Axes-of-variance matrix across the inventory, pros/cons per family, dedicated anti-pattern catalog, and (if adopt-vs-build is in scope) scored evaluation against an explicit requirements checklist.

**Tool:** `gemini-query` with `model=pro` and `thinkingLevel=high`. Long outputs are fine — depth matters here. Embed Stage 1's inventory directly in the prompt (condensed; strip citation artifacts that would consume tokens without adding signal).

### Prompt Shape

- Stage 1 inventory, condensed.
- Explicit axes for comparison (data model, persistence strategy, retrieval pattern, lifecycle, etc. — pick axes that matter for the target design's foundations).
- Anti-pattern catalog as a dedicated section, not an afterthought. Ask explicitly: *"What patterns in this inventory have failed in production for the systems that adopted them, and why?"*
- If adopt-vs-build is in scope: a scoring rubric (0 = missing, 1 = partial, 2 = fully meets) against a requirements checklist drawn from the Vision Baseline.
</core>

<guidance>
### Verification — Hedge-Word Detection

Gemini in `pro`/`high` mode will mark hedges and low-confidence claims explicitly when asked. Skim the output for hedge words and low-confidence flags; flagged speculation is honest speculation, and unflagged confidence may be hallucination.

If Stage 2 returns confident claims about systems with no flagged uncertainty, sanity-check at least one of those claims against Brave-search or WebFetch to a primary source. The staged pipeline's value depends on Stage 2 being grounded — a Stage 2 that confidently invents details poisons Stage 3.

If a sanity check reveals fabrication, journal the finding (the topic gets PARTIAL in §recording-the-result) and consider re-running Stage 2 with a tighter prompt.
</guidance>
</section>

<section id="stage-3-application">
<core>
## Stage 3 — Vision-Contextualized Brainstorm

**Goal:** Concrete architectural proposals for the target design, informed by Stages 1 and 2. This is the stage that does the actual creative work — and it can do it well because Stages 1 and 2 gave it a foundation to reason from.

**Tool:** `gemini-brainstorm` with `maxRounds=4` for depth. Populate `claudeThoughts` with substantive synthesis of Stages 1 and 2 plus the tensions, hypotheses, and questions you want chewed on.

### Prompt Shape

- Full vision description (What / Why / How-used from the Vision Baseline).
- Distilled outputs from Stages 1 and 2 (not the raw outputs — the synthesis you produced).
- Specific deliverables to address:
  - Which principles from Stage 2's pros/cons should be incorporated into the target design, and where?
  - For each anti-pattern in Stage 2's catalog, does the current target design avoid it? If unclear, flag it.
  - What novel risks does the target design face that no system in the inventory addresses?
  - What gaps in the design's delta-list does the inventory expose?
  - For each high-risk choice in the target design, what alternatives does the inventory suggest?
  - What creative patterns did the original design pass miss that the inventory makes obvious?

<mandatory>Use `claudeThoughts` to push back on yourself. List tensions you suspect, hypotheses you want tested, questions you think you already know the answer to. Gemini's value at Stage 3 is disagreeing with or refining those hypotheses — not repeating them back to you. A Stage 3 prompt that does not invite disagreement will produce confirmation. The pushback move is what turns Stage 3 from a polish pass into a useful brainstorm.</mandatory>
</core>
</section>

<section id="recording-the-result">
<core>
## Recording the Result

The 3-stage pipeline is a research diversion within the calling phase. It does NOT count as its own phase and does NOT get its own Phase Registry entry. Record it in the decision journal as a *single consolidated Research entry* after Stage 3 completes — not three separate entries, one per stage.

<mandatory>One journal entry, not three. The stage-level outputs live in session context for the duration of the diversion; only the consolidated synthesis and its decisions carry forward to subsequent phases and the Arranger. Writing one entry per stage triples journal noise and obscures the actual decisions the pipeline produced.</mandatory>
</core>

<template follow="format">
## Research: [Topic — landscape survey via staged brainstorm]
**Phase:** [Phase 3 — Vision Expansion | Phase 5 — Approach Loop]
**Question:** [What the staged pipeline was investigating, in one sentence]
**Tools used:** gemini-search (Stage 1 inventory), gemini-query pro/high (Stage 2 synthesis), gemini-brainstorm maxRounds=4 (Stage 3 application)
**Findings:** [Consolidated synthesis across all 3 stages — landscape verdict, anti-patterns avoided, principles carried forward, novel risks surfaced. Concrete enough that the Arranger can assess re-verification need without re-running the pipeline.]
**Decision:** [Adopt-vs-build verdict if applicable, plus the principles or patterns being locked in for the target design.]
**Arranger note:** [VERIFIED — 3-stage Gemini pipeline with grounded Stage 1 completed; or PARTIAL — specify which stage remained speculative or which sanity check failed.]
**Status:** settled
**Supersedes:** [reference to original entry if this re-settles a previously settled topic, or "—"]
</template>
</section>

<section id="exit">
<core>
## Exit

Return to the recorded step in the calling phase per the standard research diversion flow in `references/research-protocol.md` §diversion-flow. The "Research Diversion (in progress)" entry's status updates from `in-progress` to `settled` and the consolidated Research entry above is appended.

The Arranger note convention applies as follows:
- **VERIFIED** if the 3-stage pipeline completed end-to-end with a grounded Stage 1 (used `gemini-search` or equivalent), Stage 2 returned without unflagged-confidence concerns, and Stage 3 produced concrete findings rather than restating Stages 1-2.
- **PARTIAL** if any stage was skipped, abbreviated, or remained speculative — for example: Stage 1 used `gemini-query` without grounded search and the topic's currency matters; Stage 2 sanity-check revealed fabrication; or Stage 3 surfaced an unresolved tension the user chose to defer rather than chase.

The Arranger needs the VERIFIED/PARTIAL signal to decide whether to re-run the pipeline before committing to implementation. Default to PARTIAL when in doubt — over-flagging is recoverable; under-flagging produces silent gaps.
</core>
</section>

</skill>
