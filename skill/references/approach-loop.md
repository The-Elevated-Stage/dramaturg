<skill>

<sections>
- topic-selection
- research-discuss-settle
- tangent-bounding
- vision-change-detection
- foundational-shift
- infeasibility-pivot
- settlement-confirmation
- ripple-assessment
- session-split
- journal-templates
- exit
</sections>

<section id="topic-selection">
<core>
## Topic Selection

This is the core of the Dramaturg skill — where the big architectural decisions happen, informed by active research. A single topic's discussion may last 30-60+ minutes of back-and-forth research and exploration. Thoroughness here is the skill's primary value — never rush or compress this phase.

<mandatory>Read the Topic Map journal entry from the decision journal before beginning topic selection. The Topic Map is the authoritative list of topics to explore — do not rely on conversation memory for what was confirmed during Broad Design Scoping.</mandatory>

Pick topics from the Topic Map (established in Phase 4 and journaled) in dependency order. Foundations first — data model before API design, architecture before UI behavior, core mechanics before edge cases.

### Why Dependency Order

Foundational decisions constrain later ones. If the data model is unsettled, every topic that touches data is building on sand. If the communication protocol is undecided, topics about message handling and error recovery can't be meaningfully explored.

### Selecting the Next Topic

- Identify which unsettled topic has the fewest unresolved dependencies
- If multiple topics are equally foundational, pick the one that the user expressed the strongest opinions about during scoping — their energy signals where the most productive discussion will happen
- Topics map to major design decisions, not document sections. A single topic might eventually span multiple sections, or multiple topics might collapse into one section during Phase 6
</core>
</section>

<section id="research-discuss-settle">
<core>
## Research-Discuss-Settle Loop

This is the core of the skill — the b↔c loop where research and discussion interleave. For each topic, the flow is:

### Step a — Frame the Topic

Briefly establish what's being explored and why it matters to the design. Connect it to the user's vision and any relevant scoping answers. This framing gives the research a target.

### Fast-Path for Trivial Topics

Not every topic needs the full research-discuss-settle cycle. If a topic matches a project pattern verified during Context Grounding (not training data assumptions) and the user expressed clear preference during scoping, you can propose the approach directly: "For [topic], I'd recommend [approach] — it matches your existing [pattern] and you mentioned [preference]. Agree?" On confirmation, proceed directly to §settlement-confirmation.

Reserve the full b↔c loop for topics where research would genuinely add value.

### Step b — Research

Investigate the topic using the research diversion protocol.

<mandatory>Research FIRST, recommend AFTER. Never recommend an approach based solely on training data for substantive technical decisions. Apply the priority chain: compatibility > reliability > efficiency > security > performance. (Restated from SKILL.md §mandatory-rules for emphasis at point of use.)</mandatory>

<guidance>
Research depth should be proportional to architectural impact. Foundational decisions (data model, communication protocol, core patterns) deserve exhaustive multi-source investigation. Peripheral decisions (UI preferences, non-critical defaults) need only a quick feasibility check. Over-researching trivial decisions wastes context budget that foundational topics need more.
</guidance>

When research is needed, follow SKILL.md §research-protocol. Write the journal entry, execute research, return to the recorded step. For simple factual checks, inline Gemini queries suffice. For complex multi-source questions, load `research-protocol.md` via SKILL.md §research-protocol. (This is a reminder of the full protocol in SKILL.md §research-protocol. Follow the full protocol for complex research; this summary covers the common case.)

### Step c — Discuss Findings

Present research results conversationally. Share reasoning, explain tradeoffs, react to what was found. This is a discussion between colleagues, not a report dump.

- Explain what was found and what it means for this topic
- Connect findings to the user's vision and constraints
- Surface tradeoffs explicitly — "Option A gives us X but costs Y"
- Provide explicit recommendations: "I recommend X because Y" — not just "here are three options"

<mandatory>Recommendations must follow the priority chain: compatibility > reliability > efficiency > security > performance. (Restated from SKILL.md §mandatory-rules for emphasis at point of use.)</mandatory>

The user may redirect, suggest alternatives, or raise related concerns. Follow their lead.

### Scope Protection — Design vs. Implementation Boundary

When discussing findings, some details sit in a gray zone between design and implementation. Apply this test: **does the detail constrain future implementation choices unnecessarily?**

- "The data model needs a location field with coordinates" — design. The Arranger needs this to plan the implementation.
- "Use a REAL column named lat" — implementation. The Arranger decides column types and names.
- "Use SQLite or PostgreSQL?" — architecture. This shapes the design.
- "Use WAL mode?" — gray zone, but include it if the design's reliability requirements depend on it.

