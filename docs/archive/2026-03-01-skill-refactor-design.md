# Dramaturg SKILL.md Lean Hub Refactor Design

**Date:** 2026-03-01
**Status:** Approved

## Goal

Refactor Dramaturg's SKILL.md from a heavy hub (470 lines) to a lean hub (~280-310 lines) with prompt-injection routing, matching the Repetiteur's approach where SKILL.md acts as a router and prompt injector rather than a protocol encyclopedia. The Phase Registry's existing strict routing discipline (all transitions through the registry, zero exceptions) is preserved and improved — the leaner registry becomes a better prompt injection because the signal-to-noise ratio improves on re-read.

## Design Decisions

- **Routing model:** Keep Dramaturg's strict registry routing (every transition through SKILL.md Phase Registry). Do not adopt Repetiteur's split model where protocol references can cross-reference each other directly.
- **Research protocol home:** Merge trigger rules and diversion flow from SKILL.md into `research-strategy.md`, renamed to `research-protocol.md`. SKILL.md keeps a slim routing section.
- **Decision journal home:** New `references/decision-journal.md` owns entry types, field lists, templates, and checkpoint triggers. SKILL.md keeps a slim routing section.
- **Phase Registry density:** Medium — keep mandatory transition gates and journal checkpoint triggers visible per phase. Move framing paragraphs, loop-back conditions, and detailed constraints to reference files.
- **Conversation style:** Stays in SKILL.md. Cross-cutting behavioral guidance that applies to every phase, no natural reference home.
- **Session bootstrap:** Stays in SKILL.md. Short guidance-level content.

## SKILL.md Section Structure

```
<sections>
- mandatory-rules          # kept as-is
- preamble                 # kept as-is (role, pipeline, scope, artifacts)
- phase-registry           # lean: routing table + per-phase (role + gate + checkpoint + reference)
- research-protocol        # slim routing section (~8 lines)
- decision-journal         # slim routing section (~8 lines)
- conversation-style       # kept as-is (~45 lines)
- session-bootstrap        # kept as-is (~12 lines)
</sections>
```

Estimated SKILL.md size: ~280-310 lines (down from 470).

## Phase Registry Transformation

### Per-Phase Pattern

Each phase entry follows this pattern (~5-8 lines):

```markdown
### Phase Name

**Role:** One sentence.
<mandatory>Transition gate (if any).</mandatory>
**Journal checkpoint:** What to write on exit.

<reference path="references/file.md" load="required">
What to read in the reference.
</reference>
```

### What Stays in the Registry

