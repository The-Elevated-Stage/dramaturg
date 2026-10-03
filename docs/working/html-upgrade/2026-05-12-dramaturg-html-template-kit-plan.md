---
title: "Dramaturg HTML Render Template Kit — Adaptation Plan"
date: 2026-05-12
status: proposed-reference (not yet folded into Dramaturg skill)
target: Dramaturg skill upgrade (the-elevated-stage plugin)
source-pattern: ~/Projects/kyle-projects/skills-work/elevated-stage/arranger/docs/working/html-upgrade/
test-doc: ~/kyle-projects/hockey-game/docs/plans/designs/2026-05-12-hell-on-ice-design.md
addendum: "2026-05-25 — see READ-FIRST-rich-rendering-rules.md for the rich-body-content rules added from the Argus V3 render. The plan below covers kit STRUCTURE (file layout, dispatch flow, persistence); the rules doc covers CONTENT QUALITY (visual primitives, fidelity discipline, per-tab structure pattern). Both are required reading before integrating the kit into the formal Dramaturg skill upgrade."
---

# Dramaturg HTML Render Template Kit — Adaptation Plan

A reusable kit that future Dramaturg sessions invoke to render markdown **design documents** (not implementation plans) as desktop-readable HTML review documents with the same interactive note-taking + per-section review-status layer as the Arranger kit. This plan adapts the Arranger HTML Render Template Kit to fit Dramaturg's output shape and review needs.

The Arranger kit (sibling directory) is the source pattern. This document specifies the diff — what Dramaturg needs differently, what stays the same, and what files need to be created. We do NOT re-derive the kit's design from scratch; we inherit the architecture, persistence model, render pipeline, and most agent-prompt mechanics.

---

## 1. Why Dramaturg needs this

The Dramaturg skill produces a single comprehensive design document plus an append-only decision journal. The standard Phase 6 (Review Loop) walks the user through the design section by section in the terminal for per-section approval. This conversational review surface has the same friction Kyle identified for Arranger's plan reviews: markdown is dense, structured content (tables, mappings, archetype lists) doesn't scan well in a terminal, and capturing review notes requires copy-pasting whole sections.

The HoI design doc landed this constraint sharply in practice: 16 topics settled across 3 layers, with vocabulary swap tables, archetype distributions, momentum trigger lists, and Hell-Points formulas — all material that benefits from tabbed presentation and per-section note capture rather than scrolling a 1000-line markdown file in a CLI pager.

The HTML upgrade replaces (or augments) Phase 6's terminal review with:

- **Tabbed presentation** of the design's structural sections (Goals / Phased Roadmap / v1 / v2 / v3+ / Arranger Notes — see §4 for full nav)
- **Sidebar notes** anchored to specific design statements (e.g., "the Tormentor 20% weighting feels too high to me — suggest 15%")
- **Per-section review status** (Unreviewed / Approved / Open Questions / Needs Revision) per top-level tab
- **Render Fidelity Review** by an Opus teammate — same as Arranger's pattern, catches typos/mis-wordings/dropped content the Dramaturg author missed

The user-side benefits are identical to Arranger's. The orchestration benefits are also identical: the Dramaturg session bypasses per-section conversational review (which burns context) in favor of writing the whole doc + delegating HTML rendering + delegating fidelity check to teammates. The primary session retains context for any follow-up reconciliation or design revisions.

---

## 2. Pattern inheritance from the Arranger kit

**Inherit unchanged:**

- Five-file kit structure (template HTML + 3 agent prompts + runbook)
- Pre-render `cp` step from Dramaturg session
- Two Sonnet passes (Pass 1: top-level overview; Pass 2: per-section content)
- One Opus render-fidelity review pass
- `{{double_brace}}` variable substitution
- Persistence model (single JSON file via `/api/notes`, schema_version field, phaseStates → renamed in this kit, localStorage fallback)
- Highlight rehydration via excerpt + nearest-section-id (no fragile char offsets)
- Karpathy 4 + anti-sycophancy + flag-don't-correct discipline injected in all agent prompts
- Page layout (tab nav top, content middle, notes sidebar right, floating + Top FAB)
- 4-state per-tab review control (Unreviewed / Approved / Open Questions / Needs Revision)
- Browser-sync server expectation (with localStorage fallback)
- Validation greps after each pass
- Failure recovery procedures (`cp` template fresh on partial pass failure)
- Desktop-only / Firefox / 4K scope

