<skill>

<sections>
- enrichment-sources
- three-pass-presentation
- discussion-boundary
- ask-user-question-pattern
- gemini-prompt-design
- vision-baseline-reverification
- loop-back-integration
- tabled-items
- journal-template
- exit
</sections>

<section id="enrichment-sources">
<core>
## Enrichment Sources

Three sources generate enrichment suggestions, executed in this order:

### Source 1: Dramaturg Ideation

You generate suggestions from your own understanding of the feature domain and the project context established during Context Grounding:

- **Interaction patterns and UX conventions** — common patterns for the feature type that the user hasn't mentioned. What would a user expect to be able to do?
- **Cross-feature implications** — existing features discovered during Context Grounding that the new design should interact with, extend, or be aware of
- **Edge cases and boundary conditions** — scenarios the user's vision implicitly creates but hasn't addressed
- **Domain-specific "forgot to think about" items** — common oversights for this type of feature

### Source 2: Gemini Creative Ideation

A structured Gemini prompt that builds on your own ideas. See §gemini-prompt-design for prompt construction details.

### Source 3: Merge and Curate

Deduplicate across both sources. Tier into three groups:

- **Strong recommendations** — you are confident these add value to the user's stated goals and use cases
- **Worth asking** — plausible but not certain they fit the user's vision
- **Remainder** — creative ideas that might spark interest but aren't clear fits

Tiering is based on relevance to the user's stated goals and use cases from the Vision Baseline. Ideas that directly serve the user's stated "why" rank higher than clever but tangential suggestions.
</core>
</section>

<section id="three-pass-presentation">
<core>
## Three-Pass Presentation

### Pass 1: Strong Recommendations

Present strong recommendations via AskUserQuestion. Each question is yes/no with enough context for the user to make an informed decision. One question at a time — the same one-at-a-time rule from the rest of the skill.

See §ask-user-question-pattern for question format and usage rules.

### Pass 2: Re-Review and Promote

After Pass 1 completes, re-evaluate the "worth asking" tier against the user's Pass 1 answers. Ideas that gained relevance from the user's enthusiasm pattern get promoted.

<guidance>
Example: If the user says yes to drag-to-reschedule, "snap-to-grid time slots" jumps from "worth asking" to "obviously needed." You reassess remaining ideas against what the user actually responded to, not just your initial assessment.
</guidance>

Present promoted ideas as another AskUserQuestion round.

### Pass 3: Bulleted Remainder

Present the remaining ideas as a grouped list — "Any of these interest you?" The user can pick zero or many. Any that spark interest get brief discussion per §discussion-boundary.
</core>
</section>

<section id="discussion-boundary">
<core>
## Discussion Boundary

Mini-discussions during enrichment are natural and welcome — a "yes" to drag-to-reschedule might prompt the user to say "yeah, and it should snap to 15-minute increments." That's valuable vision context.

But when discussion starts reaching for architectural decisions — "should we use a gesture detector or a draggable widget?" — you flag it: "This is getting into Phase 5 territory — keep exploring now or table it for later?"

This is a gentle redirect, not a hard gate. The user decides whether to continue the tangent or table it.

<mandatory>When a discussion is tabled, record it in the tabled items list for the journal entry. See §tabled-items.</mandatory>
</core>
</section>

<section id="ask-user-question-pattern">
<core>
## AskUserQuestion Usage Pattern

### When to Use

When proposing a capability or interaction the user hasn't discussed yet and the question has a natural yes/no answer. The question must include enough context for an informed decision — not just "Should the planner support drag-to-reschedule?" but "Should the planner support drag-to-reschedule? This would let you grab a task and move it to a different time slot to quickly adjust your schedule."

Each question should contain:
- **The suggestion** — what you are proposing
- **Brief context** — why it's relevant to this feature and what it looks like in practice
- **Your recommendation** — lean yes or lean no, so the user knows your thinking

### When NOT to Use

- Open-ended design questions that need discussion — use normal conversation
- Questions where yes/no would be misleading — use normal conversation
- Follow-up clarification when a "yes" needs more detail — switch to conversation for that thread
- Anything outside Vision Expansion — the rest of the skill uses normal conversational flow

### Transition to Discussion

When a yes/no naturally opens brief discussion, switch seamlessly to conversation. When the thread resolves or gets tabled, return to AskUserQuestion for the next suggestion. No "switching modes" announcements.
</core>
</section>

<section id="gemini-prompt-design">
<core>
## Gemini Prompt Design

### Prompt Structure