When uncertain, err toward including it. The Arranger can abstract back up more easily than it can infer missing design intent.

### Steps b↔c Loop

Research and discussion cycle as the topic needs. Discussing findings raises new questions — research those. Research reveals unexpected angles — discuss them. This loop is the heart of the skill's value.

<mandatory>One question at a time for open-ended exploration. Tight, bounded questions (binary choices, parameter confirmations) may be batched when all expect short answers. (Restated from SKILL.md §mandatory-rules for emphasis at point of use.)</mandatory>

Gemini naturally suggests alternatives when analyzing an approach — this is valuable. Briefly assess each against the current context. Deep-dive only on alternatives that look genuinely promising or that challenge an assumption. Not all alternatives need full investigation.

### Step d — Settlement

When the discussion converges on an approach, proceed to §settlement-confirmation.
</core>
</section>

<section id="tangent-bounding">
<core>
## Tangent Bounding

Natural tangents are allowed when topics are genuinely related. Discussing the communication protocol naturally touches on error handling. Exploring the data model may surface questions about caching.

### The Bounding Rule

If a tangent would be its own section in the design document, table it rather than exploring it inline.

- Briefly note it: "Error handling deserves its own topic — I'll add it to the list."
- Return to the current topic.
- Add the tangent to the topic list for later exploration (after the current topic settles).

### Why This Matters

Each individual tangent feels natural and productive. But tangents compound — three "natural" digressions later, the conversation has drifted far from the original topic. The user may lose track of what's been settled vs. what was a tangent thought. Bounding prevents this aggregate drift while preserving the genuinely related discussion that makes Phase 5 fluid.
</core>
</section>

<section id="vision-change-detection">
<core>
## Vision Change Detection

During approach exploration, the user may reveal that their fundamental vision has changed. This is subtler than in the Review Loop (Phase 6) because there are no structured sections to compare against — the change emerges organically from discussion.

### Detection Heuristic

Watch for these patterns:
- **Scope contraction:** "Actually, I just need file sharing, not full collaboration" — the "what" has narrowed fundamentally
- **Goal shift:** "I'm realizing this is really about X, not Y" — the "why" has changed
- **New use case that invalidates settled topics:** "Actually, this needs to work offline too" — the "how-used" has expanded in a way that affects foundations

<mandatory>When a potential vision change is detected: pause forward progress, present the observation to the user, and ask: "This sounds like your vision has shifted from [original baseline] to [new understanding]. Should we revisit the vision?" If confirmed, route to Phase Registry → Vision Loop. The mandatory Vision Expansion (Phase 3) pass-through applies on the return path. If denied, continue in the Approach Loop.</mandatory>

This is a simpler test than the Review Loop's three-question regression check. The Approach Loop detects vision changes through conversational signals, not structured comparison.
</core>
</section>

<section id="foundational-shift">
<core>
## Foundational Shift Protocol

When research or discussion reveals a change that invalidates the basis for multiple settled topics — e.g., client-server → peer-to-peer, synchronous → asynchronous, single-user → multi-user — the individual topic's infeasibility pivot is insufficient. The foundation under multiple settled topics has shifted.

### Protocol

1. **Stop forward progress** — do not continue to the next topic
2. **Present the foundational shift** — explain what changed and what it means: "Research shows [finding]. This changes the foundation that [topic A], [topic B], and [topic C] were built on."
3. **Identify all affected settled topics** — list which previously-settled decisions are built on the now-invalid foundation. Include topics that are NOT affected and why — this reassures the user that the shift is bounded.
4. **User confirms invalidation scope** — the user confirms which topics should be re-explored. They may narrow or expand the scope.
5. **Re-enter topic exploration** for affected topics in dependency order (foundations first, per §topic-selection), using the standard §research-discuss-settle loop and settlement confirmation

<mandatory>Write journal entries for every re-settled topic. Mark each as superseding the original settlement: "Supersedes: [original entry title]"</mandatory>

This protocol is distinct from the ripple assessment (§ripple-assessment), which handles Phase 6 → Phase 5 re-entry. The foundational shift protocol handles mid-Phase-5 invalidation discovered internally.
</core>
</section>

<section id="infeasibility-pivot">
<core>
## Infeasibility Pivot

When research proves the current approach is not feasible, do not try to force it. The skill's value is catching these before they become design flaws.

### Pivot Protocol

1. Present the infeasibility finding immediately: "Research shows [approach] won't work because [finding]."
2. Present alternatives with recommendations: "Here are the alternatives I'd recommend exploring..."
3. Loop back to the start of this topic's exploration with the new alternatives.

