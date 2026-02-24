<skill>

<sections>
- question-strategy
- gate-verification
- loop-back-conditions
</sections>

<section id="question-strategy">
<core>
## Question Strategy

The Vision Loop draws out What/Why/How-used entirely from the user's own words. Each dimension requires a different questioning approach because users volunteer them differently.

### Drawing Out "What"

Usually the easiest — users typically arrive with at least a rough "what." The Dramaturg's job is to expand and clarify, not to accept the first description as complete.

- Ask about scope: "When you say [feature], what does that include?"
- Ask about specifics the user glossed over: "You mentioned [thing] — tell me more about what that looks like."
- Do not assume the user's first description is the full picture. Follow up on anything that sounds like it has unexplored depth.

### Drawing Out "Why"

The hardest dimension. Users often skip straight from "what" to "how" without articulating motivation. The "why" matters because it distinguishes features that look identical but serve different purposes — and different purposes lead to different designs.

- Ask about the problem: "What's the situation today that makes you want this?"
- Ask about the trigger: "What happened that made you think of this feature?"
- Ask about impact: "If this worked exactly as you imagine, what changes for you?"
- Do not infer "why" from project context. The user's actual motivation may surprise you.

### Drawing Out "How-Used"

Requires concrete scenario questions. Abstract use-case descriptions ("users will manage their tasks") are not useful — the Dramaturg needs specific scenarios that reveal the user's mental model.

- Ask for a walkthrough: "Walk me through a typical day using this feature."
- Ask for edge cases: "What's the least obvious way you'd use this?"
- Ask for specifics: "You said [scenario] — what exactly happens when [specific moment]?"
- Concrete scenarios reveal constraints that abstract descriptions hide.

<mandatory>One question at a time. Open-ended questions only. Do not present checklists, forms, or multi-question dumps. Each question should invite the user to talk, not to select from options.</mandatory>
</core>

<core>
### Research During Vision Gathering

The user's vision statements frequently contain implicit technical assumptions and feasibility-dependent claims. These must not pass through unvalidated.

<mandatory>When the user's vision includes specific technical approaches, platform capabilities, or integration assumptions — research them via SKILL.md §research-protocol before continuing the vision discussion. "I want WiFi-based indoor positioning" contains a feasibility claim. "It should sync across devices in real-time" assumes infrastructure. "The app should work offline and sync when reconnected" implies an architecture. Do not wait until Phase 5 to discover that the vision's foundation is infeasible. If research reveals a feasibility issue, surface it immediately: "Before we go further, I checked on [claim] and found [issue]. This affects the vision because..." The user deserves to know early.</mandatory>

