<skill>

<sections>
- context-grounding
- broad-design-scoping
- reconciliation
- final-design-doc
</sections>

<section id="context-grounding">
<core>
## Context Grounding

Launch Explore subagents to understand the project before the design discussion begins. The goal is "what exists today" — enough to have an informed conversation, not an exhaustive audit.

### What to Explore

- **Architecture and scope** — what frameworks, patterns, and conventions does the project use?
- **Relevant existing features** — what already exists that the new design might interact with, extend, or depend on?
- **Integration points** — what APIs, services, data stores, or modules would a new feature need to connect to?

### Subagent Strategy

Use Explore subagents to keep the main conversation context clean. Frame subagent prompts around specific questions:
- "What is the project's architecture? What frameworks and patterns does it use?"
- "What existing features relate to [topic the user mentioned]?"
- "How is [relevant subsystem] currently implemented?"

One or two focused subagents are sufficient. Do not launch a broad codebase analysis — that wastes context budget that Phase 5 needs more.
</core>

<guidance>
### Judging "Enough Context"

Exit when you can describe:
1. The project's tech stack and primary patterns
2. Existing features related to the user's topic
3. Key integration points the design might touch

Err toward moving on. If the Vision Loop reveals unexplored project context (e.g., "this needs to integrate with the notification system" and that area hasn't been explored), the Phase Registry routes back here. Starting the vision discussion early is better than over-exploring context that may turn out to be irrelevant.

### Context Pressure Awareness

If context pressure builds during Phase 1, Phase 2, or Phase 3, the Dramaturg may suggest capturing progress: "We've covered a lot of ground — let me write the Vision Baseline now and we can continue refining in the next phase." Early phases (Context Grounding, Vision Loop, Vision Expansion) should not exhaust context budget that Phase 5 (Approach Loop) needs more.
</guidance>

<core>
### Exit

Proceed to Phase Registry → Vision Loop.
</core>
</section>

<section id="broad-design-scoping">
<mandatory>
### Question Constraints

- Maximum 6 questions total across all areas
- Maximum 2 questions per area (backend, frontend, data model, etc.) unless the feature is particularly heavy in that area
- Open-ended questions only — designed to make the user talk, not answer yes/no or pick from a list
- One question at a time — do not batch open-ended scoping questions

These limits force high-quality questions. When limited to 2 backend questions, the Dramaturg picks the ones that most shape the design. The cap also respects the user's time — scoping is not the interview.
</mandatory>

<core>
### Question Framing

Design questions to draw out the user's mental model and priorities. The question's form determines the answer's value:

- **"How do you see the frontend and backend communicating for this feature?"** gets assumptions, preferences, and constraints that inform the entire design.
- **"Should the backend use REST or GraphQL?"** gets a one-word answer with no context.

The first form is always better for scoping. Scoping questions establish the *territory* — what areas need exploration, what the user's instincts are, where the complexity lives. They do not settle approaches (that's Phase 5's job with research backing).

### Topic Map Construction

After all scoping questions are answered:

1. List the topics to explore in Phase 5, derived from the scoping answers. Topics map to major design decisions, not document sections.
2. Ask the user to confirm the list is complete: "These are the areas I think we need to explore in detail. Is there anything missing?"
3. The confirmed list becomes the Topic Map journal entry.

<mandatory>Write the Topic Map journal entry before proceeding to Phase 5. The topic map is both a coverage check and a recoverable checkpoint if the session is interrupted.</mandatory>
</core>

<template follow="format">
## Topic Map
**Phase:** Phase 4 — Broad Design Scoping
**Areas to explore in Phase 5:**
- [Topic 1] — [why it needs exploration]
- [Topic 2] — [why it needs exploration]
- [Topic 3] — [why it needs exploration]
**User confirmed coverage:** [yes/no — must be yes before proceeding]
**Status:** confirmed
</template>

<guidance>
If the user identifies missing topics when confirming coverage, add them to the Topic Map and re-confirm. The list is complete only when the user says it is.
</guidance>

<core>
### Complexity Threshold

If the Topic Map exceeds 10 topics spanning 3 or more distinct systems, present the option of decomposing into multiple focused design sessions: "This feature touches [N] topics across [M] systems. Would you prefer to split this into separate design sessions with clear boundaries, or proceed as one large design?"

The user decides. Some complex features benefit from a single coherent design session; others are better served by focused sessions with well-defined interfaces between them.

### Exit

Proceed to Phase Registry → Approach Loop.
</core>
</section>

<section id="reconciliation">
<core>
## Reconciliation Workflow

Follow these steps in order. This is the prescribed workflow referenced from the Phase Registry.

### Step 1: Gather Approved Sections

Read the decision journal's Section Approval entries for a compact view of all approved sections. List them with their key decisions. This is the working set for reconciliation — avoid re-reading full section text unless a conflict needs detailed inspection.

If no Section Approval journal entries exist (e.g., journal was not maintained during Review Loop), reconstruct the approved sections from conversation context and present the reconstruction to the user for verification before proceeding with reconciliation.

### Step 2: Cross-Section Consistency Check

For each section, check: did any later section's revisions, research findings, or settled approaches affect this section's assumptions? Common inconsistency patterns:

- Terminology drift between sections (one says "device," another says "client")
- A data format settled in a later section contradicts an earlier section's assumption
- An approach change during review altered the dependencies between sections
- A research finding in one topic invalidated an assumption in another

### Step 3: Classify Each Inconsistency

Every inconsistency falls into exactly one severity level:

**Cosmetic** — Surface-level differences that don't affect meaning or architecture.
- Terminology drift ("device" vs "client" for the same concept)
- Formatting differences between sections
- Slightly different phrasing for the same constraint
- **Action:** Fix silently. Update the section without user re-approval. Mention the fix in passing: "I standardized the terminology across sections."

**Clarification** — A section references something that a later section made more specific, but the architecture doesn't change.
- "Stores location" in Section A, later settled on "GeoJSON point" in Section D
- "Communicates with the backend" in one section, "via FCM push notifications" settled in another
- **Action:** Update the earlier section to match the later settlement. Present the change as an informational note: "I updated Section A to say GeoJSON point, matching what we settled in Section D." No re-approval needed.

**Substantive** — A section's approach or logic conflicts with a later decision. The architecture would change.
- Section A assumed polling but Section C switched to WebSockets
- Section B's error handling relies on synchronous responses but Section E introduced an async pattern
- A research finding in Phase 5 invalidated an assumption that an earlier section was built on
- **Action:** Present the conflict and the proposed revision. Require user re-approval of the affected section.

### Step 4: Handle Substantive Conflicts

For each substantive inconsistency:
1. Present both sides clearly: "Section A assumes X, but Section C decided Y. These conflict because..."
2. Propose a resolution
3. Get user approval of the revised section

### Circuit Breaker

<mandatory>If reconciliation routes to Review Loop more than twice (i.e., the same design keeps generating substantive inconsistencies after re-approval), the underlying design has a structural tension that reconciliation-level fixes cannot resolve. Present the tension to the user explicitly: "These sections keep conflicting because [root cause]. Rather than continuing to patch, should we revisit [the underlying design question]?" Route to the Approach Loop for the conflicting topic if the user agrees.</mandatory>
</core>

<mandatory>
Substantive inconsistencies require user re-approval. Do not silently resolve them.

If a section needs rework beyond reconciliation-level fixes — the conflict reveals a deeper design issue that can't be resolved by updating one section — route to Phase Registry → Review Loop for that section.
</mandatory>

<core>
### Step 5: Implementation Readiness Test

Before moving to Final Design Doc, evaluate completeness separately from consistency:

**The test:** Could the Arranger take this design and produce an executable implementation plan without needing to make design-level decisions?

If **yes** — the design is ready. Tell the user: "The design is ready when you are." Proceed to Phase 8 when the user confirms.

If **no** — identify what's unresolved. Common gaps:
- A vague data model ("stores user data" without specifying what data or how)
- Unexplored error handling ("what happens when the connection drops?")
- Missing edge cases that would force the Arranger to make design choices
- An interaction pattern described at high level but not worked through

<mandatory>Consistency does not imply completeness. A perfectly consistent design with a vague data model or unexplored error handling is not ready. Keep probing — ask more questions, research open areas, revisit underspecified sections — until the design is genuinely ready to hand off.</mandatory>

### Exit

Proceed to Phase Registry → Final Design Doc.
</core>
</section>

<section id="final-design-doc">
<core>
## Final Design Doc

Compile all reconciled sections into a single design document at `docs/plans/designs/YYYY-MM-DD-<topic>-design.md`. This is the design doc's permanent location — it is not moved after finalization.

### Goals Section (Opening — Required)

The design doc must open with a verbose, narrative statement of the user's goals. This section is the anchor that every downstream skill relies on.

**Writing guidance:**
- Rich with context — not "location-aware app" but "the app should be location-aware, with notifications that respect the user's physical locations..."
- Independent of technical decisions — goals describe *what the user wants to achieve*, not *how* the design achieves it
- Every goal the user stated during the session belongs here, including specific behaviors, types, and capabilities they mentioned
- Self-contained — readable without the rest of the document. If the Arranger needs to fundamentally redesign the implementation approach, the goals section tells it exactly what the user wants
- The journal captures individual goal entries with verbatim user words. The goals section synthesizes them into a coherent narrative picture. Both are needed — they serve different purposes.

### Document Body (Open-Ended)

No rigid template. Whatever best captures the design vision developed with the user. The document is more akin to a structured prompt for the Arranger than a formal specification.

<guidance>
Most design decisions appear naturally as part of the narrative. The decision journal handles the structured decision trail separately — the design doc doesn't need to duplicate it.
</guidance>

### Synthesis Quality

The design doc must convert messy conversational input into structured output while preserving fidelity. When compiling:

- **Distinguish confirmed decisions from design-level inferences.** Decisions explicitly settled with the user are stated as decisions. Requirements the Dramaturg derived from conversation that the user didn't explicitly state are noted as inferences: "Based on the user's goal of [X], this implies [Y]."
- **Note open questions explicitly.** If something couldn't be fully resolved during the design phase, say so. Don't silently paper over gaps.
- **Capture implied requirements separately from stated requirements.** "The user said they want DAG visualization" is stated. "DAG visualization implies a graph data structure" is implied. Both are valuable, but the Arranger needs to know which is which.
- **Capture intent, not casual specifics.** When the user explored multiple options casually ("maybe a chain icon?"), capture the intent ("indicate blocked status visually") rather than the casual exploration.
- **Handle user self-corrections.** When the user changed their mind during conversation, the final position is what matters. The journal tracks the evolution; the design doc reflects the outcome.

### Arranger Notes Appendix (Closing — Required)
</core>

<template follow="format">
## Arranger Notes

### New Protocols / Unimplemented Patterns
- [Protocol/pattern not currently in the codebase that the design relies on]
- [Reference the journal entry where this was researched]

### Open Questions
- [Anything that couldn't be fully resolved during the design phase]
- [Project-specific details that need investigation against the actual codebase]

### Key Design Decisions
- [Brief pointers to the most architecturally significant decisions]
- [Not a full list — the journal has the complete trail]
</template>

<core>
### Journal Archival

<mandatory>Archive the decision journal from `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md` to `docs/archive/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md` when the design doc is finalized. The journal is a working artifact, not a deliverable — but it persists in the archive for the Arranger to consume. The design document stays in `docs/plans/designs/`.</mandatory>

<mandatory>Verify journal completeness before archival — confirm every checkpoint trigger produced an entry. The journal must be complete when it reaches the Arranger. Missing entries mean lost decisions.</mandatory>
</core>

<guidance>
### Scope Protection

The design doc captures architecture-level decisions. When writing, resist the pull toward implementation specifics — file paths, function signatures, variable names, package versions. These details are the Arranger's concern. If a section drifts into implementation territory, pull it back to the design level: describe *what* the component does and *why*, not *how* it's coded.
</guidance>

<context>
### What the Arranger Receives

Three artifacts from the Dramaturg:
1. **Design document** — goals section + vision, architecture, creative narrative
2. **Decision journal** — structured decision trail with goal/use-case/decision categorization, verbatim user input, VERIFIED/PARTIAL research flags
3. **Arranger Notes appendix** — quick-orientation flags for the Arranger's feasibility audit
</context>
</section>

</skill>
