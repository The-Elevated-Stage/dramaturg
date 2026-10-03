# Dramaturg HTML Render Template Kit — Runbook

**Date:** 2026-05-12 (kit), 2026-05-25 (rich-rendering rules upgrade)
**Status:** working (not yet folded into Dramaturg skill)
**Kit location:** `~/Projects/kyle-projects/skills-work/elevated-stage/dramaturg/docs/working/html-upgrade/`

> **READ FIRST:** Before running the kit on a new design, read `READ-FIRST-rich-rendering-rules.md` in this directory. It defines the universal rules for rich body content (summary cards, visual primitives, fidelity discipline) added 2026-05-25 from the Argus V3 render learnings. The runbook below describes the procedural flow; the rules doc describes the content quality bar.

---

## Overview

This runbook is the operator's manual for the Dramaturg HTML Render Template Kit. The kit takes a markdown design document (and optionally its decision journal) as input and produces a desktop-readable, interactive HTML review document — complete with tabbed section navigation (Overview / Phased Roadmap / V1 / V2 / V3+ / Arranger Notes / Journal), a persistent notes sidebar, per-tab review status controls, and a fidelity report written by an Opus review teammate.

**Who uses this runbook:** a future Dramaturg session (an LLM operating in a Claude Code context). Read this top-to-bottom and execute without skipping steps. Each step names its next step and its stopping criterion.

**Prerequisites:**
- The Dramaturg session has the ability to dispatch subagents via the `Agent` tool with `model="sonnet"` and without a model override (Opus default).
- A browser-sync server with notes-middleware is running (or the user understands that notes persistence falls back to `localStorage` only).
- The source markdown design document exists and is readable.
- The Dramaturg decision journal exists and is readable (if `{{include_journal}}=true`).
- The kit directory contains: `dramaturg-doc-template.html`, `agent-prompt-pass1.md`, `agent-prompt-pass2.md`, `agent-prompt-review.md`.

**Estimated wall-clock time:** 20–30 minutes per render cycle (Pass 1 + Pass 2 + Review, fully sequential — no step parallelizes with another).

---

## Inputs Needed at Run Time

Collect all inputs before starting. Record them; you will substitute them into every step.

| Input | Description | Example |
|---|---|---|
| `<plan_path>` | Absolute path to source markdown design document | `/home/claude/kyle-projects/hockey-game/docs/plans/designs/2026-05-12-hell-on-ice-design.md` |
| `<journal_path>` | Absolute path to the Dramaturg decision journal | `/home/claude/kyle-projects/hockey-game/docs/plans/designs/decisions/hell-on-ice/dramaturg-journal.md` |
| `<output_path>` | Absolute path for the rendered HTML output | `/home/claude/Projects/html-docs/hoi-design-full.html` |
| `<output_report_path>` | Absolute path for the fidelity report output | `/home/claude/Projects/html-docs/hoi-design-fidelity-report.md` |
| `<is_phased>` | Boolean: does the design use v1/v2/v3+ phasing? | `true` (HoI is phased) |
| `<include_journal>` | Boolean: render the Journal tab? | `true` (default; set false to suppress) |

The kit directory path is referred to as `<kit_dir>` throughout this runbook. Set it to the directory containing this file.

---

## Prerequisite Checks

Run these before dispatching anything. If any check fails, stop and resolve it before proceeding.

