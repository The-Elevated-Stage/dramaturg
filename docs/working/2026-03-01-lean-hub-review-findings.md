# Lean Hub Refactor — Post-Review Findings

**Date:** 2026-03-01
**Status:** Pending triage
**Context:** Findings from 5 parallel reviews after the SKILL.md lean hub refactor (commit `0de1c69`).

## Fixed

- `support-phases.md` line 166: "Route to the Approach Loop" → "Route to Phase Registry → Approach Loop" (routing phrasing consistency)

## Lazy Student Risk

### Vision Loop (Phase 2) — Medium risk
The hub provides the full transition gate (What/Why/How-used) inline, which is enough structure that a reader could attempt the phase without loading `vision-loop.md`. They would miss: question strategy, research-during-vision protocol, vision baseline journal template, loop-back conditions.

**Mitigation option:** Add one sentence naming a specific non-obvious procedure, e.g., "The reference defines a questioning strategy that avoids leading questions and a protocol for research triggers during vision gathering."

### Review Loop (Phase 6) — Medium risk
The hub provides "present one section at a time" + routing mandatories, which is close to actionable without loading `review-loop.md`. They would miss: feedback classification framework, vision regression test, security lens, journal template.

**Mitigation option:** Add one sentence, e.g., "The reference defines a vision regression test applied after each section and a security lens analysis step."

### Broad Design Scoping annotation leaks constraints
SKILL.md line 155: reference annotation says "constraints (max 6 questions, max 2 per area)" — this gives specific implementation details that partially defeat the lean hub purpose. Consider shortening to "Question framing constraints, topic map construction."

## Structural

### `support-phases.md` overload
Covers 4 phases (Context Grounding, Broad Design Scoping, Reconciliation, Final Design Doc) in one 277-line file. SKILL.md reference tags say `load="required"` then "Read §X only" — reader loads 277 lines but needs ~60. Options:
1. Split into individual files (one per phase)
2. Add an internal loading guide comment block (like `research-protocol.md` has)

### Conversation Style inline weight
46 lines of behavioral rules with no reference delegation. Defensible as cross-phase concern (applies in every phase), but noticeably heavier than anything in Repetiteur's hub. First extraction candidate if context pressure grows. The mandatory rules (Scope Protection, Output Length) should stay at hub level; softer guidance (Conversational Tone, Fluid Transitions, Disproportionate Complexity, Scope Awareness) could move to a reference.

### `review-loop.md` sections index format
Uses HTML comment category headers in its `<sections>` index — unique convention not used by any other reference file. Either adopt across files or remove for consistency.

### `vision-loop.md` missing `<section id="exit">`
Exit routing is embedded within `gate-verification` section (lines 129-132) instead of having a dedicated section. All other loop files (`vision-expansion.md`, `approach-loop.md`, `review-loop.md`) have a dedicated `<section id="exit">`.

### No examples section
The Repetiteur has an `examples` section pointing to three example files. The Dramaturg has none. Example design documents and decision journals would reduce ambiguity about expected output format.

### Approach Loop registry entry missing session split hint
The Phase Registry entry for Approach Loop (SKILL.md lines 160-172) doesn't mention session split evaluation, which is a required step before exiting Phase 5. Documented in `approach-loop.md` §session-split and §exit but not hinted in the hub.

## Wording

### Person inconsistency
Dramaturg uses third person ("The Dramaturg is...") throughout. Repetiteur uses second person ("You are the Repetiteur") in its preamble. Second person is more direct and creates stronger identification. Consider switching to "You are the Dramaturg" for consistency across the orchestra.

### "co-visionary" (SKILL.md line 45)
Unusual word choice. Less immediately clear than Repetiteur's "autonomous mid-implementation re-planner." Consider "design collaborator" or "research-augmented design partner."

### Double-negative framing (SKILL.md lines 286-292)
Lines 286-288 and 290-292 both define conversational behaviors then immediately say "This is not a formal gate" / "This is informational, not a gate." The Repetiteur avoids this pattern. Rephrase positively: "Mention it conversationally and continue."

## Authority Tags

### Session Bootstrap authority level
SKILL.md lines 296-308: reading the journal on resume is tagged as `<guidance>` but is operationally critical — a corrupted resume could send the session into the wrong phase. Consider elevating the first two steps (read journal, identify current phase) to `<mandatory>`.

### Mandatory block dilution (SKILL.md line 37)
"Vision Loop gate and Vision Expansion are non-skippable. Other phases can be abbreviated if the user has done the thinking, but never skipped entirely." The second sentence is permissive language inside a `<mandatory>` block — it describes flexibility, not a constraint.

### Context Pressure Awareness (support-phases.md lines 43-45)
Tagged as `<guidance>` but exhausting context in Phase 1-3 would be a session failure. Could arguably be `<mandatory>`.

## Minor / Informational

- `approach-loop.md` line 65: phrasing "load `research-protocol.md` per SKILL.md §research-protocol" could read more clearly as "via" instead of "per"
- Stale `research-strategy.md` references exist in `docs/working/` files (historical/design docs, not operational)
- Journal archival path (`support-phases.md` lines 255-257) assumes `docs/archive/plans/designs/` exists in target project
- `approach-loop.md` lines 49-53: fast-path "established project pattern" is ambiguous — should clarify it means patterns verified during Context Grounding, not training data knowledge
