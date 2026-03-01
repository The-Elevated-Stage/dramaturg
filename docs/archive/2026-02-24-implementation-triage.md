# Dramaturg Skill — Implementation Triage

**Date:** 2026-02-24
**Source:** DRAMATURG-CHANGES.md (89 findings) + Vision Expansion design
**Purpose:** Maps every review finding to an implementation batch

---

## Batch Structure

| Batch | Scope | Count |
|-------|-------|-------|
| **1 — Structural** | Vision Expansion + findings that interact with VE | 5 findings + VE implementation |
| **2 — Behavioral** | Independent CRITICAL + IMPORTANT fixes | 33 findings |
| **3 — Polish** | Independent MINOR + SUGGESTION fixes | 45 findings |
| **Resolved** | Addressed by Vision Expansion design | 6 findings |

Batches are ordered — Batch 1 establishes the new phase structure, Batch 2 changes behavior within that structure, Batch 3 polishes.

---

## Resolved — Addressed by Vision Expansion (6)

No implementation work needed. The Vision Expansion design (2026-02-24-vision-expansion-design.md) resolves these.

- **C11** — Elicitation without enrichment → VE adds systematic enrichment
- **I3** — No fast-path for complete visions → users with complete visions breeze through VE
- **I11** — Phase numbering not formally defined → renumbered registry
- **I23** — Effort mismatch detection absent → true scope revealed during enrichment
- **S3** — "Co-visionary" framing could be more explicit → VE makes it explicit
- **S18** — Category-triggered checklists → Gemini prompt includes category-aware triggers

(I1 is a redirect to C11, not a separate finding.)

---

## Batch 1 — Structural (Vision Expansion + 5 interacting findings)

Implement the Vision Expansion phase AND these findings together. They are structurally entangled — phase routing, numbering, and flow changes that must be coordinated.

### VE Implementation

- Add Phase 3 (Vision Expansion) to Phase Registry in SKILL.md
- Renumber all subsequent phases (old 3→4, 4→5, 5→6, 6→7, 7→8)
- Update all phase references across ALL files (SKILL.md, vision-loop.md, approach-loop.md, review-loop.md, support-phases.md, research-strategy.md)
- Create new reference file: `vision-expansion.md`
- Add Vision Expansion journal entry template
- Update preamble to reflect co-visionary role

### Interacting Findings

**C1 — Vision changes only detected in Phase 5 — no Phase 4→2 routing**
- Add Phase 5→2 routing to Phase Registry (Approach Loop can detect vision changes)
- Add vision-change detection heuristic to approach-loop.md
- All vision-change routing must specify mandatory VE pass-through: Phase 5→Registry→Phase 2→Phase 3 (VE)→Phase 4
- Files: SKILL.md (Phase Registry), approach-loop.md, review-loop.md (existing routing)

**C10 — Vision change during Phase 5 — recovery path nearly full restart**
- Add guidance for scope of re-exploration after vision change
- Must account for VE: specify which enrichments carry forward vs. get re-evaluated
- Add: "Topics from old and new maps where approaches are still valid need re-confirmation, not full re-exploration"
- Files: review-loop.md, SKILL.md (Phase Registry Vision Loop entry)

**I25 — Long discussion context consumption — no pre-design phase management**
- Add context pressure awareness to Phases 2 and 3 (VE is now a third pre-design phase)
- Guidance: "If context pressure builds during Phase 2 or 3, suggest capturing progress"
- Files: SKILL.md (§conversation-style or §mandatory-rules), vision-expansion.md

**M12 — Re-do of Broad Design Scoping after vision change should be mandatory-tagged**
- With VE, the mandatory sequence after vision change is: Vision Loop → VE → BDS
- The mandatory tag must cover the full chain, not just BDS re-do
- Files: vision-loop.md (loop-back conditions)