```bash
# 1. Source design doc exists and is readable
test -r <plan_path> && echo "OK: design doc readable" || echo "FAIL: design doc not found or not readable"

# 2. Journal exists and is readable (only if include_journal=true)
test -r <journal_path> && echo "OK: journal readable" || echo "FAIL: journal not found or not readable (only blocks if include_journal=true)"

# 3. Output directory is writable
test -w "$(dirname <output_path>)" && echo "OK: output dir writable" || echo "FAIL: output dir not writable"

# 4. Kit template exists
test -f <kit_dir>/dramaturg-doc-template.html && echo "OK: template found" || echo "FAIL: template missing"

# 5. Prompt templates exist
test -f <kit_dir>/agent-prompt-pass1.md && echo "OK: pass1 prompt found" || echo "FAIL: pass1 prompt missing"
test -f <kit_dir>/agent-prompt-pass2.md && echo "OK: pass2 prompt found" || echo "FAIL: pass2 prompt missing"
test -f <kit_dir>/agent-prompt-review.md && echo "OK: review prompt found" || echo "FAIL: review prompt missing"

# 6. Browser-sync running (optional — recommended)
pgrep -f browser-sync && echo "OK: browser-sync running" || echo "WARN: browser-sync not running — notes will use localStorage only"
```

All `OK` results: proceed to Step 1.
Any `FAIL` result: stop. Resolve the failure (missing file, unwritable path) before continuing.
`WARN` on browser-sync: continue — the HTML falls back to `localStorage`. Inform the user.

---

## Step-by-Step Dispatch Sequence

### Step 1: Pre-render Copy (with Notes Reset)

The notes JSON file is keyed by the rendered HTML's plan-slug (derived from `document.title` in the shell). If a prior render exists, its notes persist across re-renders, which leaks reviewer state from a previous version. Clear the notes JSON before re-rendering to ensure a clean state.

```bash
# Derive plan-slug from output filename (matches the shell's document.title slugification).
# Conservative approach: remove any candidate notes JSON files whose stem matches
# the output filename (covering both slug variants the shell may produce).

rm -f "/home/claude/Projects/html-docs/notes/$(basename <output_path> .html)-notes.json"
rm -f "/home/claude/Projects/html-docs/notes/$(basename <output_path>)-notes.json"
rm -f "/home/claude/Projects/html-docs/notes/design-review-dramaturg-html-template-notes.json"
```

Then perform the template copy:

```bash
cp <kit_dir>/dramaturg-doc-template.html <output_path>
```

Verify:
```bash
test -f <output_path> && echo "OK: file exists" || echo "FAIL: copy failed"
tail -1 <output_path>   # expect: </html>
```

If `tail -1` does not show `</html>`, the template file is malformed. Re-check `dramaturg-doc-template.html`. Do not proceed past Step 1 until both checks pass.

**Note on localStorage:** the shell falls back to `localStorage` when the browser-sync server is unavailable. localStorage cannot be cleared server-side; if Kyle re-renders with the server down and then comes back online, any localStorage-persisted notes may show alongside the (now empty) server state. He can use the sidebar's "Clear all" button to wipe localStorage in the browser. This is a known limitation — the `rm -f` above only clears server-side JSON.

**Stopping criterion:** notes JSON cleared; file exists at `<output_path>` and ends with `</html>`. Then go to Step 2.

---

### Step 2: Resolve Pass 1 Prompt Variables

Read the Pass 1 prompt template and substitute its variables with actual values.

```
Template:   <kit_dir>/agent-prompt-pass1.md
Variables to substitute:
  {{plan_path}}   → <plan_path>
  {{output_path}} → <output_path>
  {{is_phased}}   → <is_phased>   (string "true" or "false")
```

```python
prompt = open("<kit_dir>/agent-prompt-pass1.md").read()
prompt = prompt.replace("{{plan_path}}", "<plan_path>")
prompt = prompt.replace("{{output_path}}", "<output_path>")
prompt = prompt.replace("{{is_phased}}", str(<is_phased>).lower())
```

**Stopping criterion:** resolved prompt string contains no remaining `{{` sequences. If any remain, a substitution was missed — stop and fix before dispatching. Then go to Step 3.

---

### Step 3: Dispatch Pass 1

Dispatch a Sonnet subagent with the resolved prompt. Pass 1 injects the Overview (Goals) section and the Phased Roadmap section into the HTML shell.

```python
Agent(
    subagent_type="general-purpose",
    model="sonnet",
    prompt=<resolved_pass1_prompt>,
    run_in_background=True
)
```

