# Dramaturg README Deep Dive

**Date:** 2026-02-26  
**Scope:** Audit current `README.md`, compare against current Dramaturg runtime docs, and benchmark against Lethe README depth/style.

## Direct Answers

1. **Does Dramaturg have a README?** Yes: `dramaturg/README.md`.
2. **Is it up-to-date with Dramaturg v1.0 content?** Partially. It is directionally correct, but materially incomplete and somewhat stale relative to the current skill contract and docs structure.

## What I Reviewed

- `README.md`
- `skill/SKILL.md`
- `skill/references/*.md`
- `docs/designs/2026-02-17-dramaturg-skill-design.md`
- `docs/working/DRAMATURG-CHANGES.md`
- `docs/working/2026-02-24-implementation-triage.md`
- `docs/working/2026-02-24-vision-expansion-design.md`
- Lethe baseline: `/home/kyle/claude/skills_staged/kyle-skills/lethe/README.md`

## Findings

### A) README-to-Runtime Drift

1. **README lacks most of the runtime contract now present in `SKILL.md`.**
   - Current README is only 53 lines and covers pipeline/structure/install/status.
   - It does not document core runtime behavior now required by v1.0:
     - Gemini-required gate (`skill/SKILL.md:25`)
     - 8-phase workflow with mandatory Vision Expansion (`skill/SKILL.md:89-99`, `138-152`)
     - research diversion protocol (`skill/SKILL.md:267-330`)
     - decision journal model/statuses/templates (`skill/SKILL.md:333-404`)
     - session bootstrap/resume protocol (`skill/SKILL.md:455-467`)

2. **README repository tree is stale after docs structure standardization.**
   - README only shows `docs/designs/...` (`README.md:20-23`).
   - Repo now also includes:
     - `docs/working/*`
     - `docs/archive/.gitkeep`
   - This changed in commit `8a51a05` ("standardize docs/ structure").

3. **README does not describe artifact lifecycle now formalized in references.**
   - Current runtime defines specific design/journal paths and archival behavior:
     - design doc path: `docs/plans/designs/...` (`support-phases.md:202`)
     - journal archive path: `docs/archive/plans/designs/...` (`support-phases.md:255`)
   - README still uses broad high-level language and misses these mechanics.

4. **README status block likely stale or too weakly specified.**
   - `Deployment: staged (not yet deployed)` (`README.md:47`) has no date/version context and no release state detail.
   - Skill is explicitly marked `version="1.0"` in `SKILL.md` (`skill/SKILL.md:6`).

### B) Internal Doc Inconsistencies (Blockers Before Final README Rewrite)

These are not README issues alone, but they must be resolved or consciously normalized in README wording:

1. **Journal archival destination mismatch between SKILL hub and reference file.**
   - SKILL says archive to `docs/archive/` "alongside design doc" (`skill/SKILL.md:257`).
   - Support phase says archive to `docs/archive/plans/designs/...` and design doc stays in `docs/plans/designs/` (`support-phases.md:255`).

2. **Artifact count framing mismatch (2 vs 3).**
   - SKILL preamble says two artifacts (design doc + journal) and notes Arranger Notes is part of design doc (`skill/SKILL.md:61-63`).
   - Support phases context says Arranger receives three artifacts including Arranger Notes appendix (`support-phases.md:269-273`).

Recommendation: settle these two source-of-truth choices first, then encode one consistent story in README.

### C) Gap vs Lethe README Depth/Shape

Lethe README has strong operational completeness. Dramaturg README currently does not match that bar.

Lethe includes:
- usage modes and invocation paths
- configuration model
- architecture/how-it-works flow
- explicit phases
- safety and failure behavior
- testing evidence
- project structure detail
- planned features and requirements

Dramaturg README is currently missing equivalents for most of the above (where applicable).

## What to Change in Dramaturg README (Lethe-Style Blueprint)

### 1) Expand from "overview doc" to "operator doc"

Add sections equivalent in rigor to Lethe:

1. **Positioning**
   - Dramaturg vs Brainstorming vs Deep-plan (scope boundaries).
2. **Installation**
   - Local install path + any plugin install guidance (if applicable).
3. **Usage**
   - Explicit invocation: `/dramaturg`.
   - Expected workflow entry/exit.
4. **How It Works**
   - 8-phase table (Context Grounding → Final Design Doc).
   - Loop-back rules and phase-registry behavior.
5. **Research Protocol**
   - Gemini requirement, fallback behavior, diversion flow.
6. **Decision Journal**
   - Why it exists, entry types, statuses, checkpoint triggers.
7. **Outputs and Handoff**
   - Design doc path pattern, journal path, Arranger handoff semantics.
8. **Safety / Guardrails**
   - One-question-at-a-time, one-section-at-a-time, scope protection, priority chain.
9. **Session Management**
   - Session split and bootstrap/resume behavior.
10. **Project Structure**
   - Include `docs/designs`, `docs/working`, `docs/archive`, `skill/references`.
11. **Status**
   - v1.0 state with date and known follow-ups.
12. **Requirements**
   - Gemini MCP + other tool assumptions.

### 2) Keep and strengthen authority hierarchy

Current authority section is good and should remain, but include explicit hierarchy:
1. `skill/SKILL.md` (runtime truth)
2. `skill/references/*.md` (phase execution detail)
3. `docs/designs/*` and `docs/working/*` (rationale/history)

### 3) Add "Known Documentation Caveats" until source conflicts are resolved

Short temporary note in README that calls out the archival-path and artifact-count inconsistencies if not fixed first.

## Proposed Execution Plan

1. Resolve source conflicts in skill docs:
   - archival path wording
   - artifact-count framing
2. Rewrite README to Lethe-style structure above.
3. Cross-check every README claim against:
   - `SKILL.md`
   - all `skill/references/*.md`
   - current `docs/` tree
4. Add a lightweight "README drift check" checklist in `docs/working/` for future updates.

## Definition of Done for README Refresh

- README reflects all current Dramaturg phases (including Vision Expansion).
- README includes explicit `/dramaturg` usage and prerequisites.
- README includes artifact lifecycle and handoff behavior.
- README tree matches actual repository layout.
- README authority section is explicit and conflict-free.
- No contradictory statements vs `SKILL.md` and reference files.