**Inherit with adaptation:**

- HTML template — rename references; adapt slot markers; adjust section count
- Agent prompts — re-target from "phases" to "design sections"; reflect Dramaturg-specific structure (Goals, phased v1/v2/v3+ if applicable, Arranger Notes)
- Runbook — update inputs, validation greps, dispatch sequence to match new slot markers
- Variable matrix — replace `{{phase_count}}` and `{{phase_titles}}` with Dramaturg-relevant equivalents

**New (Dramaturg-specific):**

- An optional **Journal tab** that surfaces the decision journal alongside the design doc (see §5)
- A pattern for variable section count, since not every Dramaturg design uses the v1/v2/v3+ phasing (HoI does; many designs won't)

---

## 3. Structural differences: Dramaturg design vs Arranger plan

| Dimension | Arranger plan | Dramaturg design |
|---|---|---|
| Top-level organization | Fixed: Overview → Phase Summary → Phase 1..N | Variable: Goals → Phased Roadmap (optional) → Section content (varies per design) → Arranger Notes |
| Number of major sections | Predictable (1 Overview + 1 Phase Summary + N phases) | Variable (depends on design: phased vs flat, journal-included vs not) |
| Per-section review semantics | "Is this phase implementable as written?" | "Does this design section capture intent? Is anything wrong, missing, or overstated?" |
| Content density | Tables of implementation steps, file modifications, supervisor gates | Tables of design decisions, voice guides, archetype distributions, formula derivations |
| Decision trail location | Spread across plan + supervisor gates | Separate append-only journal file (the Dramaturg journal) |
| Author-knowledge artifact | One plan markdown file | One design markdown file + the journal file |

The most consequential difference for the HTML kit: **variable section count and variable phasing**. Arranger's kit assumes "1 overview + 1 phase summary + N phases" structure. Dramaturg's kit needs to handle:

- Designs with v1/v2/v3+ phasing (HoI is the first such)
- Designs without phasing (single-shot designs)
- Designs that bundle Arranger Notes as a section
- Designs that include or exclude the journal as a separate tab

The kit's slot markers and Pass 2 prompt need to accommodate this variability without bloating the template.

---

## 4. Proposed Dramaturg tab structure

For the HoI design specifically (and for future phased designs), the tab nav should be:

```
[Overview] [Phased Roadmap] [V1] [V2] [V3+] [Arranger Notes] [Journal]
```

For non-phased designs (a single-shot design without v1/v2/v3+ structure), the nav simplifies to:

```
[Overview] [Section 1] [Section 2] ... [Section N] [Arranger Notes] [Journal]
```

Where:

- **Overview tab** = the design doc's Goals section (the verbose narrative anchor); renders as continuous prose, no sub-sections
- **Phased Roadmap tab** (if phased) = the v1/v2/v3+ at-a-glance summary; one card per phase with high-level scope
- **V1 / V2 / V3+ tabs** (if phased) = the detailed content of each phase; for HoI, V1 is the longest and most reviewed
- **Section tabs** (if non-phased) = the major content sections (one tab per top-level `##` heading in the design doc)
- **Arranger Notes tab** = the appendix (new protocols, open questions, key decisions) — distinct from other tabs because it's the explicit downstream-handoff content
- **Journal tab (optional, default ON)** = a rendered view of the decision journal, organized by topic, with cross-references back to the design sections; provides reviewer access to the full decision trail without leaving the HTML

For HoI specifically that's 7 tabs. The v2 and v3+ tabs are shorter (mostly lists) than v1 (heavy structured content); some users may approve them quickly. The Journal tab is reference material; expected review state stays "Unreviewed" or is opt-out.

**Per-section internal structure (v1 specifically, since it carries most content):**

Inside the V1 tab, the proposal renders three sub-sections (one card-styled grouping each), matching the v1 layer structure:

1. **Platform Layer** — Topics 1-4 content (multi-user, calendar, auto-pilot, Commissioner)
2. **Off-ice Game Systems** — Topics 5-11 content (Owner/GM, draft, coach, icons, contracts/FA, cash, HP)
3. **On-ice Simulation + Presentation** — Topics 12-16 content (sim engine, steroids, fighting, brand, broadcast)

Each layer renders as a card with title chip + content. This mirrors Arranger's per-phase sub-track card pattern.

---

## 5. The Journal tab — Dramaturg-specific addition

The Dramaturg decision journal is a separate artifact from the design doc. Currently the design doc references the journal via a path footer ("Decision journal at: `docs/plans/designs/decisions/<topic>/dramaturg-journal.md`"). For the HTML render, including a Journal tab provides:

- Direct visibility into the full settlement trail without context-switching to another file
- Per-topic decision cards (one per settled topic) with user verbatim, alternatives discussed, and supersession references
- Search across the entire decision history
- The Arranger receiving handoff can verify journal entries match design doc claims without opening a second file

**Render approach for the Journal tab:**
- Parse the journal markdown's `## Decision: ...` and `## Research: ...` entry headers
- Render each entry as a collapsible card (expanded by default for the most-recent, collapsed for prior topics)
- Include a tab-internal filter ("show only superseded entries" / "show only research-backed entries")
- Cross-link: clicking a "supersedes" reference in a journal entry scrolls to the superseded entry

**v1 of this kit:** include the Journal tab as static rendered content (no filtering UI; no cross-link interactivity beyond standard anchor links). Filtering UI is a polish-pass v2 of the kit.

**Configurable inclusion:** the runbook accepts a `{{include_journal}}` boolean. When false, the Journal tab is omitted from the tab nav. Default true.

---

## 6. Files to create

Mirroring the Arranger kit's five-file structure. Eventual home (when the kit is formalized into the Dramaturg skill upgrade): `<dramaturg-plugin>/skills/dramaturg/references/html-render/`. Current working location:

```
~/Projects/kyle-projects/skills-work/elevated-stage/dramaturg/docs/working/html-upgrade/
├── 2026-05-12-dramaturg-html-template-kit-plan.md   ← this file
├── 2026-05-12-dramaturg-html-template-design.md     ← TO CREATE: detailed design analog to Arranger's
├── dramaturg-doc-template.html                       ← TO CREATE: shell with slot markers
├── agent-prompt-pass1.md                             ← TO CREATE: Overview + Phased Roadmap (or first major section)
├── agent-prompt-pass2.md                             ← TO CREATE: per-section content injection
├── agent-prompt-review.md                            ← TO CREATE: Opus render-fidelity review
└── runbook.md                                        ← TO CREATE: operator's manual
```

| File | Role | Adaptation from Arranger source |
|---|---|---|
| `2026-05-12-dramaturg-html-template-design.md` | Detailed design analog to `2026-05-12-html-template-design.md` in Arranger kit | Adapt sections 1-12; replace "phase" with "section" where structurally appropriate; add §5-style Journal tab section; capture variable section-count handling |
| `dramaturg-doc-template.html` | HTML shell with slot markers | Start from `lethe-plan-template.html`. Rename slot markers (`<!-- LETHE_INJECT_* -->` → `<!-- DRAMATURG_INJECT_* -->`). Adjust tab nav for Dramaturg structure (Overview / Phased Roadmap / V1 / V2 / V3+ / Arranger Notes / Journal). Adjust CSS class names where prefixed `lethe-`. Document title set to "Dramaturg Design Review" placeholder. |
| `agent-prompt-pass1.md` | Sonnet Pass 1 — Overview + Phased Roadmap (or first non-section sections) | Start from Arranger's Pass 1. Adapt slot replacement targets (Overview, Phased Roadmap). Reflect Goals section's verbose narrative format. |
| `agent-prompt-pass2.md` | Sonnet Pass 2 — per-section content | Start from Arranger's Pass 2. Variable section count (not fixed phase count). Adapt for either phased (V1/V2/V3+) or flat (Section 1..N) layouts. Include Journal tab render if `{{include_journal}}` is true. |
| `agent-prompt-review.md` | Opus render-fidelity review | Start from Arranger's review. Adapt source-doc terminology (plan → design doc). Include journal-render fidelity check as an additional dimension. |
| `runbook.md` | Procedural runbook | Start from Arranger's runbook. Update inputs (add `{{journal_path}}`, `{{include_journal}}`, `{{is_phased}}`). Update slot-marker names in validation greps. Adapt failure recovery for variable section count. |

---

## 7. Test target: the HoI design doc

The HoI design doc just written at `/home/claude/kyle-projects/hockey-game/docs/plans/designs/2026-05-12-hell-on-ice-design.md` is the first real-world test target. Its characteristics make it ideal for kit validation:

- **Phased structure (v1/v2/v3+):** exercises the kit's phased-design rendering path
- **Variable section depth:** Goals is a single narrative; V1 has three layered sub-sections; V2 and V3+ are flat lists; Arranger Notes is structured
- **Long-form content with tables:** vocabulary swap table, archetype Fighting-stat range table, PED detection curve table, Cash inflow structure — exercises HTML table rendering
- **Cross-references:** the doc references topic numbers (Topic 1, Topic 12, etc.) and prior phase decisions — exercises the Journal tab's cross-link usefulness
- **External artifact dependency:** the doc cites the decision journal at `docs/plans/designs/decisions/hell-on-ice/dramaturg-journal.md` — exercises the Journal tab render

**End-to-end test plan:**

1. Build the Dramaturg kit files (§6 above)
2. Validate the HTML shell renders standalone (Validation Gate 1, per Arranger's pattern §12.1)
3. Run the kit against the HoI design doc:
   - Inputs: `plan_path = .../2026-05-12-hell-on-ice-design.md`, `journal_path = .../dramaturg-journal.md`, `output_path = ~/Projects/html-docs/hoi-design-full.html`, `output_report_path = ~/Projects/html-docs/hoi-design-fidelity-report.md`, `is_phased = true`, `include_journal = true`
   - Pre-render `cp` template → Pass 1 (Overview + Phased Roadmap) → Pass 2 (V1 layers + V2 + V3+ + Arranger Notes + Journal) → Render Fidelity Review (Opus)
4. Validation Gate 2: Kyle reviews the HoI HTML render + the fidelity report
5. Iterate based on Kyle's feedback
6. Once stable, fold the kit into the Dramaturg skill upgrade

---

## 8. Order of operations

The implementation order mirrors Arranger's kit (§12 of the design doc), with one Dramaturg-specific addition:

1. **Build `dramaturg-doc-template.html`** — the shell with all CSS/JS/UI mechanics + slot markers. Adapt from Arranger's `lethe-plan-template.html`:
   - Rename all `LETHE_INJECT_*` slot markers to `DRAMATURG_INJECT_*`
   - Adjust tab-nav HTML for the Dramaturg section layout
   - Update document title and global meta
   - Add a new slot marker for the Journal tab (conditional rendering based on `{{include_journal}}`)
   - **Validation Gate 1:** copy template to test output path, open in Firefox, verify the shell renders + JS interactions work (note-taking, FAB, tab switching, status control). Kyle reviews shell-only render and approves or requests changes before proceeding.

2. **Build `agent-prompt-pass1.md` and `agent-prompt-pass2.md`** — adapt from Arranger's analogs. Key adaptation: Pass 2 must handle both phased (HoI-style V1/V2/V3+) and flat (Section 1..N) structures based on `{{is_phased}}` variable.

3. **Build `agent-prompt-review.md`** — adapt from Arranger's review. Add a Journal-tab-fidelity check dimension.

4. **Build `runbook.md`** — adapt from Arranger's. Update prerequisite checks, validation greps, failure recovery for new slot markers.

5. **Build `2026-05-12-dramaturg-html-template-design.md`** — the detailed design analog to Arranger's. This formalizes the architecture, persistence model, render pipeline, variable matrix, etc. — the reference document the future Dramaturg skill upgrade will pull from.

6. **End-to-end test against the HoI design doc** (§7). **Validation Gate 2:** Kyle reviews the rendered HTML AND the fidelity report. Compares against expectations.

7. **Iterate on Kyle's feedback** from both gates; fold into the formal Dramaturg skill upgrade.

---

## 9. Configuration toggle for Dramaturg skill upgrade

When the kit is formalized into the Dramaturg skill, expose a `use_html_reviews=true|false` configuration option (parallel to the Arranger upgrade plan). Behavior:

- **`use_html_reviews=true`** — Dramaturg session writes the full design doc directly to disk (bypassing conversational Phase 6 review) and dispatches the HTML kit pipeline. Reports the rendered HTML + fidelity report path to the user. User reviews on the HTML side; Dramaturg session retains capacity for follow-up reconciliation/revisions based on user findings.
- **`use_html_reviews=false`** (default until kit is stable) — Standard Phase 6 conversational per-section review. Original Dramaturg behavior.
- **`use_html_reviews=ask`** — at Phase 6 entry, Dramaturg asks the user whether to run HTML review or standard review. Useful while the kit is in beta.

The HoI session (2026-05-12) operated as a manual application of the future `use_html_reviews=true` pattern — Kyle invoked the bypass explicitly mid-Phase-5.

---

## 10. Out of scope (for this adaptation plan)

- Implementing the kit files themselves — this plan specifies what to build; the build is the next concrete step
- Modifying the Dramaturg skill's `SKILL.md` or reference files to call the kit — that's the skill upgrade, separate from kit creation
- Multi-doc rendering (e.g., rendering both design doc AND a related plan in one HTML) — single-doc focus
- Render output to non-HTML formats (PDF, slide deck, etc.) — HTML only
- Adapting for Tier-2 formalism / Conductor pipeline — Dramaturg output is the only input

---

## 11. Open questions

- **Journal tab rendering depth:** should the journal render include ALL entries (Vision Baseline, every Topic decision, supersession entries) or just settled-Topic entries? Current proposal: all entries. Reviewer benefit favors all; render cost is one Pass-2 Edit so adding journal entries doesn't compound.
- **Inline cross-link from design doc to journal entries:** the design doc currently references topic numbers in prose ("per Topic 12 settlement..."). Should the HTML render those as inline hyperlinks to the Journal tab's corresponding entry card? Strong UX win but adds Pass 2 complexity. Could be v2 of the kit.
- **Phased vs flat design handling:** the kit needs to handle both. The current proposal uses `{{is_phased}}` boolean. Alternative: always render with the section structure derived from `##` heading parsing — no flag needed. Simpler but less explicit. **Tentative resolution:** use `{{is_phased}}` for now; revisit if heading-driven section detection proves more reliable.
- **HoI's "V1" tab carrying disproportionate content:** the V1 tab has three layered sub-sections (Platform / Off-ice / On-ice + Presentation), each substantial. Should this be three tabs instead of one nested tab? Current proposal: keep as one tab with card-styled sub-sections (matches Arranger's per-phase sub-track pattern). Push back if Kyle prefers three separate tabs for V1.
- **Notes JSON file naming:** Arranger uses `<plan-slug>-notes.json`. For Dramaturg, follow the same pattern with the design doc filename slug. No conflict expected.

---

## 12. Immediate next concrete action

After Kyle reviews this plan: **build `dramaturg-doc-template.html` by adapting `lethe-plan-template.html` from the Arranger kit.** This is the longest single file in the kit and the foundation for everything else. Once the shell renders cleanly (Validation Gate 1), the four prompt/runbook files follow quickly.

If Kyle wants to skip the explicit Validation Gate 1 review of the shell, the kit can proceed straight to end-to-end testing against the HoI design doc (§7), folding shell-quality feedback into the validation-gate-2 review of the full rendered output. This is faster but loses one feedback loop.

---

## 13. Cross-references

- Arranger kit plan: `~/Projects/kyle-projects/skills-work/elevated-stage/arranger/docs/working/html-upgrade/2026-05-12-arranger-html-template-kit-plan.md`
- Arranger kit design doc: `~/Projects/kyle-projects/skills-work/elevated-stage/arranger/docs/working/html-upgrade/2026-05-12-html-template-design.md` (large; not yet read in full during this planning session — recommended reading before implementation)
- Arranger kit shell template (reference for adaptation): `~/Projects/kyle-projects/skills-work/elevated-stage/arranger/docs/working/html-upgrade/lethe-plan-template.html`
- Arranger kit prompts: `agent-prompt-pass1.md`, `agent-prompt-pass2.md`, `agent-prompt-review.md` in same directory
- Arranger runbook: `runbook.md` in same directory
- HoI design doc (first test target): `~/kyle-projects/hockey-game/docs/plans/designs/2026-05-12-hell-on-ice-design.md`
- HoI Dramaturg journal: `~/kyle-projects/hockey-game/docs/plans/designs/decisions/hell-on-ice/dramaturg-journal.md`
- Original Dramaturg HTML upgrade pointer: `~/Projects/kyle-projects/skills-work/elevated-stage/dramaturg/docs/working/html-upgrade.md` (points to Arranger kit; this adaptation plan is the Dramaturg-side detailed plan)