Wait for the completion notification. Do NOT poll. Do NOT sleep. The agent makes exactly 2 Edits to `<output_path>`:
- Replaces `<!-- DRAMATURG_INJECT_OVERVIEW_BOUNDARY_DO_NOT_MODIFY -->` with rendered Goals HTML.
- Replaces `<!-- DRAMATURG_INJECT_PHASED_ROADMAP_BOUNDARY_DO_NOT_MODIFY -->` with rendered Phased Roadmap HTML (or a placeholder note if `is_phased=false`).

**Stopping criterion:** completion notification received. Then go to Step 4.

---

### Step 4: Validate Pass 1

Confirm both Pass 1 injection slots were filled and the file structure is intact.

```bash
grep -c 'DRAMATURG_INJECT_OVERVIEW_BOUNDARY_DO_NOT_MODIFY' <output_path>
# expect: 0  (slot consumed by injection)

grep -c 'DRAMATURG_INJECT_PHASED_ROADMAP_BOUNDARY_DO_NOT_MODIFY' <output_path>
# expect: 0  (slot consumed by injection — even if is_phased=false, slot is replaced with placeholder)

tail -1 <output_path>
# expect: </html>
```

**If any check fails:** see Failure Recovery — "Pass 1 fails partway." Do not proceed to Step 5 until all three checks pass.

**Stopping criterion:** all three checks pass. Then go to Step 5.

---

### Step 5: Resolve Pass 2 Prompt Variables

Read the Pass 2 prompt template and substitute its variables.

```
Template:   <kit_dir>/agent-prompt-pass2.md
Variables to substitute:
  {{plan_path}}        → <plan_path>
  {{output_path}}      → <output_path>
  {{journal_path}}     → <journal_path>
  {{is_phased}}        → <is_phased>
  {{include_journal}}  → <include_journal>
```

```python
prompt = open("<kit_dir>/agent-prompt-pass2.md").read()
prompt = prompt.replace("{{plan_path}}", "<plan_path>")
prompt = prompt.replace("{{output_path}}", "<output_path>")
prompt = prompt.replace("{{journal_path}}", "<journal_path>")
prompt = prompt.replace("{{is_phased}}", str(<is_phased>).lower())
prompt = prompt.replace("{{include_journal}}", str(<include_journal>).lower())
```

**Stopping criterion:** resolved prompt contains no remaining `{{` sequences. Then go to Step 6.

---

### Step 6: Dispatch Pass 2

Dispatch a Sonnet subagent with the resolved prompt. Pass 2 injects 5 section tabs (V1, V2, V3+, Arranger Notes, Journal), each into its own slot.

```python
Agent(
    subagent_type="general-purpose",
    model="sonnet",
    prompt=<resolved_pass2_prompt>,
    run_in_background=True
)
```

The agent makes 5 Edits to `<output_path>`, one per section. Each Edit replaces one of:

- `<!-- DRAMATURG_INJECT_V1_BOUNDARY_DO_NOT_MODIFY -->`
- `<!-- DRAMATURG_INJECT_V2_BOUNDARY_DO_NOT_MODIFY -->`
- `<!-- DRAMATURG_INJECT_V3_BOUNDARY_DO_NOT_MODIFY -->`
- `<!-- DRAMATURG_INJECT_ARRANGER_NOTES_BOUNDARY_DO_NOT_MODIFY -->`
- `<!-- DRAMATURG_INJECT_JOURNAL_BOUNDARY_DO_NOT_MODIFY -->`

with the rendered HTML for that section.

Wait for the completion notification. Do NOT poll. Do NOT sleep.

**Stopping criterion:** completion notification received. Then go to Step 7.

**Note on size:** for substantial designs (e.g., HoI with a 2260-line journal), Pass 2 may approach the 32k output cap. The prompt instructs the agent to stop after the current section if approaching 28k. If the agent stops early, see Failure Recovery — "Pass 2 mid-cap split."