The Gemini prompt contains four parts:

1. **Feature context** — the user's confirmed What/Why/How-used in natural language. A conversational description, not journal template format.

2. **Broad tech stack** — language/framework level only (e.g., "Flutter/Dart mobile and web app, Node.js/Express backend, SQLite database"). Not library versions, not architectural patterns, not schema details. Enough to ground suggestions in reality without constraining them.

3. **Ideas already generated** — your own suggestions from Source 1, framed as "here's what we've already thought of" so Gemini builds on them rather than duplicating.

4. **The ask** — explicitly request: interaction ideas, UX patterns users would expect, edge cases worth considering, cross-feature opportunities, and "what would delight a user" creative suggestions. Ask Gemini to organize its response by theme/category to make the merge-and-curate step easier.

### What NOT to Include

- Implementation details or existing codebase structure
- User preferences or constraints beyond the vision
- Full project feature set — only mention existing features directly related to the new feature

### Handling Output

Gemini will return 15-30+ suggestions of varying quality. You curate by removing:
- Suggestions that contradict the user's stated vision
- Implementation details disguised as features
- Duplicates of your own ideas

Remaining items are tiered by relevance to the user's stated goals and use cases per §enrichment-sources.
</core>
</section>

<section id="vision-baseline-reverification">
<core>
## Vision Baseline Re-verification

<mandatory>After all passes and discussions complete, re-check the What/Why/How-used against what was added during enrichment. If enrichment materially changed the vision — which it often will — update and re-present the Vision Baseline summary for user confirmation. This updated baseline is what gets journaled and what Phase 6 tests against for regression.</mandatory>

The re-verification follows the same format as the Vision Loop's gate verification: present the updated What/Why/How-used summary explicitly and ask the user to confirm.

If the Vision Baseline has not materially changed (enrichment added capabilities but didn't alter the core What/Why/How-used), note this and confirm with the user: "The core vision is the same — enrichment added capabilities within the existing scope. Does the original baseline still capture it?"
</core>
</section>

<section id="loop-back-integration">
<core>
## Loop-back Integration

When Phase 6 (Review Loop) detects a vision change and routes back to Phase 2 (Vision Loop):

1. The user re-establishes their vision through normal elicitation
2. After the Vision Loop gate passes, Vision Expansion runs again — mandatory
3. Your ideation and Gemini prompt incorporate the changed vision
4. Previously accepted enrichments that conflict with the new vision are dropped; compatible ones carry forward without re-asking

<mandatory>Vision Expansion is mandatory after every Vision Loop pass, including loop-backs. The vision shift may have created new enrichment opportunities that weren't relevant before.</mandatory>
</core>
</section>

<section id="tabled-items">
<core>
## Tabled Items

Items tabled during Vision Expansion — discussions that drifted toward Phase 5 territory — should be checked against the Topic Map when Phase 5 begins topic selection. They may need to become topics or fold into existing ones.

The journal entry captures all tabled items with the reason they were tabled.
</core>
</section>

<section id="journal-template">
<core>
## Journal Entry

A single journal entry captures the enrichment results. Written once after the exit gate, before transitioning to Broad Design Scoping.

<mandatory>Write the Vision Expansion journal entry before proceeding to Broad Design Scoping. This entry captures what was suggested, accepted, rejected, and tabled — providing Phase 5 and the Arranger visibility into enrichment decisions.</mandatory>

### Why Capture Rejections

The Arranger benefits from knowing what was explicitly considered and rejected. This prevents downstream skills from re-litigating settled decisions.

### Why Capture Tabled Items

Tabled items are discussions intentionally deferred to Phase 5. They should be checked against the Topic Map during Approach Loop topic selection.
</core>

<template follow="format">
## Vision Expansion
**Phase:** Phase 3 — Vision Expansion
**Sources:** Dramaturg ideation, Gemini creative ideation
**Accepted:**
- [capability] — [brief context of why user accepted]
- [capability] — [brief context]
**Rejected:**
- [capability] — [user's reason, or "not relevant to goals"]
**Tabled:**
- [capability] — [reason tabled, e.g., "needs Phase 5 exploration"]
**Vision Baseline updated:** [yes/no — whether the What/Why/How-used changed]
**Status:** settled
</template>
</section>

<section id="exit">
<core>
## Exit

Simple confirmation: "That covers the expansion — ready to move on to scoping?" No strict verification beyond the Vision Baseline re-check (§vision-baseline-reverification).

Proceed to Phase Registry → Broad Design Scoping.
</core>
</section>

</skill>
