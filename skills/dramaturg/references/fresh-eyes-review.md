<skill>

<sections>
- anchoring-problem
- three-agent-pattern
- scoping
- evaluation-criteria
- when-to-use
- teammate-spawning
- behavioral-reviewer
- technical-reviewer
- three-agent-discussion
- scaling-per-domain
- journal-template
- exit
</sections>

<!-- Loading guide:
  - §anchoring-problem, §three-agent-pattern: Load on Phase 6.5 entry
  - §scoping: Load BEFORE §evaluation-criteria so the Dramaturg-specific framing of the criteria is set
  - §evaluation-criteria: Load BEFORE deciding whether to invoke Phase 6.5
  - §when-to-use: Narrative companion to §evaluation-criteria for mid-session reading
  - §teammate-spawning, §behavioral-reviewer, §technical-reviewer: Load when invoking the phase
  - §three-agent-discussion: Load when teammates return findings
  - §scaling-per-domain: Load when the design is large enough to warrant per-domain teammates
  - §journal-template: Load when writing the evaluation-decision entry or any findings entry
  - §exit: Load when all findings have been resolved or deferred
-->

<section id="anchoring-problem">
<core>
## The Anchoring Problem

By the time the design document is complete, you have spent the entire session building the mental model that justifies its decisions. You cannot effectively review your own work — you share the same assumptions that produced the flaws. This is the "marking your own homework" problem.

The flaws Phase 6.5 catches are rarely in obscure edge cases. They sit in core assumptions that the designing session has internalized and stopped questioning. A reader without your context spots them quickly because they are not anchored by those assumptions.

Phase 6.5 introduces a third party that does not share the session's accumulated context. The value is specifically in the lack of shared context — the reviewer questions things you and the user have stopped questioning.
</core>
</section>

<section id="three-agent-pattern">
<core>
## The Three-Agent Pattern

Phase 6.5 is a three-agent design review:

1. **You (the Dramaturg)** — coordinator and relay. Hold full context. Cross-reference teammate findings against the decision journal so the user can see what each finding implies for previously-settled decisions.
2. **The reviewer teammate(s)** — fresh-context Claude or Gemini, spawned with ONLY the design document. No conversation history, no decision journal, no awareness of the design process. The lack of context is the feature, not a limitation.
3. **The user** — participates interactively. Provides constraints, empirical evidence, and decisions. Not a passive reviewer of conclusions.

This is structurally different from Phase 6 Review Loop. Phase 6 is you presenting your own work to the user — both parties share the session's accumulated context. Phase 6.5 introduces a participant who does not share that context.
</core>

<guidance>
### Why a Teammate Within This Session, Not a Separate Session

A separate Claude Code session would force the user to manage two terminals, two contexts, and manually relay information between them. Spawning a teammate within the current session lets you act as relay/coordinator while the teammate has its own bounded context window — clean enough to review without inheriting your assumptions, but reachable from your seat at the conversation.

The 1M context window does not change this calculation. The teammate's value is its independent context, not its size. Putting the reviewer in a separate session imposes coordination overhead without adding review quality. Putting the reviewer in your own context destroys the entire point of the pattern — you would be marking your own homework again. The teammate-within-session shape is load-bearing.
</guidance>
</section>

<section id="scoping">
<context>
## Scoping — Criteria Apply to Dramaturg Artifacts

The §evaluation-criteria block below is implemented against Dramaturg-specific artifacts: Topic Map entries, decision journal entry types (Decision, Research, Tension), `Status: settled`, `Supersedes:`, `User verbatim`, and Vision Baseline. Future cycles wanting to apply the Phase 6.5 fresh-eyes pattern to non-Dramaturg artifacts (refactor PRs, technical proposals, plan reviews, skill-rework cycles) will need an artifact-agnostic abstraction layer — for example, "complexity signal: count of decision points," "complexity signal: presence of acknowledged unresolved tensions," or domain-specific equivalents.

The pattern's *intent* generalizes (fresh reviewer, no shared context, binary auditable gate). This *implementation* of the gate is Dramaturg-specific by design. When applying the pattern outside a Dramaturg session, treat the criteria as a template — preserve the binary-mechanical structure and the mandatory-evaluation-decision-entry discipline, but rewrite each criterion's `*Verify:*` clause against the target domain's artifacts.
</context>
</section>