---

### Step 7: Validate Pass 2

Confirm all 5 section injection slots were filled and the file structure is intact.

```bash
grep -c 'DRAMATURG_INJECT_V[1-3]_BOUNDARY_DO_NOT_MODIFY\|DRAMATURG_INJECT_ARRANGER_NOTES_BOUNDARY_DO_NOT_MODIFY\|DRAMATURG_INJECT_JOURNAL_BOUNDARY_DO_NOT_MODIFY' <output_path>
# expect: 0  (all 5 slots consumed)

tail -1 <output_path>
# expect: </html>
```

**If any check fails:** see Failure Recovery — "Pass 2 fails partway." Do not proceed to Step 8 until both checks pass.

**Stopping criterion:** both checks pass. Then go to Step 8.

---

### Step 8: Resolve Review Prompt Variables

Read the Opus review prompt template and substitute its variables.

```
Template:   <kit_dir>/agent-prompt-review.md
Variables to substitute:
  {{plan_path}}          → <plan_path>
  {{output_path}}        → <output_path>
  {{output_report_path}} → <output_report_path>
  {{journal_path}}       → <journal_path>
  {{include_journal}}    → <include_journal>
```

```python
prompt = open("<kit_dir>/agent-prompt-review.md").read()
prompt = prompt.replace("{{plan_path}}", "<plan_path>")
prompt = prompt.replace("{{output_path}}", "<output_path>")
prompt = prompt.replace("{{output_report_path}}", "<output_report_path>")
prompt = prompt.replace("{{journal_path}}", "<journal_path>")
prompt = prompt.replace("{{include_journal}}", str(<include_journal>).lower())
```

**Stopping criterion:** resolved prompt contains no remaining `{{` sequences. Then go to Step 9.

---

### Step 9: Dispatch Review (Opus)

Dispatch the Opus review teammate with the resolved prompt. This agent reads both the source markdown design doc (and journal, if included) and the rendered HTML, cross-checks render fidelity, and writes a structured fidelity report. It does NOT modify any source.

```python
Agent(
    subagent_type="general-purpose",
    prompt=<resolved_review_prompt>,
    run_in_background=True
)
```

**No `model` override.** Omitting the model field causes the agent to inherit the Opus default (opus[1m]). Do not pass `model="sonnet"` here — the review benefits from Opus's reasoning depth.

Wait for the completion notification. Do NOT poll. Do NOT sleep.

**Stopping criterion:** completion notification received. Then go to Step 10.

---

### Step 10: Validate Review

Confirm the fidelity report was written and contains findings.

```bash
test -f <output_report_path> && wc -l <output_report_path>
# expect: file exists; line count > 0 (non-empty)

grep -c 'Severity' <output_report_path>
# expect: ≥1  (at least one finding entry with a severity field,
#              or a section header with this word — confirms structured output)
```

**If `test -f` fails or file is empty:** see Failure Recovery — "Review fails."
**If `grep -c 'Severity'` returns 0:** the report may be malformed or the agent did not follow the structured format. Surface the report to the user anyway and note the formatting issue.

**Stopping criterion:** file exists, is non-empty. Then go to Step 11.

---

### Step 11: Surface to User

Provide the user with:
1. The rendered HTML path: `<output_path>`
2. The fidelity report path: `<output_report_path>`
3. A one-line summary of finding count and verdict drawn from the Opus teammate's return message.

Example summary format:
```
Render complete. HTML: <output_path>. Fidelity report: <output_report_path>.
Opus review: N finding(s) — [faithful / partial-drift / significant-drift — see report].
```

The user opens `<output_path>` in Firefox and `<output_report_path>` in their editor. They review at their own pace. No further Dramaturg action is required unless the user asks for iteration (see Optional Iteration below).

---

## Variable Substitution Mechanics

All three prompt templates (`agent-prompt-pass1.md`, `agent-prompt-pass2.md`, `agent-prompt-review.md`) use `{{double_brace}}` placeholders.

