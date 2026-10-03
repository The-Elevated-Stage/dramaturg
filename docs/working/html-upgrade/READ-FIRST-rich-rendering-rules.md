# READ-FIRST — Rich-Rendering Rules for Dramaturg HTML Output

**Status:** load-bearing reference (2026-05-25, post-Argus-V3 render learnings)
**Authority:** these rules MUST be followed for every Dramaturg HTML render going forward
**Companion docs:** `2026-05-12-dramaturg-html-template-kit-plan.md` (kit overview), `dramaturg-doc-template.html` (the shell), `runbook.md` (dispatch sequence)
**Where these rules came from:** the Argus V3 design render at `~/Documents/kyle-sessions/docs/argus-v3/html-review/argus-v3-design-review.html` (4,506 lines, 15 tabs) — Kyle's first explicit "flair" request and the test bed for everything below.

---

## 0. Why this file exists

The original template upgraded Dramaturg with two HTML benefits: **tabbed navigation** and **anchored notes**. The body of each tab was a 1:1 markdown→HTML transcription — plain headings, paragraphs, lists, and tables. This is acceptable for small designs (≤5 tabs, short sections), but it produces walls of text for any substantial design.

**The Argus V3 render established a new policy:** the BODY of each tab must use rich visual primitives (tables, diagrams, matrices, color-coded callouts, hierarchy trees, decision flows, budget ladders, etc.) so reviewers can scan dense sections without losing the large-scale scope. Plain prose is the fallback, not the default.

This file is the load-bearing reference for that policy. It is intentionally prescriptive and "strict" so future Dramaturg sessions (or skill upgrades) have unambiguous guidance.

---

## 1. Scope — when these rules apply

**ALWAYS apply** for any of these triggers:
- Design has ≥5 tabs (reviewer fatigue compounds at scale)
- Any tab contains a list of ≥3 comparable units (roles, options, modes) → use `role-grid`
- Any tab contains a numeric threshold ladder → use `budget-ladder`
- Any tab contains an authority/visibility/reachability table → use `matrix-table` with yes/no/conditional cells
- Any tab contains a parent-child hierarchy with addressing/permission rules → use `hierarchy-tree` with worked examples
- Any tab references settled-vs-rejected alternatives → use `callout` variants (decision / rejected) with semantic color
- Design has cross-cutting principles → use `principle-grid` (one card per principle)

**MAY skip** only for: single-shot designs without phasing, ≤3 tabs, short narrative-only content.

When in doubt: apply the rules. The cost of "too much visualization" is dramatically lower than the cost of a wall of text.

---

## 2. Universal rules (the 10 commandments)

### Rule 1 — Every tab opens with a `summary-card`
The card answers, in ≤7 seconds: **what this tab covers, what's settled, what the metrics are, what status it's in.** Required structure:

```html
<div class="summary-card">
  <div class="summary-label">At a glance</div>
  <div class="summary-title">One-sentence claim that captures the whole tab.</div>
  <div class="summary-blurb">2-4 sentences of why-this-matters context.</div>
  <div class="summary-grid">
    <div class="sg-item"><div class="sg-label">Metric</div><div class="sg-value">…</div></div>
    <!-- 4-8 metric items, each ≤3 words in value -->
  </div>
</div>
```

The `summary-card` is the **contract**: a reviewer who only reads the summary cards should be able to navigate the design competently. Long-form content is for reviewers who want to drill in.

### Rule 2 — Every numerical/structural artifact uses a visual primitive (not prose)
- Thresholds → `budget-ladder` (vertical %-gauge with severity coloring)
- Roles/options/modes → `role-grid` of `role-card`s
- Hierarchies → `hierarchy-tree` (ASCII-style with colored node chips)
- Ordered steps → `cascade-list` (numbered, with allow/fail color cues)
- Reachability/visibility/authority → `matrix-table` (yes/no/conditional cells)
- State transitions → `decision-flow` (mini-flowchart)
- Settled-vs-prior comparison → `compare-table` with `col-old`/`col-new` cells
- Key-value attributes (3-8) → `kv-grid`
- Timeline/walkthrough → `event-trace` (numbered, with actor tags)
- Tool signatures → `code-spec` (dark-themed)
- Wire-format previews (e.g., pointer message format) → `pointer-preview`

**Anti-pattern:** rendering a threshold ladder as a markdown table. Wrong even if the source did. Convert to `budget-ladder`.