<section id="evaluation-criteria">
<core>
## Evaluation Criteria

Phase 6.5 is recommended-not-mandatory: simple designs do not benefit from the overhead, and the value scales with complexity. The *evaluation* of whether to invoke is mandatory; the *execution* is conditional on what the evaluation finds.

The criteria below are mechanically answerable from artifacts you have already produced (Topic Map, decision journal, Vision Baseline). No criterion depends on "Dramaturg's judgment" or "if it feels complex" — that wiggle room lets skip-by-default win and the gate self-deceives.

### Phase 6.5 SHOULD be invoked if any of the following criteria fire:

1. **Topic Map has 4 or more settled approaches that interact.** *Verify:* count Decision and Research entries with `Status: settled` in the journal; "interact" means any topic that appears in the `Alternatives-discussed` field of another topic's Decision/Research entry, OR in a Tension entry's `Requirements in tension` list. Purely textual — no judgment call. The criterion fires when the count reaches 4 *and* at least 2 of those settled topics meet the textual interaction test.
2. **Design touches concurrency, multi-process, or distributed-system concerns.** *Verify:* keyword-scan the Topic Map and Vision Baseline for any of: `concurrent`, `concurrency`, `multi-process`, `multi-agent`, `distributed`, `parallel`, `async`, `race`, `lock`, `WAL`, `OCC`, `transaction`, `sidecar`, `daemon`. One match is sufficient.
3. **Design will be used by Claude as a tool or system.** *Verify:* the Vision Baseline What/How-used entries name a Claude-facing artifact — `MCP server`, `hook system`, `tool interface`, `skill`, `agent`, `subagent`, `plugin`, or similar. One match is sufficient.
4. **Journal contains 2 or more entries with `Status: ruled-out` or non-empty `Supersedes:` references.** *Verify:* grep the journal for `Status: ruled-out` and for `Supersedes:` lines whose value is not the literal `—` placeholder. Counts of 2 or more across either field combined trigger the criterion. (Significant in-session evolution = more accumulated assumptions = more value from fresh eyes.)
5. **User provided empirical evidence captured verbatim in the journal.** *Verify:* grep the `User verbatim` and `User context` fields for any of: `tried`, `didn't work`, `broken`, `failed when`, `fell over`, `crashed`, `couldn't`, `wouldn't`, `had to`. One match is sufficient. The teammate can catch when the design does not account for the user's lived experience.
6. **Journal contains 1 or more Tension entries with `Status: acknowledged` and a non-default resolution.** *Verify:* grep the journal for `## Tension:` headings and check the corresponding `Current resolution approach:` field. Any Tension entry whose resolution-approach is "unresolved — Arranger decides" or whose status remains acknowledged without an in-design resolution fires this criterion. (Tensions journaled in Phase 5 indicate productive conflicts that the design has not fully resolved — fresh-eyes review can surface whether the design's chosen resolution holds under independent scrutiny.)

### Phase 6.5 MAY be skipped only if all of the following are true:

- Topic Map has 2 or fewer settled approaches.
- No concurrency, multi-process, or multi-agent concerns appear in the Topic Map or Vision Baseline (per criterion 2's keyword set).
- Design is not a Claude-facing tool or system (per criterion 3's keyword set).
- Journal shows no `Supersedes:` entries (i.e., no settled topic was re-settled during the session) and at most one `ruled-out` entry.
- Journal shows no Tension entries (or all Tension entries have in-design resolutions other than "unresolved — Arranger decides").

If any SHOULD-invoke criterion fires, you invoke Phase 6.5. The MAY-skip block is the *negation* of the SHOULD-invoke set, restated for legibility — it is not an independent gate. If you find yourself looking for reasons to skip when a SHOULD-invoke criterion fires, the gate has self-deceived.

<mandatory>The evaluation itself is mandatory regardless of whether execution proceeds. Every Phase 6.5 transition writes an evaluation-decision journal entry per §journal-template, capturing which criteria fired and the resulting decision (invoke or skip). This makes the evaluation auditable by the user, by Reconciliation, and by the Arranger downstream. Skipping the evaluation entry — even when the decision is "skip" — is the failure mode this rule prevents.</mandatory>
</core>
</section>

<section id="when-to-use">
<guidance>
## When to Use — Narrative Companion

This section is the conversational version of §evaluation-criteria for a Dramaturg reading mid-session. The formal answer is binary and lives there; this is the why.

### Always Recommended

- **Designs for systems Claude will use as a tool** (MCP servers, hook systems, tool interfaces, skills, plugins). The behavioral self-assessment from a fresh Claude reviewer is uniquely valuable here — it is the only reliable way to predict actual usage patterns under context pressure.
- **Designs with concurrency, multi-process, or distributed-system concerns.** Fresh technical review catches assumption-based architecture flaws that the original session's research smoothed over.
- **Designs that evolved significantly during the session** — many approach changes, vision revisions, or supersedes references in the journal. More evolution means more accumulated assumptions, which means more value from fresh eyes.

### Recommended for Complex Designs

- **Designs with 4+ settled approaches where 2+ are textually interacting** — i.e., topics appearing in each other's `Alternatives-discussed` fields or in shared Tension entries (per criterion 1's mechanical test). Interaction effects are invisible to individual approach reviews. A reviewer looking at the whole design sees couplings that you stopped seeing once each topic settled.
- **Designs where the user provided empirical evidence** ("I tried X and it didn't work") that may not be fully reflected in the design. The reviewer can catch when the design does not account for the user's experience.

### Optional / Skip

- Simple designs with 1-2 approaches and no concurrency or multi-agent concerns.
- Designs where the user has deep domain expertise and has already caught the major issues during Phase 6.

Cross-reference: every "always recommended" tier here corresponds to one or more SHOULD-invoke criteria in §evaluation-criteria. The criteria are the binding answer — this section exists so the criteria are not a bare checklist disconnected from the reasoning behind them.
</guidance>
</section>

<section id="teammate-spawning">
<core>
## Teammate Spawning

<mandatory>Spawn the reviewer as a teammate within the current Dramaturg session. Do not direct the user to start a separate Claude Code session. The whole point of the pattern is that you act as relay/coordinator so the user participates in one conversation, not two.</mandatory>

<mandatory>Provide ONLY the finalized design document to the teammate. Do NOT provide:
- The decision journal (anchors the teammate to the "why" behind decisions)
- The conversation history (anchors the teammate to inherited assumptions)
- Your synthesis or summary of the design (anchors the teammate to your framing)

The teammate sees the artifact as a cold reader would encounter it. The lack of context is the feature.</mandatory>

### Spawning Protocol

1. Confirm Phase 6.5 invocation with the user (cite the SHOULD-invoke criteria that fired per §evaluation-criteria).
2. Spawn one or more reviewer teammates per §behavioral-reviewer and §technical-reviewer. For large designs, spawn parallel per-domain teammates per §scaling-per-domain.
3. Pass the design document only. The teammate prompt frames the reviewer's perspective and lists the questions to address — see the per-reviewer sections.
4. Wait for findings. Each finding becomes a journal entry per §journal-template.
5. Convene the three-agent discussion per §three-agent-discussion.
</core>
</section>

<section id="behavioral-reviewer">
<core>
## Behavioral Reviewer (Claude Teammate)

**Purpose:** Review the design from the perspective of the AI that will use the system being designed. Behavioral self-assessment that the designing session cannot provide.

**Tool:** Claude teammate, spawned within this session with the design document only.

### Prompt Pattern

Frame the teammate as the user of the system, not as a reviewer of the design. The framing produces qualitatively different output — the teammate stops critiquing the prose and starts predicting its own behavior.

Required questions for the teammate to address:

- "Would I actually use this consistently, or would I default to something simpler under time pressure?"
- "What is the cognitive overhead at point-of-use?"
- "What happens when my context gets compacted mid-task — does this design still work?"
- "Where does this design assume I will behave consistently, and is that assumption realistic?"
- "What will I do when I am unsure whether to invoke this — invoke it, skip it, or ask?"

### Reporting Shape

Ask the teammate to return findings as:
- The behavior the design assumes
- The behavior the teammate predicts under realistic conditions
- The gap, if any
- Severity classification: breaking / oversight / assumption-flaw
</core>

<guidance>
### Why This Works

Claude reviewing its own tool interfaces produces uniquely honest behavioral self-assessment. You cannot do this from the designer's seat — you see the tools as intended. The reviewer sees them as they would be experienced. Honest probability estimates from the reviewer ("I would use the promote tool maybe 55-65% of the time, drop to 15-20% under context pressure") are the most actionable data the phase produces. They are not available from any other source — not the user, not Gemini, not your own introspection.

Trust low numbers. A reviewer reporting 60% reliability on a behavior the design assumes is 100% reliable is the finding — do not argue the number up.
</guidance>
</section>

<section id="technical-reviewer">
<core>
## Technical Reviewer (Gemini in Fresh-Architect Mode)

**Purpose:** Independent technical architecture review — infrastructure, concurrency, failure modes, scalability. Fresh perspective on the *complete* design rather than answering specific research questions in your frame.

**Tool:** Gemini via MCP, with the finalized design document as input.

### How This Differs from Approach Loop Gemini Usage

During Phase 5, Gemini answers specific research questions you frame ("Does X work with Y?"). The frame is set by your already-formed mental model.

In Phase 6.5, Gemini reviews the COMPLETE design as an independent architect with no prior framing. Different prompt, different value. The technical reviewer should not be asked the same questions you asked Gemini during research — those questions were already answered in your frame, and reasking them in the same frame will not surface new problems.

### Prompt Pattern

Required focus areas:

- Infrastructure assumptions — what does the design assume about its runtime environment, and are those assumptions defensible?
- Concurrency and multi-process behavior — race conditions, locking, shared-state coordination
- Failure modes — what breaks first under load, network partition, partial failure?
- Interactions between settled approaches — couplings the design treats as independent that may not be
- Scalability boundaries — does the design have implicit capacity limits the user has not flagged?

Ask for findings classified by severity: breaking / oversight / assumption-flaw.
</core>
</section>

<section id="three-agent-discussion">
<core>
## Three-Agent Discussion

When the teammate(s) return findings, do not present them as a one-shot dump for the user to react to. Convene the three-agent discussion: you relay, the user provides constraints and empirical evidence, the teammate provides independent assessment, and the loop iterates per finding until resolution.

### Flow

1. **Sort findings by severity.** Breaking issues first, then assumption flaws, then oversights. Within a severity tier, sort by which previously-settled decision the finding affects (foundational decisions first).
2. **For each finding, you present it cross-referenced against the decision journal.** State the finding plainly. State which journal entry the finding contradicts or affects. State the reasoning the original entry recorded for the decision (the "why" the teammate did not have).
3. **Highlight contradictions explicitly.** "The reviewer flags X as breaking. Our journal entry on [topic] settled X because [reason]. The reviewer's argument is [argument], which the original reasoning did not address."
4. **User provides constraints or evidence.** The user knows their lived experience and constraints; they can confirm or refute the reviewer's argument from outside the design's framing.
5. **Teammate provides independent assessment if needed.** When the user disagrees with the reviewer's framing, you can pass the user's response back to the reviewer for a follow-up. Iterate until the finding resolves: either the design changes, the design holds with new journal evidence, or the finding defers to the Arranger.
6. **Resolution is collaborative, not top-down.** The reviewer is not authoritative — it lacks context. The user is not authoritative — they may share your anchoring. You are not authoritative — same problem. The synthesis across all three is what produces the resolution.

### Resolution Outcomes

For each finding, the discussion produces exactly one outcome:

- **Resolved in Phase 6.5** — the design changes. Route back through Phase Registry → Review Loop with the affected sections. The Section Approval entries for affected sections are invalidated; re-review is required. Journal the finding entry with `resolution-status: resolved-in-review-loop`.
- **Deferred to Arranger** — the finding is real but cannot be resolved at the design level (it requires implementation-level investigation, or it is an acceptable risk the user has chosen to take). Journal with `resolution-status: deferred-to-arranger` and ensure the deferred finding propagates into Reconciliation's Arranger Notes.
- **Held without change** — the design holds because the original reasoning addresses the reviewer's concern, and the user has confirmed this. Journal the finding entry anyway (so the rejection is auditable) with `resolution-status: held-with-justification` and capture the user's confirmation in the entry.

<mandatory>Every finding gets a journal entry, even findings that were rejected. The Arranger and any future skill-rework cycle need to see what was raised, not just what stuck. Silently dropping findings the discussion rejected destroys the audit trail.</mandatory>
</core>
</section>

<section id="scaling-per-domain">
<guidance>
## Scaling Per Domain

For large designs, multiple review teammates can be spawned in parallel, each focused on a specific domain:

- One teammate per major design section (data model / protocol / lifecycle / security)
- One cross-cutting teammate for interaction effects between sections
- For very large designs (e.g., systems with many tag domains, entry types, or sub-features), teammates can be further specialized by sub-feature with a coordinator handling cross-domain concerns

You synthesize teammate findings before presenting to the user. Do not relay six teammate outputs serially — the user will lose track. Synthesize first: group findings by which settled decision they affect, dedupe identical findings raised by multiple teammates (these are signals — flag them as such), and order by severity.

This is an optimization for designs whose surface area exceeds what one teammate can review thoroughly within its context. For most designs, one behavioral reviewer plus one technical reviewer is sufficient. The per-domain pattern is invoked when a single teammate's review feels skimming-rather-than-deep on multiple sections.
</guidance>
</section>

<section id="journal-template">
<core>
## Journal Templates

Phase 6.5 writes two kinds of journal entries:

1. **Evaluation Decision** — written every time Phase 6.5 is *evaluated*, regardless of whether it executes. Mandatory per §evaluation-criteria.
2. **Fresh Eyes Findings** — written per finding when Phase 6.5 *executes*. One entry per finding.

The Evaluation Decision entry exists so a reader of the journal can confirm the gate fired correctly, even when the decision was "skip." Without it, a future Arranger or skill-rework reader cannot distinguish "Phase 6.5 was correctly skipped" from "Phase 6.5 was silently bypassed."
</core>

<template follow="format">
## Phase 6.5 Evaluation Decision
**Phase:** Phase 6.5 — Fresh Eyes Review (evaluation)
**Criteria fired:** [list of SHOULD-invoke criterion numbers/names that fired, with the verifying counts/keywords. Use "—" if none fired.]
**Criteria not applicable:** [list of criteria that were vacuously not-fired due to phase ordering or absence of the relevant artifact class. Use "—" if all criteria were genuinely evaluable.]
**Decision:** [invoke | skip]
**Reasoning:** [one to two sentences explaining the decision. If invoke: which criteria fired and the counts. If skip: confirmation that all MAY-skip conditions hold.]
**Status:** settled
</template>

<template follow="format">
## Fresh Eyes Finding: [short label]
**Phase:** Phase 6.5 — Fresh Eyes Review
**Reviewer perspective:** [behavioral | technical | per-domain (specify domain)]
**Finding:** [the substance of what the reviewer raised, in concrete terms]
**Severity:** [breaking | oversight | assumption-flaw]
**Original decision affected:** [reference to the journal entry/entries this finding contradicts or affects, e.g., "Decision: WAL Mode for Multi-Process SQLite"]
**Discussion summary:** [user constraints, teammate follow-up, reasoning that drove the resolution]
**Resolution status:** [resolved-in-review-loop | deferred-to-arranger | held-with-justification]
**Resolution detail:** [if resolved-in-review-loop: which sections re-review covers. If deferred-to-arranger: the specific Arranger Note that should carry forward. If held-with-justification: the user's confirmation and the existing journal entry whose reasoning stands.]
**Status:** settled
</template>
</section>

<section id="exit">
<core>
## Exit

Phase 6.5 is complete when ALL of the following are true:

1. The Phase 6.5 Evaluation Decision journal entry exists (always — even if the decision was skip).
2. If invoked: every finding raised by every spawned teammate has a Fresh Eyes Finding journal entry with a non-empty `Resolution status`.
3. If invoked: every finding with `resolution-status: resolved-in-review-loop` has been routed back through Phase Registry → Review Loop and the affected sections have new Section Approval entries reflecting the post-revision state.
4. If invoked: every finding with `resolution-status: deferred-to-arranger` is flagged for inclusion in Reconciliation's Arranger Notes.

When all conditions are true, proceed to Phase Registry → Reconciliation. Reconciliation consumes Section Approval entries plus (if Phase 6.5 was invoked) Fresh Eyes Finding entries — substantive deferred findings carry forward into the final design doc's Arranger Notes section.
</core>
</section>

</skill>