**Substitution rule:** read the template file as a plain string. Call `String.replaceAll('{{var_name}}', value)` (or equivalent) for each declared variable. Pass the result directly as the `prompt` argument to the `Agent` tool.

No templating engine, no eval, no external tool required.

**Variable matrix — quick reference:**

| Variable | Pass 1 | Pass 2 | Review |
|---|:---:|:---:|:---:|
| `{{plan_path}}` | yes | yes | yes |
| `{{output_path}}` | yes | yes | yes |
| `{{output_report_path}}` | — | — | yes |
| `{{journal_path}}` | — | yes | yes |
| `{{is_phased}}` | yes | yes | — |
| `{{include_journal}}` | — | yes | yes |

**WARNING:** prompt templates must not contain literal `{{` sequences that are not intended as substitution placeholders. Future edits to any template file must preserve this property — otherwise a substitution loop will leave stray `{{` text in the dispatched prompt.

**Check before each dispatch:** after substitution, scan the resolved prompt for any remaining `{{` substring. If found, stop — a variable was missed or a template was edited incorrectly.

---

## Failure Recovery

| Failure | What happened | Recovery |
|---|---|---|
| **Pass 1 fails partway** (agent crashes, partial Edits, wrong slot filled) | HTML output is in an inconsistent state; Edit is not idempotent on a partially-filled slot | `cp <kit_dir>/dramaturg-doc-template.html <output_path>` to restore clean substrate. Re-dispatch Pass 1 from Step 3. |
| **Pass 2 fails partway** (agent crashes mid-section, wrong slots filled, malformed output) | Same as Pass 1 partial failure | `cp <kit_dir>/dramaturg-doc-template.html <output_path>` to restore clean substrate. Re-dispatch Pass 1 (Step 3), then Pass 2 (Step 6). Cannot resume Pass 2 mid-way from a dirty file. |
| **Pass 2 mid-cap split** (agent hits output context limit before injecting all 5 sections — most likely on substantial-journal designs) | Remaining slots unfilled (visible as non-zero `grep -c` in Step 7 validation) | Dispatch Pass 2b: resolve the Pass 2 prompt again but include in the prompt a specific instruction "inject only the remaining sections: <list>". Dispatch as a separate Sonnet agent. Validate after completion. The most common split point is V1+V2+V3+Arranger Notes in Pass 2a, with Journal alone as Pass 2b. |
| **Pass 1 or Pass 2 returns hallucinated content or wrong slot replacement** | Prompt-following failure; content may be injected into wrong sections or fabricated | Treat as a pass failure. `cp` the template fresh. Re-dispatch with prompt tightening (e.g., add a stronger constraint in the resolved prompt about which slot marker to replace). If the pattern repeats on second attempt, escalate to user. |
| **Review fails** (agent crashes, no output file written, file empty) | Opus review did not complete | Re-dispatch with the same resolved review prompt (Step 9). The review is idempotent — it reads source files and overwrites the report. No `cp` of the HTML template is needed. |
| **Review reports fidelity gaps** | Render content diverges from source design or journal in one or more dimensions | Surface the full report to the user. User decides: (a) accept the render with the gaps noted, (b) have the Dramaturg re-render Pass 2 with a refined prompt for the affected sections, or (c) edit the source design doc/journal and re-invoke the full kit from Step 1. Do not resolve findings unilaterally. |
| **Cross-source inconsistency reported in review** | Design doc and journal disagree (Step H finding in review report) | Cross-source inconsistencies are a Dramaturg-specific finding type. The journal is authoritative (per review prompt §10). User decides whether to update the design doc to match the journal, update the journal entry (rare; journal is append-only), or accept the inconsistency. Do not silently resolve. |

---

## Optional Iteration After Fidelity Report

The kit does not automatically iterate. After Step 11, the user directs next action.

