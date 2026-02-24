# Vision Expansion Phase — Design Document

**Date:** 2026-02-24
**Status:** Approved
**Skill:** Dramaturg
**Type:** New phase addition

---

## Problem Statement

The Dramaturg currently operates as a researcher/validator — it draws out what the user already knows (elicitation) but doesn't contribute what they don't know but should (enrichment). For an amateur developer who may not think of interaction patterns, UX conventions, edge cases, or cross-feature implications, this gap means the design document only captures what the user happened to think of, not what the feature actually needs.

The review's strongest cross-reviewer finding (C11, rated CRITICAL by three independent probers) identified this as the single most impactful design question: "The skill draws out what the user knows but doesn't add what they don't know but should."

## Design

### Phase Registry Changes

Vision Expansion becomes Phase 3, inserted between Vision Loop and Broad Design Scoping. All subsequent phases shift up by one:

| Phase | Name | Role |
|-------|------|------|
| 1 | Context Grounding | Project exploration |
| 2 | Vision Loop | Elicitation — What/Why/How-used from user's words |
| **3** | **Vision Expansion** | **Enrichment — proactive capability suggestions** |
| 4 | Broad Design Scoping | High-level topic mapping |
| 5 | Approach Loop | Research-backed topic exploration |
| 6 | Review Loop | Per-section design review |
| 7 | Reconciliation | Cross-section consistency |
| 8 | Final Design Doc | Compile and output |

**Phase Registry entry:**

- **Role:** Proactively enrich the user's vision with capabilities, interactions, edge cases, and cross-feature implications they haven't considered. Shifts from elicitation (Phase 2) to enrichment — the Dramaturg contributes design ideas, not just captures them.
- **Mechanism:** AskUserQuestion for yes/no decisions with context. Brief discussion allowed but bounded — when discussion drifts into architectural detail, table it for Phase 5.
- **Transition:** Exit when all enrichment passes complete and the user confirms. Re-verify the Vision Baseline (What/Why/How-used) against expansion results before proceeding — if enrichment materially changed the vision, update the baseline and re-confirm.

**Loop-back routing rule:** When Phase 6 routes back to Vision Loop, the subsequent pass through Vision Expansion is mandatory — the vision shift may have created new enrichment opportunities.

This also addresses review finding I11 (phase numbering never formally defined) — the renumbered registry makes numbers explicit.

---

### Enrichment Sources (Ordered)

Three sources, executed in this order:

1. **Dramaturg ideation** — The Dramaturg generates suggestions from its own understanding: interaction patterns and UX conventions for the feature type, cross-feature implications from Context Grounding (what existing features does this touch?), edge cases and boundary conditions, and common "forgot to think about" items for the domain.

2. **Gemini creative ideation** — A structured prompt to Gemini containing: the user's confirmed vision (What/Why/How-used), the Dramaturg's own ideas (so Gemini builds on them rather than duplicating), broad tech stack only (e.g., "Flutter/Dart frontend, Node.js/Express backend, SQLite database"), and a request for interaction ideas, UX patterns, and "what would delight a user" suggestions.

3. **Merge and curate** — Deduplicate across both sources. Tier into three groups:
   - **Strong recommendations** — the Dramaturg is confident these add value
   - **Worth asking** — plausible but not certain they fit
   - **Remainder** — creative ideas that might spark interest

---

### Three-Pass Presentation

**Pass 1:** Strong recommendations via AskUserQuestion. Each question is yes/no with enough context for the user to understand the recommendation. One question at a time.

**Pass 2: Re-review and promote.** After Pass 1, the Dramaturg re-evaluates the "worth asking" tier against the user's Pass 1 answers. Ideas that gained relevance from the user's enthusiasm pattern get promoted and presented as another AskUserQuestion round.

**Pass 3: Bulleted remainder.** Present the remaining ideas as a grouped list — "Any of these interest you?" The user can pick zero or many. Any that spark interest get brief discussion.

---

### Discussion Boundary

Mini-discussions are natural and welcome — a "yes" to drag-to-reschedule might prompt the user to say "yeah, and it should snap to 15-minute increments." That's valuable vision context. But when discussion starts reaching for architectural decisions, the Dramaturg flags it: "This is getting into Phase 5 territory — keep exploring now or table it for later?" This is a gentle redirect, not a hard gate.

---

### Vision Baseline Re-verification

After all passes and discussions complete, the Dramaturg re-checks the What/Why/How-used against what was added. If enrichment materially changed the vision (which it often will), the Dramaturg updates and re-presents the Vision Baseline summary for user confirmation. This updated baseline is what gets journaled and what Phase 6 tests against for regression.