No permission is needed to pivot — the finding speaks for itself. This is a natural part of the b↔c loop, not a failure.

### Disproportionate Complexity

If research reveals an approach is feasible but dramatically over-engineered for the use case, mention it conversationally: "This works, but it's significantly more complex than [simpler alternative] for what you need."

This is not a formal gate — just being a good collaborator. Detailed cost/effort analysis is the Arranger's job, but catching obvious over-engineering early prevents wasting a design cycle on an approach the Arranger would later flag as impractical.

### All Alternatives Infeasible

If research proves that ALL reasonable alternatives for a topic are infeasible, the topic may be impossible to achieve as envisioned. Escalate to the user: "None of the approaches I've found can achieve [goal] as described. This may require adjusting the vision — should we revisit what we're trying to accomplish here?" If the user agrees, route to Phase Registry → Vision Loop.

### Journaling Infeasibility

<mandatory>Journal the infeasible approach as a research entry even though it wasn't "settled." The Arranger benefits from knowing what was ruled out and why. Use the research-backed journal template with Status: "ruled-out" instead of "settled."</mandatory>
</core>
</section>

<section id="settlement-confirmation">
<core>
## Settlement Confirmation

When you and the user have converged on an approach, explicitly confirm before moving on.

### Confirmation Format

"So we're going with [approach] for [reason]. Moving on?"

The user either confirms or clarifies. Only on confirmation does the topic count as settled and the conversation move to the next topic.

### Why Explicit Settlement

Without it, topics get left ambiguous. The user might think they were still exploring options while you move on assuming a decision was made. The brief confirmation takes a second but prevents misaligned assumptions that compound across the design.

### Journal Entry on Settlement

<mandatory>Write a journal entry for every settled topic immediately after settlement confirmation. This is the most journal-intensive phase — every settled topic produces an entry. Skipping a journal entry means a settled decision exists only in conversation context, where it will eventually compact away. The journal is your external memory.</mandatory>

<mandatory>After writing the journal entry, confirm to the user: "Journaled — [one-line summary of settled decision]." This confirmation serves as both a checkpoint and a conversation beat that naturally separates topics. The verbal confirmation prevents journal entries from being silently skipped.</mandatory>

<mandatory>Before discussing results of significant autonomous research — journal the findings first, then discuss. This ensures research survives even if the discussion takes an unexpected turn or the session is interrupted.</mandatory>

Journal entry templates are in §journal-templates. Choose the appropriate type (user-discussed or research-backed) based on how the topic was settled.
</core>
</section>

<section id="ripple-assessment">
<core>
## Ripple Assessment — Re-Entry Protocol

This section is executed when the Review Loop detects an approach change during Phase 6 section review and routes back here through the Phase Registry. You arrive with a specific topic whose approach has changed.

### Step 1: Identify the Ripple

Starting from the changed topic, assess which other settled topics are affected by this change. Common ripple patterns:

- A protocol change affects all topics that assumed the old protocol's behavior
- A data model change affects topics that read from or write to the affected entities
- An architecture change (e.g., sync → async) affects topics that assumed the old execution model
- A security or auth change may affect any topic that handles user data

### Step 2: Present Scope to User

List all affected topics explicitly: "The change to [topic] affects these other settled topics: [list with brief explanation of how each is affected]."

Include topics that are *not* affected and why — this reassures the user that the ripple is bounded.

### Step 3: User Confirms Invalidation Scope

The user confirms which topics should be invalidated and re-explored. They may narrow the scope ("X isn't really affected because...") or expand it ("actually Y is also affected because..."). Unaffected sections retain their approval.

### Step 4: Order Affected Topics

Order the invalidated topics by dependency, same as §topic-selection — foundations first. If the original changed topic is also in the list, it goes in dependency order with the rest.

### Step 5: Route to Re-Exploration

Proceed to §research-discuss-settle with the first affected topic. Work through the ripple set one by one using the standard research-discuss-settle loop and settlement confirmation. When the ripple set is exhausted, proceed to §exit to return to the Review Loop.

<mandatory>Write journal entries for every re-settled topic. Mark each entry as superseding the original settlement: reference the original journal entry that is being replaced.</mandatory>

### Cascading Ripples

After re-settling each topic in the ripple set, check: does the new settlement affect other topics not in the original ripple set? If so, expand the ripple set before proceeding to the next topic. Present the expansion to the user: "Re-settling [topic] also affects [new topic] because [reason]. Adding it to the re-exploration list."
</core>
</section>

<section id="session-split">
<core>
## Session Split Evaluation

Before transitioning to Phase 6, evaluate whether the session should split. The Phase 5→6 boundary is the designed natural cut point for session management.

