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

The decision journal is your progressive external memory. It captures decisions, research findings, and vision context as the session progresses, ensuring no work is lost to context compaction or session interruption.

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
