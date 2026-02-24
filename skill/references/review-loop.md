<skill>

<sections>
<!-- Presentation -->
- section-presentation
<!-- Feedback handling -->
- feedback-classification
- vision-regression
- approach-change-routing
<!-- Quality -->
- security-lens
<!-- Completion -->
- section-approval
- exit
</sections>

<section id="section-presentation">
<core>
## Section Presentation

Present the detailed design as structured sections for focused review. By this point, the approaches are already settled in the Approach Loop — this phase is about the details within those approaches. Feedback here is typically detail-level, not approach-level.

### Writing Sections

Each section is a detailed writeup of a settled approach from Phase 5, including:
- Data schemas and formats
- Error handling and edge cases
- Interaction patterns and flows
- Constraints and assumptions
- How the section connects to other sections

<mandatory>Present one section at a time. One section per message — never present multiple sections in a single reply. Multi-topic reviews lose detail. Single-section reviews keep the scope focused and ensure every important point gets proper attention.</mandatory>

<guidance>
"One section per message" means one logical section per review turn. If a section is complex and spans multiple messages, that is one section reviewed across multiple turns — the constraint is that the user reviews one section at a time, not that one section must fit in a single message. If a section is too large for a single message, split the presentation across messages but keep the review focused on that one section before moving to the next.
</guidance>

### Section Breakdown

Before presenting the first section, establish how settled topics from Phase 5 map to design sections. Topics don't map 1:1 to sections — a single topic might span multiple sections, or multiple topics might collapse into one section.

Present the proposed section breakdown to the user for confirmation: "Based on our settled approaches, I'm planning to present these sections in this order: [list]. Does this cover everything, or should anything be split or combined?"

<mandatory>
### Section Ordering

Sequence sections by dependency, foundations first:
- Architecture and data model before UI behavior
- Core mechanics before error handling
- Shared infrastructure before features that depend on it

Foundations-first prevents unnecessary rework if a detail change in a foundational section ripples outward to later sections.
</mandatory>
</core>
</section>

<section id="feedback-classification">
<core>
## Feedback Classification

Every piece of user feedback during review falls into exactly one of four categories. Classify before acting — the category determines the response.

| Feedback Type | Detection | Action |
|---|---|---|
| **Vision change** | Alters the answer to What/Why/How-used | See §vision-regression |
| **Approach change** | Same goal, different technical method | See §approach-change-routing |
| **Detail change** | Refinement within the current approach | Revise the section in place |
| **Approved** | User accepts the section | See §section-approval |

### Detail Changes

The most common feedback type. The user wants the same approach but with adjustments — a different timeout value, an additional edge case handled, a clarified interaction pattern. Revise the section in place and re-present the changed portion for confirmation. No phase transition needed.

### Ambiguous Feedback

Sometimes feedback sits on the boundary between categories. "What if we used a different message format?" could be a detail change (different serialization) or an approach change (different communication pattern entirely). When ambiguous, ask the user to clarify before classifying: "Is this a tweak to how we serialize the data, or are you thinking of a different communication approach entirely?"
</core>
</section>

<section id="vision-regression">
<core>
## Vision Regression Detection

The most subtle classification — detecting when user feedback during review actually changes the fundamental vision, not just the implementation approach.

<mandatory>Before testing for vision regression: retrieve the Vision Baseline journal entry from the decision journal. Compare feedback against the journal entry, not against conversation memory. The baseline is the authoritative anchor — conversation context may have compacted or drifted. If the Vision Baseline entry is missing or unclear, ask the user to clarify the original baseline rather than reconstructing from memory.</mandatory>

### The Three-Question Test

Compare the feedback against the Vision Baseline journal entry. Does the feedback change the Dramaturg's answer to any of these:

1. **What** does the user want to build?
2. **Why** do they want to build it?
3. **How** will they use it?

### Detection Examples

- "Actually, this should also work offline" — changes **how they'll use it**. The use cases are now different. Vision change.
- "I don't think we need the scheduling part anymore" — changes **what** they want to build. Vision change.
- "The real problem isn't notifications, it's knowing where I am" — changes **why** they want it. Vision change.
- "Use WebSockets instead of SSE" — same goal, same use cases, different technical method. NOT a vision change (approach change).
- "Make that timeout 30s instead of 10s" — same approach, minor parameter. NOT a vision change (detail change).

### User Confirmation Protocol

Vision regression detection has false positives. What looks like a vision change may be poor phrasing of a detail change.

1. **Surface the detection:** "This sounds like it changes the original vision — you initially wanted [What/Why/How from baseline], and this feedback suggests [new direction]. Should we revisit the vision?"
2. **User confirms or denies.** The user decides whether regression is real.
3. **If confirmed:** Route to Phase Registry → Vision Loop. Previously approved sections' Section Approval journal entries are preserved. These sections will be re-evaluated during Reconciliation (Phase 7) against the updated vision — Reconciliation checks whether each section's design is still consistent with the new What/Why/How-used.
4. **If denied:** Continue in the current phase. Reclassify the feedback as a detail change or approach change per the user's clarified intent.

### Scope of Re-exploration After Vision Change

When routing back to Vision Loop after a confirmed vision change:

