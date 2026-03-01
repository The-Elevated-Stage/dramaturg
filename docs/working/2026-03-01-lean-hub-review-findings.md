# Lean Hub Refactor — Post-Review Findings

**Date:** 2026-03-01
**Status:** Resolved
**Context:** Findings from 5 parallel reviews after the SKILL.md lean hub refactor (commit `0de1c69`).

## Fixed

### First pass (commit `b22cb8d`)
- `support-phases.md` line 166: "Route to the Approach Loop" → "Route to Phase Registry → Approach Loop" (routing phrasing consistency)

### Second pass — triage fixes
- Vision Loop reference annotation: added "including techniques for avoiding leading questions" and research-during-vision protocol to reduce lazy student risk
- Review Loop reference annotation: added "framework" to feedback classification and "(applied after each section)" to vision regression test to reduce lazy student risk
- Broad Design Scoping annotation: shortened from "constraints (max 6 questions, max 2 per area)" to "Question framing constraints, topic map construction"
- Approach Loop registry entry: added "(required before exiting Phase 5)" to session split evaluation mention
- Double-negative framing: replaced "This is not a formal gate" / "This is informational, not a gate" with "Mention it and continue"
- Session Bootstrap: elevated first two steps (read journal, identify current phase) from `<guidance>` to `<mandatory>`, section wrapper from `<guidance>` to `<core>`
- Mandatory block dilution: replaced permissive "can be abbreviated if the user has done the thinking, but never skipped entirely" with constraint "No phase may be skipped entirely — every phase must be entered and its exit conditions met"
- `approach-loop.md` line 65: "per" → "via"
- `approach-loop.md` fast-path: clarified "established project pattern" means patterns verified during Context Grounding, not training data assumptions

### Third pass — deferred items resolved
- **Person switch:** All files switched from third person ("The Dramaturg is...") to second person ("You are the Dramaturg..."). Kept third person only in proper noun skill name references and `<context>` tags describing the role to external consumers (Arranger).
- **"co-visionary" → "design collaborator":** SKILL.md preamble updated.
- **`support-phases.md` loading guide:** Added loading guide comment block matching `research-protocol.md` pattern. Each section marked as self-contained.
- **Context Pressure Awareness → mandatory:** Elevated from `<guidance>` to `<mandatory>` in `support-phases.md`.
- **Conversation Style extraction:** Moved softer guidance (collaborative tone, fluid transitions, disproportionate complexity, scope awareness) to new `references/conversation-style.md`. SKILL.md retains mandatories (one-question, scope protection, output length) and open-ended questions preference, with reference pointer to the extracted content. SKILL.md reduced from 311 → 296 lines.
- **`review-loop.md` sections index:** Removed HTML comment category headers for consistency with all other reference files.
- **`vision-loop.md` exit section:** Extracted exit routing from `gate-verification` into dedicated `<section id="exit">`, matching the pattern used by all other loop files.

### Fourth pass — remaining items
- **Examples section:** Created `examples/example-design-document.md` (goals section, document body, Arranger Notes) and `examples/example-decision-journal.md` (all entry types: Vision Baseline, Vision Expansion, Topic Map, decision with goal/use-case/decision categories, research with VERIFIED/PARTIAL, tension, section approval). Added `examples` section to SKILL.md with `load="optional"` reference pointers.
- **Journal archival path:** Added "Create the archive directory if it doesn't exist" to the archival mandatory in `support-phases.md`.
- **Stale `research-strategy.md` refs in docs/:** Confirmed all occurrences are in historical documents (`docs/plans/`, `docs/archive/`, `docs/working/`) that correctly reference the old filename in their historical context. No operational files affected. No action needed.