### Rule 3 — Every panel ends with an inline `<h3>Arranger notes</h3>` section (when source has one)
The journal contains per-decision arranger notes (VERIFIED / PARTIAL with hints). These MUST surface inline at the bottom of the corresponding panel, BEFORE the review-status fieldset. Reviewer sees the implementation-readiness verdict while reviewing the panel, not after.

Wrap with status badges:
```html
<h3>Arranger notes</h3>
<p><span class="badge badge-verified">VERIFIED</span> [paragraph] …</p>
<p><span class="badge badge-partial">PARTIAL</span> Arranger should verify: …</p>
```

If the source genuinely has no arranger note, skip; do not invent one to fill the slot.

### Rule 4 — Verbatim discipline for ALL quoted-attribution text
ANYTHING rendered as `<em>"..."</em>` and attributed to a person (Kyle, CEO #5, etc.) MUST trace to source materials verbatim. Single-word capitalization changes need `[...]` editorial markers. Paraphrases must NOT be styled as quotes.

**This is the #1 fidelity failure mode.** It happens when a section header is paraphrased into a quote-shaped sentence. The fidelity review catches it; don't let it through in the first place.

If a quote IS verbatim but you can't attribute the exact source location, do NOT render it as a quote. Use neutral phrasing ("per Kyle's framing…").

### Rule 5 — Numbers, codes, signatures match source verbatim
- All thresholds (percentages, counts, time windows, TTLs) — verbatim
- All `reason` codes (e.g., `crewmate_name_collision`) — verbatim
- All tool names (`argus_bash`, `mcp__hermes__send_message`) — verbatim
- All file paths (`~/.argus/handoffs/<uuid>.txt`) — verbatim
- All default values — verbatim

The HTML may correct a clear typo in the source (e.g., a wrong section-number reference), but the fidelity report should flag it for upstream correction.

### Rule 6 — Tab nav uses per-tab key prefixes
Each tab label has a short key prefix so reviewers can scan by topic-key, not just by label:
```html
<button class="tab-btn" data-tab-key="t1-routing" data-state="unreviewed">
  <span class="tab-key">T1</span>Routing &amp; Authority
</button>
```
Key prefix appears in muted monospace. Keys: section numbers (§1), topic numbers (T1, T2), or symbols (★ for Goals, » for Arranger Notes, ¶ for Journal).

### Rule 7 — Semantic color palette (don't invent new colors)

Use existing CSS tokens. When a new variant is needed, ADD it to the token block (don't inline RGB).

| Intent | Token | When to use |
|---|---|---|
| Approved / settled | `--c-approved` (green) | Settled decisions, "OK" cells, accepted outcomes |
| Open question / warn | `--c-warning` (amber) | Open questions, partial verifications, caveats |
| Rejected / blocked | `--c-rejected` (red) | Rejected alternatives, NO cells, dropped scope |
| V4 deferred | `--c-v4` (magenta) | Deferred-to-V4 items |
| V5+ horizon | `--c-v5` (gray) | V5+ horizon notes |
| Primary (CEO) | `--c-primary-clr` (orange-brown) | Primary-session exception, CEO-specific |
| Decision / info | `--c-decision` (blue), `--c-info` (teal) | Architectural choices, informational asides |
| Principle | `--c-principle` (purple) | Cross-cutting principles |
| Key / load-bearing | `--c-key` (green) | Foundational architectural reveals |
| Action / shutdown | `#1f2937` (dark) | Action steps that change system state |

Color choice MUST follow intent, not aesthetic preference. A "warning" callout in blue is wrong.

### Rule 8 — Worked examples for foundational topics
Topics that other sections depend on (T1 addressing, T9 plugin pattern) MUST include a worked example: a `hierarchy-tree` showing concrete instances, then a bulleted list of "X calls Y → OK / REJECTED" walking through edge cases. Cement the rules with concrete cases.

### Rule 9 — Mandatory Opus fidelity review
After the HTML is rendered, dispatch an Opus subagent to cross-check rendered content against the source design doc + journal. The agent writes a fidelity report at `<output_dir>/fidelity-report.md`. The author MUST address every CRITICAL and HIGH finding before delivery; MEDIUM findings should be addressed; LOW findings noted.

This is non-negotiable. The fidelity review caught an invented Kyle quote in the V3 render that two passes of authorial review missed.

### Rule 10 — Direct authorship is acceptable (and often better) for designs with custom visualizations
The kit's Pass 1/Pass 2 prompts produce competent markdown-to-HTML. They do NOT (and cannot, without enormous prompt expansion) produce custom diagrams, hierarchies, or matrices. For substantial designs with structural content:

- **Direct authorship:** the Dramaturg author writes the HTML body directly (after writing the design doc + journal). Better visual control; higher context cost; one-shot result.
- **Agent dispatch (Pass 1/Pass 2):** the kit's standard flow. Lower context cost; better for plain-prose-heavy designs; less visual variety.

Choose based on design shape. Argus V3 used direct authorship. The kit's standard flow remains valid for designs that don't need custom visualizations.

When dispatching agents, the Pass 2 prompt MUST reference this file so the agent knows the visual primitive library exists.

---

## 3. Per-tab structure pattern (load-bearing)

Every tab MUST follow this structure:

```html
<div id="panel-<key>" class="tab-panel" role="tabpanel">
  <div class="phase-content" id="<key>-content">

    <!-- 1. H1 title with optional suffix -->
    <h1><span class="h1-suffix">§N / TX</span> Topic name</h1>

    <!-- 2. summary-card (REQUIRED) -->
    <div class="summary-card"> … </div>

    <!-- 3. Optional: opening callout for cross-cutting context -->
    <div class="callout callout-warning"> … </div>

    <!-- 4. Sub-sections (H2 with §N.M key prefix) -->
    <h2><span class="h2-key">§N.M</span>Sub-section title</h2>
    [body with visual primitives]

    <!-- … more sub-sections … -->

    <!-- 5. Arranger notes (REQUIRED when source has one) -->
    <h3>Arranger notes</h3>
    <p><span class="badge badge-verified">VERIFIED</span> … </p>
  </div>

  <!-- 6. Review-status fieldset (REQUIRED — exists in template) -->
  <fieldset class="review-status" id="rs-<key>" data-tab-key="<key>">
    [4 radio options: unreviewed / approved / open-questions / needs-revision]
  </fieldset>
</div>
```

**Order matters:** summary-card always first after H1; arranger notes always last before review-status. The reviewer's eye expects this rhythm.

---

## 4. Tab nav rules (variable tab count)

Three coordinated places hold the tab list. All three must be kept in sync:

1. **HTML tab nav** — `<button class="tab-btn" data-tab-key="…">…</button>` × N
2. **HTML tab panels** — `<div id="panel-…" class="tab-panel">…</div>` × N (plus their `<fieldset class="review-status" data-tab-key="…">` × N)
3. **JS `TAB_KEYS` array** — must be a 1:1 mirror of HTML `data-tab-key` attributes in tab-nav order
4. **JS `TOTAL` constant** — must equal the tab count

The template ships with 15 tabs (Argus V3 default). For different designs:
- Add/remove `<button class="tab-btn" …>` in the tab nav
- Add/remove corresponding panel div + fieldset
- Update `TAB_KEYS` array
- Update `TOTAL` constant
- Update header `<div id="progress-label">N of M reviewed</div>` initial M

**Validation grep:**
```bash
grep -c 'class="tab-btn"' file.html     # nav count
grep -c 'class="tab-panel"' file.html   # panel count
grep -c 'class="review-status"' file.html  # fieldset count
# All three counts must be equal AND equal to TOTAL in JS
```

---

## 5. CSS primitives library (reference)

All primitives live in the template's `<style>` block. This section lists each with intent + when-to-use. The actual CSS is in `dramaturg-doc-template.html` — see §10 for the visual primitives block.

| Primitive | Intent | When to use |
|---|---|---|
| `summary-card` + `summary-grid` + `sg-item` | At-a-glance opening | Top of every tab |
| `badge` (16 variants) | Compact status/category | Inline status markers |
| `callout` (9 variants) | Highlight important context | Maximum 2-3 per sub-section to avoid noise |
| `layer-stack` + `layer-row` | Stacked architectural layers | N-tier architecture diagrams |
| `phase-flow` + `pf-cell` | Horizontal phase ladder | Showing parallel/sequential phases with status |
| `principle-grid` + `principle-card` | N-card cross-cutting principles | 3-6 principles with equal visual weight |
| `hierarchy-tree` + `ht-node` | ASCII tree with colored chips | Parent/child hierarchies with worked examples |
| `cascade-list` (ol) | Numbered steps | Ordered resolution/decision steps |
| `matrix-table` + `cell-yes`/`cell-no`/`cell-cond`/`cell-na` | Reachability matrix | Caller-vs-target authority, role-vs-scope |
| `alias-chip` + `alias-chip-row` | Compact tag-style | Reserved aliases, regex patterns, short technical terms |
| `role-card` + `role-grid` | Detail cards with label/value rows | Enumerating N comparable units |
| `budget-ladder` + `bl-row` | Vertical %-gauge | Threshold ladder (budget, ctx, etc.) |
| `decision-flow` + `df-node` | Vertical mini-flowchart | Decision tree, flow logic |
| `compare-table` + `col-old`/`col-new` | V3 vs prior side-by-side | Surfacing what changed |
| `kv-grid` + `kv-key`/`kv-val` | Compact key-value | 3-8 named attributes |
| `event-trace` (ol) | Numbered timeline with actor tags | Walkthrough trace |
| `code-spec` | Dark-themed code block | Tool signatures, JSON shapes |
| `pointer-preview` | Yellow-themed wire-format block | Pointer messages, wire formats |
| `qa-block` + `qa-q`/`qa-a` | Open question with tag | Arranger-notes open-questions section |
| `sec-anchor` | Section divider with key + title | Major sub-section break within a tab (optional) |

**Strict rule:** before inventing new CSS, check if an existing primitive fits. If a new primitive is genuinely needed, add it to the token block + this table.

---

## 6. Badge taxonomy (strict — do not invent new ones casually)

| Badge | Use for |
|---|---|
| `badge-settled` | Settled decisions |
| `badge-dropped` | Rejected / dropped alternatives |
| `badge-deferred` | Deferred to a later version |
| `badge-v4` | Specifically V4-deferred (alias for deferred when version-specific) |
| `badge-v5` | V5+ horizon items |
| `badge-primary` | Primary/CEO-specific items |
| `badge-new` | New in V3 (or new in this version) |
| `badge-verified` | Arranger note: VERIFIED |
| `badge-partial` | Arranger note: PARTIAL |
| `badge-open` | Open question / unresolved |
| `badge-info` | Informational severity |
| `badge-warn` | Warning severity |
| `badge-urgent` | Urgent severity |
| `badge-critical` | Critical severity |
| `badge-action` | Action step (changes system state) |
| `badge-key` | Foundational / load-bearing decision |

If a new semantic intent is needed, add a token + a badge variant + a row here.

---

## 7. Callout taxonomy (strict)

| Callout | Color | Use for |
|---|---|---|
| `callout-decision` | Blue | Architectural choice; "why we picked this" |
| `callout-warning` | Amber | Caveat reviewer must know |
| `callout-info` | Teal | Informational aside; "by the way" |
| `callout-rejected` | Red | Rejected alternative; "we did NOT do X" |
| `callout-principle` | Purple | Cross-cutting principle restated |
| `callout-key` | Green | Foundational / load-bearing reveal |
| `callout-primary` | Orange | Primary/CEO-specific note |
| `callout-quote` | Gray italic | Verbatim quote with attribution |

Callouts have structure:
```html
<div class="callout callout-X">
  <div class="co-head"><span class="co-icon">⚠</span> Head text</div>
  <div class="co-body">Body content (paragraphs, lists, code).</div>
</div>
```

Icons: pick one short symbol or letter, not a long phrase. Common choices: `⚠` (warning), `ℹ` (info), `◆` (decision), `▣` (key), `✕` (rejected), `★` (primary), `"` (quote), `⇄` (asymmetric), `⇣` (downstream), `⊳` (deferred), `∑` (computation), `⊘` (retired).

---

## 8. Fidelity discipline (mandatory checks before delivery)

Run these checks before reporting "render complete":

### Structural

```bash
test "$(tail -1 file.html | tr -d '\n')" = '</html>'        # ends correctly
grep -c 'V3_INJECT_\|DRAMATURG_INJECT_' file.html            # → 0
grep -c 'injection-placeholder' file.html | grep -v ':1$'    # only the CSS rule definition
TAB_COUNT=$(grep -c 'class="tab-btn"' file.html)
[ "$TAB_COUNT" = "$(grep -c 'class="tab-panel"' file.html)" ]
[ "$TAB_COUNT" = "$(grep -c 'class="review-status"' file.html)" ]
```

### Content

- Every threshold/code/signature in HTML traces to source verbatim
- Every `<em>"..."</em>` quote traces to source verbatim (or uses `[...]` for editorial change)
- Every panel that should have arranger notes has them
- Tab nav and panel divs match (count + key strings)

### Fidelity review

Dispatch Opus to write a fidelity report. Address CRITICAL+HIGH before delivery. Document MEDIUM/LOW.

---

## 9. Authorship trade-off

| Approach | Use when | Trade-offs |
|---|---|---|
| **Direct authorship** | Design has structural content needing custom visualizations (matrices, hierarchies, flowcharts) | Higher context cost; better visual control; one-shot |
| **Agent dispatch (Pass 1/2)** | Design is mostly prose with standard tables/lists; reusability matters | Lower context cost; less visual variety; more idiomatic for the kit |
| **Hybrid** | Shell + most tabs via agent; visually-rich tabs hand-authored | Best of both; coordination cost; explicit handoff between author and agent |

The kit's Pass 2 prompt MUST be updated to reference this rules file when designs are dispatched via the kit (see §10 of `agent-prompt-pass2.md`).

---

## 10. Validation checklist (pre-delivery)

- [ ] File ends with `</html>`
- [ ] All injection markers consumed (none of `V3_INJECT_*` / `DRAMATURG_INJECT_*` remain)
- [ ] Tab nav, panels, and review-status fieldsets all count to TOTAL
- [ ] JS `TAB_KEYS` array is 1:1 with HTML `data-tab-key` attributes
- [ ] JS `TOTAL` constant equals tab count
- [ ] Every tab opens with a `summary-card`
- [ ] Every threshold ladder uses `budget-ladder`
- [ ] Every matrix uses `matrix-table` (not a plain table)
- [ ] Every hierarchy uses `hierarchy-tree`
- [ ] Every quoted-attribution text traces to source verbatim
- [ ] Every panel with a source arranger note has `<h3>Arranger notes</h3>` inline
- [ ] Tab labels include short key prefixes (`<span class="tab-key">T1</span>`)
- [ ] No invented CSS colors (all from `--c-*` tokens)
- [ ] No invented badge / callout variants without adding tokens
- [ ] Opus fidelity review dispatched and CRITICAL+HIGH addressed

When all boxes are checked, the render is ready for delivery.

---

## 11. What changed from the original template (kit history)

For future reference when integrating these rules into the formal Dramaturg skill upgrade:

### Original template (`dramaturg-doc-template.html` as of 2026-05-12)
- 7 hardcoded tab keys (`overview`, `phased-roadmap`, `v1`, `v2`, `v3`, `arranger-notes`, `journal`)
- 4 CSS classes for body content: `subtrack-grid`, `subtrack-card`, `supervisor-gate`, `injection-placeholder`
- 1 hardcoded `TOTAL = 8` in JS (off-by-one vs the 7 tabs — pre-existing bug)
- Body content style: plain markdown→HTML transcription

### Argus V3 render (2026-05-25) — what was added
- Variable tab count (Argus V3 ships 15 tabs)
- 20+ new CSS primitives for rich body content (see §5)
- 16 badge variants (see §6)
- 9 callout variants (see §7)
- Per-tab `summary-card` pattern (mandatory)
- Per-tab inline arranger notes (mandatory when source has them)
- Tab nav with per-tab key prefixes
- Mandatory Opus fidelity review
- Semantic color palette tied to intent (see §2 Rule 7)

### What's now in the upgraded template
The upgraded template (`dramaturg-doc-template.html` as of 2026-05-25) includes ALL CSS primitives from the V3 render. The HTML body uses 7 stub tabs (kept for backward compatibility with the existing Pass 1/Pass 2 prompts), with comments indicating where to regenerate the tab structure for variable-tab designs. The `TOTAL = 8` bug is fixed to `TOTAL = 7` (matching the 7 stub tabs).

### What still needs updating (future work)
- **Pass 2 prompt** (`agent-prompt-pass2.md`): currently instructs agents to use only `card` + `callout` + `<table>`. Should be expanded to know about the full CSS primitive library and reference this rules file.
- **Pass 1 prompt** (`agent-prompt-pass1.md`): currently doesn't know about `summary-card`. Should add a section instructing the agent to always open with a `summary-card`.
- **Review prompt** (`agent-prompt-review.md`): currently checks fidelity vs source markdown. Should add explicit checks for "no invented quotes" and "every threshold-list uses `budget-ladder`".
- **Runbook** (`runbook.md`): should reference this rules file as a prerequisite read for any author working with the kit.

---

## 12. The one-paragraph summary (for skill upgrade integration)

> Dramaturg HTML renders use a rich-visualization policy: every tab opens with a summary card, every threshold/hierarchy/matrix uses a dedicated CSS primitive (not plain prose), every per-panel arranger note surfaces inline, all attributed quotes trace to source verbatim, and an Opus fidelity review is mandatory before delivery. Direct authorship is acceptable (often better) for designs with custom visualizations; agent dispatch via Pass 1/Pass 2 is for prose-heavy designs and must reference this rules file. The full CSS primitive library and badge/callout taxonomies are in the template's `<style>` block — when in doubt, use an existing primitive; never invent new colors.

End of READ-FIRST.