**S6 — No guidance for skipping phases**
- Add phase-skip guidance that includes VE
- VE should be non-skippable (can't know it's unnecessary until you've done it) or have clear skip criteria
- "Vision Loop gate is non-negotiable. Vision Expansion is non-skippable. Other phases can be abbreviated."
- Files: SKILL.md

---

## Batch 2 — Behavioral (33 independent CRITICAL + IMPORTANT fixes)

These change how existing phases behave. Applied against the new 8-phase structure from Batch 1.

### SKILL.md (8 findings)

- **C3** — Remove unenforceable selective-read mandatory rule, reframe as guidance: "Focus on the section relevant to your current phase"
- **C4** — Replace fragile MEMORY.md breadcrumb system: either journal-based tracking, or scope breadcrumbs to subagent-delegated research only
- **C8** — Align "two artifacts" vs "three artifacts" — clarify Arranger Notes is part of the design doc
- **I6** — Tighten one-question exception: "If any question in the batch could produce a multi-paragraph response, it cannot be batched"
- **I8** — Make Phase Registry re-read concrete: "use the Read tool on SKILL.md"
- **I9** — Reorder §conversation-style: mandatory behavioral rules first, style guidance after
- **I15** — Update Review Loop description: "detection criteria for approach changes (with routing to Approach Loop's ripple assessment protocol)"
- **I26** — Add "research during vision gathering" to vision-loop.md reference pointer description

### vision-loop.md (2 findings)

- **I2** — Add adapted gate for non-feature work: "What change / Why needed / How measured"
- **I28** — Strengthen research-during-vision from guidance to mandatory: "When uncertain, research"

### approach-loop.md (8 findings)

- **C2** — Add "foundational shift" protocol: stop forward progress, present shift, identify affected topics, re-enter in dependency order
- **C5** — Add behavioral anchor to settlement-confirmation: "Journaled — [one-line summary]" confirmation to user
- **C9** — Delete stray `</output>` tag at EOF (line 273)
- **I7** — Add scope protection gray-zone heuristic: "Does the detail constrain future implementation choices unnecessarily?"
- **I13** — Add cascading ripple handling: "After re-settling each topic, check for further ripples"
- **I22** — Add research depth calibration: depth proportional to architectural impact
- **I24** — Add to §session-split: ensure journal entries detailed enough for fresh session reasoning
- **I27** — Make exit criteria explicit and mandatory: all topics explored + all journaled + session split considered

### review-loop.md (6 findings)

- **I12** — Add conditional entry point: "If re-entry from Review Loop for ripple assessment: start at §ripple-assessment"
- **I14** — Add resumption guidance: re-derive section breakdown, resume from first affected section
- **I16** — Add section derivation step: establish topic→section mapping before first section, confirm with user
- **I17** — Add partial approval handling: defer Section Approval until full section settled

### support-phases.md (6 findings)

- **C7** — Specify exact paths for design doc final location and journal archive location
- **C12** — Add synthesis quality guidance to §final-design-doc: distinguish decisions from inferences, note open questions, capture implied requirements
- **I4** — Add complexity threshold to Broad Design Scoping: if 10+ topics spanning 3+ systems, suggest decomposition
- **I10** — Add Context Grounding exit criteria checklist: tech stack, related features, integration points
- **I18** — Add reconciliation circuit breaker: 2 rounds max, then surface underlying tension
- **I19** — Specify exact journal archive path (overlaps with C7)

### research-strategy.md (3 findings)

- **I20** — Add subagent type guidance: Explore for codebase, general-purpose for external research
- **I21** — Reorder conflict resolution: quick Gemini check before thorough web search

### Cross-file (2 findings)

- **C6** — Make journal path session-specific: `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md`
- **I5** — Add "tension" journal entry type for contradictory requirements

---

## Batch 3 — Polish (45 independent MINOR + SUGGESTION fixes)

Wording changes, template fixes, tag corrections, clarifications. Lower risk, can be done more rapidly.

### SKILL.md (3)
- **M1** — "Repetiteur" → "Arranger" (find and replace)
- **M2** — Align journal format (Tier 3 vs plain markdown)
- **M32** — Add consistent section restrictions to reference tags, or acknowledge full-read intent

### vision-loop.md (5)
- **M5** — Remove duplicate exit section
- **M8** — Fix example questions that violate one-at-a-time
- **M9** — Consolidate two overlapping mandatory blocks for research-during-vision
- **M10** — Clarify re-verify: full re-presentation, not just acknowledgment
- **M11** — Add position-tracking for Context Grounding loop-back
- **M13** — Define status vocabulary (settled, confirmed, etc.)

### approach-loop.md (7)
- **M14** — Add "Restated for emphasis" to duplicated mandatory rules
- **M15** — Add fast-path for trivially-settled topics matching established patterns
- **M16** — Add explicit "read Topic Map entry" before topic selection
- **M17** — Add "Supersedes" field to journal templates for ripple re-settlement
- **M18** — Journal infeasibility findings (what was ruled out)
- **M19** — Frame inline research protocol as reminder, or remove duplication
- **M20** — Clarify session-split exit: who proceeds to Review Loop

### review-loop.md (7)
- **M21** — Specify which sections need re-review after ripple return
- **M22** — Clarify "remain in place": entries preserved, reconciliation re-evaluates
- **M23** — Make security lens concrete: data retention, access control specifics
- **M24** — Define user approval: "any clear positive signal; if ambiguous, confirm"
- **M29** — Upgrade section ordering from `<core>` to `<mandatory>`
- **M30** — Clarify: one logical section per turn, may span messages
- **M31** — Add: read Vision Baseline journal entry before regression testing

### support-phases.md (3)
- **M25** — Topic Map status: "settled" → "confirmed"
- **M26** — Add instruction: if user says coverage incomplete, add topics and re-confirm

### research-strategy.md (4)
- **M6** — Add loading boundary markers
- **M7** — Add: research diversions are sequential, not nested
- **M27** — Add degraded Gemini handling (intermittent, low-quality, generic responses)
- **S15** — Add general-purpose subagents to tool-hierarchy
- **S16** — "Web Wins" → "Ground Truth Wins"

### Cross-file / Other (8)
- **M3** — Add section-level re-entry hints to Phase Registry
- **M4** — Add breadcrumb stale-on-session-death recovery note (related to C4)
- **M28** — Add mechanism to surface organic scope expansion to user
- **S1** — Add session bootstrap/resumption protocol
- **S2** — Clarify README authority hierarchy (design doc vs SKILL.md)
- **S4** — Add "Next Steps" at end of Phase 8 mentioning Arranger
- **S5** — Add research annotation field to Vision Baseline template
- **S7** — Add journal corruption recovery guidance
- **S8** — Add cross-reference to SKILL.md in session-split section
- **S9** — Add grouping comments to Review Loop sections list
- **S10** — Add Topic Map coverage verification to exit check
- **S11** — Add repeated loop-back detection
- **S12** — Add reconciliation recovery when no Section Approval entries
- **S13** — Add guidance for all-alternatives-infeasible → escalate to vision change
- **S14** — Add verbatim capture guidance: design-relevant sentence, not full transcription
- **S17** — Add UNRESEARCHED as third Arranger note status