Research during the Vision Loop is targeted, not exhaustive. Research specific claims and assumptions, not the entire feature space. The goal is to ensure the vision is grounded in reality, not to explore implementation approaches (that's Phase 5).

<mandatory>
**When to research in Phase 2 vs. deferring to Phase 5:**
- User says "I want push notifications" → defer. This is a well-understood capability. Phase 5 explores the approach.
- User says "I want WiFi triangulation for indoor positioning" → research now. This is a specific technical claim with real feasibility constraints.
- User says "it should work without internet" → clarify intent first (what does "without internet" mean to them?), then research the specific architecture they describe.

The test: does the user's statement contain a feasibility assumption that, if wrong, would invalidate the vision itself? If yes, research now. If the statement is about desired behavior that could be achieved multiple ways, defer to Phase 5. When uncertain, research.
</mandatory>
</core>
</section>

<section id="gate-verification">
<core>
## Gate Verification

The Vision Loop gate is strict. It is the one phase transition that should be explicit rather than fluid, because the consequences of moving forward without genuine understanding are severe — the entire design may be built on a misread of the user's intent.

### The Three Questions

Before transitioning, the Dramaturg must be able to answer all three from user input alone:

1. **What** does the user want to build?
2. **Why** do they want to build it? (the motivation, the problem it solves)
3. **How** will they use it? (concrete use cases, user scenarios)

<mandatory>These answers must come strictly from what the user has said. Do not infer the "why" from project context. Do not fill in use cases from training data. Do not count the Dramaturg's own reasoning as user input. If any answer is missing or relies on inference, keep asking.</mandatory>

### Adapted Gate for Non-Feature Work

The What/Why/How-used gate assumes user-facing features. For system qualities (error handling, performance), quality initiatives (test coverage), and infrastructure work, the "How-used" question is meaningless. Use this adapted gate instead:

1. **What** change is being made?
2. **Why** is it needed? (the motivation, what's wrong or missing today)
3. **How measured** — how will success be evaluated? (metrics, acceptance criteria, observable improvements)

<mandatory>These adapted answers must still come from the user's own words. The adapted gate has the same verification procedure and strictness as the standard gate.</mandatory>

### Verification Procedure

When the Dramaturg believes all three questions are answered:

1. **Summarize explicitly.** Present the understanding back to the user in a structured summary: "Here's what I understand about your vision — **What:** [summary]. **Why:** [summary]. **How you'll use it:** [summary]."
2. **Ask for confirmation.** "Does this capture it? Is there anything I'm missing or getting wrong?"
3. **If the user corrects or adds** — incorporate the correction, re-present the full updated summary (not just the changed part), and ask for confirmation again. The gate doesn't pass until the user confirms the complete summary.
4. **If the user confirms** — write the Vision Baseline journal entry, then transition.

<mandatory>The explicit summary and confirmation is not optional. Do not transition to Broad Design Scoping without the user confirming the summary. This prevents the most common planning failure: racing ahead with an assumed understanding.</mandatory>

### Vision Baseline Journal Entry

<mandatory>Write the Vision Baseline journal entry immediately after user confirmation, before transitioning. This entry is the regression test anchor — Phase 6 compares against it when detecting vision regression. A missing or incomplete baseline makes regression detection impossible.</mandatory>
</core>

<template follow="format">
## Vision Baseline
**Phase:** Phase 2 — Vision Loop
**What:** [What the user wants to build — comprehensive, from their words]
**Why:** [Motivation, problem it solves — the user's actual reasoning, not inferred]
**How used:** [Concrete use cases, user scenarios — specific, not abstract]
**User confirmed:** [yes — must be yes]
**Research during vision:** [feasibility checks and results, or "none"]
**Status:** settled
</template>

<core>
### Status Values

Journal entries use these status values:
- **settled** — decision made, user confirmed
- **confirmed** — user confirmed a factual/coverage checkpoint (e.g., Topic Map coverage)
- **acknowledged** — tension or tradeoff recognized, not resolved
- **in-progress** — research diversion currently active
- **ruled-out** — approach researched and found infeasible

### Exit

Proceed to Phase Registry → Vision Expansion.
</core>
</section>

<section id="loop-back-conditions">
<core>
## Loop-Back Conditions

Two paths lead back into the Vision Loop.

### Loop-Back to Context Grounding

During vision gathering, the user may reveal project context that the Dramaturg hasn't explored. Examples:
- "This needs to integrate with the notification system" — and notifications haven't been explored
- "It should build on the existing scheduling feature" — and that feature's architecture is unknown
- "The backend already has something similar" — and that similarity hasn't been verified

When this happens, the vision discussion is missing foundational context. Route to Phase Registry → Context Grounding to explore the relevant area, then return here to continue the vision discussion. Do not attempt to continue vision gathering while guessing about project context.

Before looping back, note the current position in the vision discussion: what questions have been answered, what remains. On return from Context Grounding, resume from where the vision discussion left off rather than restarting.

### Re-Entry from Review Loop

If vision regression is detected and confirmed during Phase 6 section review, the Review Loop routes back here through the Phase Registry. On re-entry:

- The user has provided feedback that changes the answer to What, Why, or How-used
- Previously approved sections remain in place — they will be re-evaluated during Reconciliation (Phase 7)
- Re-establish the vision with the updated understanding using the same question strategy and gate verification
- <mandatory>After re-establishing: proceed through Vision Expansion (Phase 3) and then Broad Design Scoping (Phase 4). Both are mandatory — the vision shift may have created new enrichment opportunities, and the design territory may have shifted.</mandatory>
</core>
</section>

</skill>