- The full mandatory path applies: Vision Loop → Vision Expansion (Phase 3) → Broad Design Scoping (Phase 4) → Approach Loop (Phase 5)
- **Topics that existed in both old and new Topic Maps whose approaches are still valid** need re-confirmation, not full re-exploration. The Dramaturg presents the existing settlement and asks: "This approach still works with the updated vision — confirmed?"
- **Topics that are new or fundamentally affected by the vision change** need full exploration through the standard §research-discuss-settle loop
- Old journal entries are preserved (the journal is append-only) but new entries supersede them
- Previously accepted Vision Expansion enrichments that conflict with the new vision are dropped; compatible ones carry forward without re-asking

<mandatory>Do not route to Vision Loop without user confirmation. The confirmation step prevents wasted rework on false positives.</mandatory>
</core>
</section>

<section id="approach-change-routing">
<core>
## Approach Change Routing

An approach change means the user wants the same outcome but through a different technical method. The goal hasn't changed, but the path to it has.

### Detection

- The feedback proposes a different technology, protocol, architecture, or pattern for the same design goal
- The existing section's logic or structure would fundamentally change, not just adjust parameters
- The user is reacting to the detailed writeup by reconsidering the approach that was settled in Phase 5

### Routing

When an approach change is detected:

1. Confirm with the user: "It sounds like you want to change the approach for [topic] from [current] to [proposed]. Is that right?"
2. On confirmation, route to Phase Registry → Approach Loop.

<mandatory>Do not execute the ripple assessment here. The Approach Loop's §ripple-assessment contains the full protocol for identifying affected topics, scoping invalidation with the user, and re-exploring affected topics. This section detects and routes — the Approach Loop executes.</mandatory>

<mandatory>Route through the Phase Registry, not directly to approach-loop.md. The Phase Registry re-bootstraps the Dramaturg with fresh framing and constraints before entering the Approach Loop.</mandatory>

### Entry Point Guidance for Approach Loop

When routing to the Approach Loop for a ripple assessment, the Approach Loop should enter at §ripple-assessment, not §topic-selection. The ripple assessment protocol identifies affected topics and routes to §research-discuss-settle for each. Do not start a fresh topic selection pass.

When routing for a vision change (detected via §vision-regression), the full path applies: Phase Registry → Vision Loop → Vision Expansion → Broad Design Scoping → Approach Loop (starting at §topic-selection with the new Topic Map).

### Review Resumption After Return

On return from the Approach Loop after a ripple assessment: re-derive the section breakdown from the updated settlements and resume section review from the first affected section. Already-approved sections not in the ripple set retain their approval — their Section Approval journal entries remain valid.

Sections that need re-review are those whose content references or depends on the changed approach. Sections unaffected by the ripple retain their approval. When uncertain whether a section is affected, present it for a quick confirmation rather than assuming it's fine.
</core>
</section>

<section id="security-lens">
<core>
## Security Lens

When presenting each section, briefly assess whether it involves security-relevant concerns. This is design-level security surfacing, not a full audit — full security analysis is an implementation concern.

### Criteria

Flag the section if it involves any of:
- **PII handling** — storing, transmitting, or processing personal data (location, contacts, health data, credentials)
- **Authentication/authorization** — who can access what, how identity is verified
- **Data storage** — what's persisted, where, who can read it, how long it's retained
- **External service communication** — API calls, third-party integrations, data leaving the device/server

### When Triggered

Flag the security-relevant aspects explicitly within the section presentation by specifying the security-relevant requirements: what data is retained, who can access it, how long it persists, and what happens when it's no longer needed. Example: "This section stores user location data, which is PII. The design should specify: data retention period, access control (who reads this data), and deletion policy."

Surface the concern as part of the section, not as a separate checklist item.
</core>

<guidance>
### Silent When Clean

If no security concerns apply to a section, do not mention security. The check is silent when clean — no "security: N/A" noise on sections that don't involve sensitive operations.
</guidance>
</section>

<section id="section-approval">
<core>
## Section Approval

When the user approves a section (explicitly or by confirming after detail revisions), write the journal entry before presenting the next section.

<mandatory>Write a Section Approval journal entry for every approved section. The journal's section approval entries are the working set for the Reconciliation phase — missing entries mean sections that reconciliation can't verify.</mandatory>

### What Constitutes Approval

Approval is any clear positive signal from the user: "looks good," "approved," "that works," thumbs up, or similar. If the user's response is ambiguous ("I think that's fine..."), confirm: "Are you happy with this section as-is, or is there something you'd like to adjust?"
</core>

<template follow="format">
## Section Approved: [Section Name]
**Phase:** Phase 6 — Review Loop
**Key decisions reflected:** [References to earlier journal entries — e.g., "Decision: FCM Protocol", "Research: Background Processing"]
**User feedback incorporated:** [Any revisions made during review, or "none — approved as presented"]
**Status:** settled
</template>

<core>
### Partial Approval

If the user approves parts of a section but defers others ("the data model looks good but I'm not sure about the error handling"), treat deferred parts as pending detail feedback. Do not write the Section Approval journal entry until the full section is settled. Rework the deferred portions and re-present them before writing the journal entry.
</core>
</section>

<section id="exit">
<core>
## Exit

When all sections are approved, verify:
- Every section has a Section Approval journal entry
- No feedback is pending or unclassified

Proceed to Phase Registry → Reconciliation.
</core>
</section>

</skill>