### Evaluation Criteria

- **8+ topics or lengthy discussion:** Recommend the user check context usage and consider starting a fresh session for Phase 6. The decision journal preserves all settled approaches, research findings, and the vision baseline — a new session reads the journal and proceeds directly to section writing.
- **4-6 topics:** Mention the option without pushing. "We've settled all the approaches. Want to continue to section review, or start fresh? The journal has everything we need either way."
- **Small feature:** Continue without comment.

### What the Split Preserves

The journal carries across sessions:
- Vision Baseline (What/Why/How-used)
- Topic Map (confirmed coverage)
- Every settled decision and research finding
- All goal and use-case entries with verbatim user input

A new session reads the journal first and has full context to write detailed sections without any of the Phase 5 discussion needing to be in the conversation window.

<mandatory>For complex features where session splitting is recommended: ensure journal entries are detailed enough for a fresh session to understand the reasoning behind settlements — not just the outcome. If a topic required extensive debate or had closely competing alternatives, document the key differentiators that tipped the decision in the journal's "Alternatives discussed" or "Findings" fields. A post-split Phase 6 session must be able to execute a ripple assessment using only journal entries, without the Phase 5 discussion context.</mandatory>

### What the Split Is NOT

This is not about compressing output or rushing Phase 5 to save context. Thorough Phase 5 analysis is the skill's core value. The split is about starting Phase 6 with a clean context budget so section writing and review get the same thoroughness that approach discussion did.

### Post-Split Continuation

If the session splits: the fresh session reads SKILL.md first, then the decision journal, identifies the current phase as Phase 6 (Review Loop), and enters the Phase Registry to pick up from there. The journal carries all necessary context.
</core>
</section>

<section id="journal-templates">
<core>
## Journal Entry Templates

Two entry types for settled topics. Choose based on how the topic was settled.

### User-Discussed Entries

For topics settled through conversation — the user's reasoning and preferences drove the decision.

<mandatory>User input is captured verbatim in the "User verbatim" field — never paraphrased, never summarized. Downstream skills need the user's actual language to understand intent.</mandatory>

<guidance>For long user statements, capture the design-relevant sentence or sentences verbatim — not a full transcription of a multi-paragraph response. The verbatim field should contain the user's actual words that express the decision, goal, or use case, not their entire conversational response.</guidance>

<mandatory>Category tags are not optional metadata — they determine how downstream skills treat the entry. Goals and use-cases are inviolable for downstream skills — they represent user intent that survives implementation replanning. Decisions can be reworked.
- **goal** — What the user wants the feature to accomplish. Be verbose. Capture the full scope including specific behaviors, types, and capabilities.
- **use-case** — How the user plans to use the feature. Reveals the user's mental model of how it works in practice.
- **decision** — A technical or design choice. Default category when neither goal nor use-case applies.
If a user statement contains both a goal and a technical decision, journal them as separate entries.</mandatory>
</core>

<template follow="format">
## Decision: [Topic]
**Phase:** Phase 5 — Approach Loop
**Category:** [goal | use-case | decision]
**Decided:** [What was decided]
**User verbatim:** [The user's exact words — not paraphrased]
**User context:** [Additional reasoning, constraints, or motivation the user expressed]
**Alternatives discussed:** [What else was considered, why rejected]
**Status:** settled
**Supersedes:** [reference to original entry if this re-settles a previously settled topic, or "—"]
</template>

<core>
### Research-Backed Entries

For topics validated through Gemini, web search, or subagent investigation. The Arranger note enables cross-skill optimization.
</core>

<template follow="format">
## Research: [Topic]
**Phase:** Phase 5 — Approach Loop
**Question:** [What was being validated]
**Tools used:** [gemini-search / gemini-query / brave-search / subagent / WebFetch]
**Findings:** [What was discovered — specific enough for the Arranger to assess re-verification need]
**Decision:** [What was decided based on findings]
**Arranger note:** [VERIFIED — Arranger can skip re-audit] or [PARTIAL — Arranger should verify: (specific gap)]
**Status:** settled
**Supersedes:** [reference to original entry if this re-settles a previously settled topic, or "—"]
</template>
</section>

<section id="exit">
<core>
## Exit

<mandatory>Phase 5 is complete when ALL of the following are true:
1. Every topic from the Topic Map has been explored and settled (Read the Topic Map journal entry and verify each listed topic has a corresponding Decision or Research journal entry)
2. Every settled topic has a journal entry (verify by reviewing the journal)
3. The session split evaluation (§session-split) has been considered

When all three are true, proceed to Phase Registry → Review Loop.</mandatory>
</core>
</section>

</skill>