- **Mandatory transition gates** — the hard constraints that gate phase exit (e.g., Vision Loop's What/Why/How-used requirement)
- **Journal checkpoint triggers** — one line per phase ("Write the Vision Baseline entry on exit")
- **Routing table** — the existing table mapping phase numbers to names/roles/references, positioned above per-phase entries

### What Moves to Reference Files

- Framing paragraphs (brainstorming-style dialogue descriptions, thoroughness notes)
- Loop-back conditions (already described in calling references; consolidated in receiving references)
- Duplicate mandatories (one-question-at-a-time, one-section-at-a-time — already enforced in reference files)
- Detailed constraint descriptions (vision-change detection specifics, ripple assessment entry points)

## Research Protocol Migration

### SKILL.md Section (~8 lines)

Framing sentence about research being trigger-driven, not scheduled. One mandatory about journal anchoring. Reference pointer to `research-protocol.md`.

### `research-strategy.md` Renamed to `research-protocol.md`

Absorbs from SKILL.md:
1. **Research trigger rules** (5 scenarios) — new section `trigger-rules` prepended before existing content
2. **Research diversion flow** (3-step) — new section `diversion-flow` after trigger rules
3. **Journal anchoring template** — lives in `diversion-flow`

Updated sections index:
```
<sections>
- trigger-rules          # NEW — migrated from SKILL.md
- diversion-flow         # NEW — migrated from SKILL.md
- tool-hierarchy         # existing
- subagent-delegation    # existing
- conflict-resolution    # existing
- inconclusive-research  # existing
- gemini-failure         # existing
</sections>
```

Updated loading guide: "Load §trigger-rules, §diversion-flow on first research diversion. Load §tool-hierarchy, §subagent-delegation on first research execution."

File ends with no phase-routing instruction (protocol reference, not workflow stage). Diversion flow step 3 says "return to the recorded step in the decision journal."

## New `decision-journal.md` Reference

### SKILL.md Section (~8 lines)

Purpose sentence, journal location path, mandatory about append-only and checkpoint triggers. Reference pointer.

### New `references/decision-journal.md`

```
<sections>
- purpose
- entry-types
- special-entries
- checkpoint-triggers
- lifecycle
</sections>
```

- `purpose` — 4 bullet points (context preservation, session resilience, pipeline handoff, audit trail)
- `entry-types` — all three types with field lists, Category tag semantics, Tension template, mandatory about verbatim capture
- `special-entries` — Vision Baseline, Vision Expansion, Topic Map, Section Approval descriptions. Actual templates remain in phase references at trigger points.
- `checkpoint-triggers` — the 6 triggers listed explicitly
- `lifecycle` — append-only rules, existing journal handling, distinction from user-voice journals

## Content Migration Into Existing Phase References

Most migrated content is already duplicated between SKILL.md and reference files. The migration removes the SKILL.md copy and verifies the reference file's version is canonical.

### Content That Genuinely Moves (not just deduplication)

| Content | Destination | Placement |
|---|---|---|
| Vision Loop framing ("brainstorming-style dialogue, open-ended, drawing the user out") | `vision-loop.md` | Prepend to opening as phase framing context |
| Vision Expansion mandatory ("mandatory after every Vision Loop pass including loop-backs") | `vision-expansion.md` | Prepend as mandatory at section entry |
| Approach Loop thoroughness framing ("30-60+ minutes per topic") | `approach-loop.md` | Prepend to opening framing |
| Approach Loop vision-change detection specifics (scope contraction, goal shift, new use cases) | `approach-loop.md` | Verify completeness; add if missing |

### Content That's Already in References (deduplication — remove from registry only)

- Vision Loop one-question-at-a-time mandatory — already in `vision-loop.md`
- Vision Loop loop-back conditions — already in `review-loop.md` (calling side) and partially in `vision-loop.md` (receiving side)
- Broad Design Scoping constraints — already in `support-phases.md`
- Review Loop one-section-at-a-time mandatory — already in `review-loop.md`
- Review Loop vision/approach routing details — already in `review-loop.md`
- Reconciliation severity table — already in `support-phases.md`
- Final Design Doc required elements — already in `support-phases.md`
- Approach Loop research trigger note — now covered by `research-protocol.md`

## Prompt-Injection Routing Semantics

The Phase Registry acts as a prompt injector, not just a routing table. When a reference says "Proceed to Phase Registry → Vision Expansion," the reader re-reads the Phase Registry section — re-exposing themselves to:

1. Mandatory rules that apply to all phases
2. The routing table showing position in overall flow
3. The target phase's transition gate and journal checkpoint

The existing mandatory in SKILL.md (line 85) already states this: "re-read the Phase Registry entry for the target phase in SKILL.md — this is a literal instruction (use the Read tool on SKILL.md) because the framing and constraints may have compacted from context."

The lean registry improves this — re-reading ~60-70 lines (routing table + target phase gate) is more effective prompt injection than processing ~165 lines of content about phases you're not entering.

No changes needed to reference files' routing instructions. "Proceed to Phase Registry → [Phase]" is already correct.

## File Change Summary

### Modified

| File | Change |
|---|---|
| `skill/SKILL.md` | Slim Phase Registry, slim research-protocol and decision-journal sections |
| `skill/references/research-strategy.md` | Renamed to `research-protocol.md`, absorbs trigger rules and diversion flow |
| `skill/references/vision-loop.md` | Absorbs framing paragraph |
| `skill/references/vision-expansion.md` | Absorbs post-loop-back mandatory |
| `skill/references/approach-loop.md` | Absorbs thoroughness framing, verify vision-change detection completeness |

### Created

| File | Content |
|---|---|
| `skill/references/decision-journal.md` | Entry types, field lists, templates, checkpoint triggers, lifecycle rules |

### Unchanged

| File | Reason |
|---|---|
| `skill/references/review-loop.md` | Already has full content |
| `skill/references/support-phases.md` | Already has full content |

## Verification

After all changes:

1. **SKILL.md line count** — expect 280-310 lines
2. **No orphaned content** — every piece of content removed from SKILL.md exists in a reference file
3. **No broken routing** — every "Proceed to Phase Registry → [Phase]" in reference files still resolves to a valid registry entry
4. **No broken reference pointers** — `research-strategy.md` references updated to `research-protocol.md` everywhere
5. **Sections index consistency** — all `<sections>` lists match their actual `<section>` tags in modified files
6. **Mandatory preservation** — every mandatory that existed in the old SKILL.md exists either in the new SKILL.md or in a reference file (none dropped)
7. **Template preservation** — journal entry templates in phase references untouched
