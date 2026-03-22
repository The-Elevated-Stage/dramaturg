# Dramaturg — Changes Needed for Arranger Alignment

**Date:** 2026-02-28
**Source:** `stagecraft/docs/working/2026-02-28-arranger-design-action-plan.md`
**Purpose:** Document all changes needed to the Dramaturg skill so that a separate session can execute them.

---

## Context

The Arranger design review (2026-02-28) identified several mismatches between the Dramaturg's output and the Arranger's ingestion expectations. The most critical are the journal path/lifecycle conflicts. The Dramaturg was designed first with a simpler journal lifecycle; the Arranger and Repetiteur were designed later with a richer lifecycle that the Dramaturg was never updated to match.

Key alignment points that are already strong and should NOT be changed:
- Scope boundary (design decisions vs implementation decisions) — well-defined and consistent
- VERIFIED/PARTIAL mechanism — both skills describe it identically
- Design doc structure (Goals opening, Arranger Notes closing, open-ended body) — aligned
- Synthesis quality rules — directly serve the Arranger's needs

---

## Changes Required

### 1. Journal Path — Write to Decisions Directory

**Current behavior:** Dramaturg writes journal to `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md`

**Required change:** Write journal to `docs/plans/designs/decisions/{feature-name}/dramaturg-journal.md`

**Feature-name derivation:** Strip the date prefix and `-design` suffix from the design doc filename.
- Example: design doc is `2026-02-17-background-sync-design.md` → feature-name is `background-sync`
- Journal path becomes: `docs/plans/designs/decisions/background-sync/dramaturg-journal.md`

**Where to change:**
- `SKILL.md` — lines referencing journal path (around lines 32, 63, 339)
- `references/support-phases.md` — any journal path references
- Any references in other reference files

**The `decisions/{feature-name}/` directory creation:** The Dramaturg should create this directory during Phase 7 (finalization). If the directory already exists (from a prior session), the Dramaturg writes alongside existing files.

### 2. Journal Lifecycle — Remove Archival Behavior

**Current behavior:** `support-phases.md:255` contains a `<mandatory>` tag instructing the Dramaturg to archive the journal to `docs/archive/plans/designs/` on finalization.

**Required change:** Remove this archival behavior entirely. The journal persists in the `decisions/{feature-name}/` directory until the Conductor cleans up after implementation completes. This aligns with `repertoire/journal-conventions.md` lifecycle rules ("NOT deleted or archived").

**Where to change:**
- `references/support-phases.md` line 255 — remove or replace the `<mandatory>` archival instruction
- Any other references to journal archival behavior

**Rationale:** The Arranger's subagent ingestion expects to find the journal at the decisions path. The Repetiteur also expects journal persistence for mid-implementation consultations. Archiving breaks both downstream consumers.

### 3. Artifact Count Clarification

**Current state:** `SKILL.md` preamble says two artifacts (design doc + journal). `support-phases.md` final-design-doc `<context>` block lists three artifacts (design doc + journal + Arranger Notes appendix).

**Required change:** Clarify that Arranger Notes is an appendix within the design doc, not a separate artifact. The correct count is two artifacts: design document (which includes the Arranger Notes appendix) and decision journal.

**Where to change:**
- `references/support-phases.md` — the `<context>` block listing three artifacts (around line 269-272)

### 4. Transition Guidance — Add "Next Steps" at Phase 7 Exit

**Current state:** No guidance for the user on what happens after the Dramaturg completes.

**Required change:** Add a "Next Steps" note at the end of Phase 7 finalization. After the design doc is committed, suggest:

```
This design is ready for the Arranger: /arranger docs/plans/designs/{design-doc-name}.md
```

**Where to change:**
- `references/support-phases.md` — at the end of the Phase 7 finalization section

### 5. Scope Protection Clarification

**Current state:** `SKILL.md` scope protection rule says to redirect users pushing for "implementation details (specific file paths, function signatures, variable names, package versions)." The gray zone guidance in `approach-loop.md` provides examples.

**Required change:** Add explicit clarification that technology feasibility research is in-scope for the Dramaturg:
- "Does FCM support this delivery pattern?" = design-level feasibility → in-scope
- "Use a REAL column named lat" = implementation detail → out-of-scope
- "Use SQLite or PostgreSQL?" = architecture choice → in-scope

The distinction is: technology feasibility questions that shape the design are in-scope. Codebase-specific implementation details (file paths, function signatures, exact config values) are out-of-scope.

**Where to change:**
- `references/approach-loop.md` — the gray zone guidance section (around lines 79-87)

### 6. UNRESEARCHED Status — Confirm Documentation

**Current state:** The Dramaturg defines UNRESEARCHED as a third Arranger note status alongside VERIFIED and PARTIAL (`SKILL.md:367`, `research-strategy.md:181`).

**No change needed.** This is already correctly defined in the Dramaturg. The Arranger design is being updated to consume this status. Just confirm the documentation is clear and the status is defined consistently in all reference files.

---

## Validation

All verified (2026-03-01):
- [x] Journal path in SKILL.md and all references points to `docs/plans/designs/decisions/<topic>/dramaturg-journal.md`
- [x] No archival instruction exists anywhere in the skill files (anti-archival note in support-phases.md)
- [x] Feature-name derivation documented in SKILL.md §decision-journal with example
- [x] Artifact count is consistently "two" across all files
- [x] Phase 8 exit includes transition guidance for `/arranger` with path template
- [x] Scope protection gray zone examples include technology feasibility (FCM example)
- [x] UNRESEARCHED status documented in decision-journal.md and support-phases.md (research-strategy.md renamed to research-protocol.md during lean hub refactor)