**If the user requests fixes based on report findings:**
- For content gaps in one or more sections: re-dispatch Pass 2 with a refined prompt that names the affected sections explicitly (e.g., "re-render V1 specifically; ensure the Tron rink visual spec card is present").
- For Goals or Phased Roadmap gaps: re-dispatch Pass 1 with a refined prompt. Remember to `cp` the template fresh first — both passes start from a clean substrate.
- For Journal-tab gaps when `include_journal=true`: re-dispatch Pass 2 with a journal-only refinement; the Journal slot can be re-filled independently if the V1/V2/V3+/Arranger Notes slots are already correct.
- After any re-dispatch, re-run the corresponding validation step and re-dispatch the Opus review (Step 9) to produce an updated fidelity report.

**If the user resolves findings out-of-band** (edits the source design doc or journal directly):
- Re-invoke the full kit from Step 1 (`cp` template fresh, then Pass 1 → Pass 2 → Review).

**Iteration discipline:** do not iterate without a user decision. Do not pre-empt the user's judgment on whether findings warrant re-render.

---

## What the Kit Does NOT Do

The following are out of scope. Do not attempt to extend the kit to cover these during a render run.

- **Mobile responsiveness:** desktop-only (Firefox, 4K). No responsive breakpoints in the HTML shell.
- **Multi-reviewer collaboration:** the JSON persistence file supports one reviewer identity as metadata. It is not a collaboration channel.
- **Real-time updates:** no websockets, no concurrent editing, no live push from Dramaturg to browser.
- **Diff against prior design versions:** the kit renders a single design at a single point in time. No version history or diff view.
- **Generating new design content:** the kit RENDERS existing design+journal content. It does not author, summarize, or augment beyond markdown → HTML element mapping.
- **Resolving cross-source inconsistencies:** the review FLAGS design-vs-journal divergences. Resolving them is the team lead's call, executed by editing source files.

---

## Browser-Sync Setup (Reviewer Side — Not Dramaturg Side)

The Dramaturg does not start or manage the browser-sync server. This is a user-side concern.

**To start the server:**
```bash
~/Projects/html-docs/start-server.sh
```

This serves the rendered HTML with hot-reload and connects the `/api/notes` endpoint via notes-middleware.js.

**Notes JSON storage location:** `~/Projects/html-docs/notes/` (middleware default). File name: `<plan-slug>-notes.json`.

**Without the server:** the HTML's JavaScript falls back to `localStorage` using the key `<plan-slug>-notes`. Notes persist across page reloads in the same browser. When the server becomes available again, the next save writes to both localStorage and the server; on next page load, the server copy takes precedence over localStorage.

**Browser requirement:** Firefox on a 4K desktop display. The HTML shell is not tested in other environments.

---

## Quick-Reference Checklist

For a repeat run, verify and execute in this order:

- [ ] Collect all six inputs (`plan_path`, `journal_path`, `output_path`, `output_report_path`, `is_phased`, `include_journal`)
- [ ] Run prerequisite checks — all pass
- [ ] Step 1 (pre-flight): `rm -f` notes JSON for this plan-slug (3× variants) — clear prior reviewer state
- [ ] Step 1 (copy): `cp` template → verify file + `</html>`
- [ ] Step 2: resolve Pass 1 prompt — no `{{` remaining
- [ ] Step 3: dispatch Pass 1 (Sonnet, background)
- [ ] Step 4: validate Pass 1 — 2× grep returns 0, `tail -1` returns `</html>`
- [ ] Step 5: resolve Pass 2 prompt — no `{{` remaining
- [ ] Step 6: dispatch Pass 2 (Sonnet, background)
- [ ] Step 7: validate Pass 2 — grep returns 0 (combined pattern), `tail -1` returns `</html>`
- [ ] Step 8: resolve Review prompt — no `{{` remaining
- [ ] Step 9: dispatch Review (no model override — Opus default)
- [ ] Step 10: validate Review — file exists, non-empty, ≥1 `Severity` match
- [ ] Step 11: surface both paths + one-line verdict to user