---

### Exit Gate

Simple confirmation: "That covers the expansion — ready to move on to scoping?" No strict verification beyond the Vision Baseline re-check.

---

### AskUserQuestion Usage Pattern

**When to use:** When proposing a capability or interaction the user hasn't discussed yet and the question has a natural yes/no answer. The question must include enough context for an informed decision — not just "Should the planner support drag-to-reschedule?" but "Should the planner support drag-to-reschedule? This would let you grab a task and move it to a different time slot to quickly adjust your schedule."

Each question should contain:
- **The suggestion** — what the Dramaturg is proposing
- **Brief context** — why it's relevant and what it looks like in practice
- **The Dramaturg's recommendation** — lean yes or lean no

**When NOT to use:**
- Open-ended design questions that need discussion — use normal conversation
- Questions where yes/no would be misleading — use normal conversation
- Follow-up clarification when a "yes" needs more detail — switch to conversation for that thread
- Anything outside Vision Expansion — the rest of the skill uses normal conversational flow

**Transition to discussion:** When a yes/no naturally opens brief discussion, switch seamlessly to conversation. When the thread resolves or gets tabled, return to AskUserQuestion. No "switching modes" announcements.

---

### Gemini Prompt Design

**Include:**
1. Feature context — the user's What/Why/How-used in natural language
2. Broad tech stack — language/framework level only (e.g., "Flutter/Dart mobile and web app, Node.js/Express backend, SQLite database"). Not library versions, not architectural patterns, not schema details.
3. Ideas already generated — the Dramaturg's own suggestions, framed as "here's what we've already thought of"
4. The ask — interaction ideas, UX patterns, edge cases, cross-feature opportunities, "what would delight a user" suggestions. Ask Gemini to organize by theme/category.

**Exclude:**
- Implementation details or codebase structure
- User preferences or constraints beyond the vision
- Full project feature set (only mention existing features directly related to the new feature)

**Handling output:** Gemini will return 15-30+ suggestions. The Dramaturg curates by removing contradictions to user's vision, implementation details disguised as features, and duplicates of its own ideas. Remaining items are tiered by relevance to the user's stated goals and use cases.

---

### Loop-back Integration

When Phase 6 (Review Loop) detects a vision change and routes back to Phase 2 (Vision Loop):
- The user re-establishes their vision through normal elicitation
- After the Vision Loop gate passes, Vision Expansion runs again (mandatory)
- The Dramaturg's ideation and Gemini prompt incorporate the changed vision
- Previously accepted enrichments that conflict with the new vision are dropped; compatible ones carry forward without re-asking

### Relationship to Broad Design Scoping (Phase 4)

Vision Expansion feeds directly into Broad Design Scoping. Confirmed enrichments become inputs to scoping questions and Topic Map construction. The user who accepted drag-to-reschedule, inline task creation, and day/week toggle gives Phase 4 a much richer starting point than "planner view."

Broad Design Scoping doesn't change structurally — it still frames the territory with max 6 questions. But the territory is now more complete.

### Tabled Items

Items tabled during Vision Expansion (discussions that drifted toward Phase 5 territory) should be checked against the Topic Map when Phase 5 begins topic selection. They may need to become topics or fold into existing ones.

---

### Journal Entry

A new entry type capturing enrichment results. Written once after the exit gate, before transitioning to Broad Design Scoping.

```
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
```

**Why capture rejections:** The Arranger benefits from knowing what was explicitly considered and rejected. Prevents the downstream pipeline from re-litigating settled decisions.

**Why capture tabled items:** Tabled items are discussions intentionally deferred to Phase 5. They should be checked against the Topic Map during Approach Loop topic selection.

---

## Review Findings Addressed

- **C11 (Critical)** — "Elicitation without enrichment." Directly answered by Vision Expansion's systematic enrichment with Gemini-powered ideation and Dramaturg pattern recognition.
- **I3 (Important)** — "No fast-path for complete visions." Users with comprehensive visions get mostly "yes, already planned" in Pass 1, and the phase completes quickly.
- **I11 (Important)** — "Phase numbering not formally defined." Renumbered registry makes numbers explicit.
- **I23 (Important)** — "Effort mismatch detection absent." Vision Expansion reveals true scope before Phase 5 rather than late discovery.
- **S3 (Suggestion)** — "'Co-visionary' framing more explicit." Vision Expansion makes the Dramaturg explicitly a co-visionary.
- **S18 (Suggestion)** — "Category-triggered checklists." Gemini prompt and Dramaturg ideation include category-aware triggers.
