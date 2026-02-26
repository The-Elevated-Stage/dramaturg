# Dramaturg Skill — Consolidated Review Findings

**Date:** 2026-02-23
**Reviewers:** 14 teammates (12 Opus, 2 Haiku) across 6 review categories
**Source files:** `temp/dramaturg-review/01-*` through `06-*`

---

## How to Read This Document

- Findings are deduplicated across reviewers and grouped by severity
- **[Reviewers: ...]** shows which teammates independently flagged the finding — convergence = stronger signal
- **File references** point to the skill source, not the review files
- Each finding includes a concrete recommendation

---

## CRITICAL (12)

### C1. Vision changes only formally detected in Phase 5 — no Phase 4 → Phase 2 routing

[Reviewers: flow-tracing, vision-advocate, skillmd-haiku]

The skill has well-defined Phase 5 → Phase 2 (vision regression) and Phase 5 → Phase 4 (approach change) routing, but NO routing in the other direction. Vision stability is assumed after Phase 2 exits. Users can and do change what they want during Phase 3 scoping and Phase 4 approach exploration.

**Specific gaps:**
- No vision change detection mechanism in Phase 4 (approach-loop.md)
- No Phase 4 → Phase 2 routing in the Phase Registry
- Scope contraction ("actually I just need file sharing, not collaboration") during Phase 4 has no protocol
- The ripple assessment only exists as a Phase 5 → Phase 4 re-entry and cannot be triggered within Phase 4 itself

**Recommendation:** Add Phase 4 → Phase 2 routing to the Phase Registry. Add a vision-change detection heuristic to approach-loop.md (simpler than the Review Loop's three-question test — perhaps: "If the user's stated goal or scope changes fundamentally during approach exploration, confirm with the user and route to Phase Registry → Vision Loop"). This also covers scope contraction.

---

### C2. Architectural shift during Phase 4 invalidates multiple settled topics — no protocol

[Reviewers: flow-tracing]

When a foundational assumption changes mid-Phase-4 (e.g., client-server → peer-to-peer) and invalidates multiple previously-settled topics, there is no handling. The infeasibility pivot handles single-topic pivots. The ripple assessment handles Phase 5 → Phase 4 cascading changes. But nothing handles "the foundation under ALL topics just changed" from within Phase 4.

**Recommendation:** Add a "foundational shift" protocol to approach-loop.md. When research or discussion reveals a change that invalidates the basis for multiple settled topics: (1) stop forward progress, (2) present the foundational shift to the user, (3) identify all affected settled topics, (4) re-enter topic exploration for affected topics in dependency order. This is essentially bringing the ripple assessment concept into Phase 4's normal flow.

---

### C3. Selective reference file reading is unenforceable — erodes trust in mandatory rules

[Reviewers: skillmd-opus, skillmd-haiku, overall-opus]

SKILL.md line 33 mandates: "Never load a full reference file." This asks Claude to perform a two-step read (sections index first, then targeted section) that doesn't match how the Read tool works. Claude will always read the full file. The real risk isn't context waste — it's that an unenforceable mandatory rule trains Claude to treat all mandatory rules as aspirational, undermining rules that actually matter.

**Recommendation:** Remove the selective-read mandatory rule. Accept that reference files will be read in full (they're well-sized). Reframe it as guidance: "Focus on the section relevant to your current phase; other sections provide context but are not your current concern." This preserves the intent without an unenforceable mandate.

---

### C4. MEMORY.md research breadcrumb system is fragile and will be skipped

[Reviewers: skillmd-opus, skillmd-haiku, overall-opus]

The 4-step breadcrumb protocol (write → research → read → delete) is the single most likely thing Claude will skip. It requires 4-6 tool calls for bookkeeping, the benefit is only visible when context is lost (which usually doesn't happen), and Claude will rationalize skipping it. Stale breadcrumbs from skipped deletions or session death will pollute MEMORY.md.

**Recommendation:** Either (a) replace with journal-based position tracking (lower friction since the journal is already being written to), (b) only mandate breadcrumbs for subagent-delegated research that takes multiple turns, or (c) keep the current system but add: breadcrumb write is the FIRST action before formulating the research query, and add a "stale breadcrumb on session start" recovery note.

---

### C5. Journal checkpoint enforcement is verbal only — no verification mechanism

[Reviewers: skillmd-opus, skillmd-haiku]

Claude self-reports journal compliance but nothing checks whether entries were actually written. In a long Phase 4 with 8+ topics, Claude will settle topic 3, immediately start topic 4, and realize later it never journaled topic 3.

**Recommendation:** Add a behavioral anchor to settlement-confirmation in approach-loop.md: "After writing the journal entry, confirm to the user: 'Journaled — [one-line summary].' This confirmation serves as both a checkpoint and a conversation beat that naturally separates topics." Claude won't skip a step that involves saying something to the user.

---

### C6. Journal location hardcoded to single filename — concurrent session collision

[Reviewers: skillmd-opus]

`docs/plans/designs/dramaturg-journal.md` is hardcoded. Two concurrent sessions collide. A second session before archival appends to the previous journal.

**Recommendation:** Make the journal path session/feature-specific: `docs/plans/designs/YYYY-MM-DD-<topic>-dramaturg-journal.md` matching the design doc naming pattern. Or add a mandatory rule: "If a journal file already exists, archive or rename it before beginning."

---

### C7. Archival path contradiction — design doc location vs journal destination

[Reviewers: skillmd-opus, support-phases]

SKILL.md says the design doc goes to `docs/plans/designs/`. support-phases.md says "Archive the decision journal to `docs/archive/` alongside the design doc." But the design doc is in `docs/plans/designs/`, not `docs/archive/`. The journal archival destination is also underspecified (which subdirectory of `docs/archive/`?).

**Note:** This also conflicts with the Arranger's expectation — see ARRANGER-CHANGES.md for the cross-skill resolution.

**Recommendation:** Clarify the final state explicitly. Specify exact paths for both the design doc's final location and the journal's archive location.

---

### C8. "Two artifacts" vs "Three artifacts" contradiction

[Reviewers: support-phases]

SKILL.md preamble says the Dramaturg produces **two artifacts** (design doc + journal). The `<context>` block in support-phases.md final-design-doc section lists **three artifacts** (design doc + journal + Arranger Notes appendix). The Arranger Notes is a section *within* the design doc, not a separate artifact. A Claude instance could produce an unnecessary separate file.

**Recommendation:** Align the count. Either update support-phases.md to say "two artifacts (design document including Arranger Notes appendix, and decision journal)" or update SKILL.md to clarify the Arranger Notes is part of the design doc.

---

### C9. Stray `</output>` tag at EOF of approach-loop.md

[Reviewers: approach-loop]

Line 273 has a `</output>` after the closing `</skill>` tag — a copy-paste artifact that could cause parse errors.

**Recommendation:** Delete line 273.

---

### C10. Vision change during Phase 5 — recovery path is nearly a full restart

[Reviewers: flow-tracing]

The Phase 5 → Phase 2 → 3 → 4 → 5 → 6 → 7 redo path after a vision change is the most expensive flow in the skill. Ambiguity about what gets re-explored vs. retained. "Previously approved sections remain in place but will be re-evaluated during Reconciliation" — but Reconciliation isn't designed to handle sections built for a completely different vision.

**Recommendation:** Add guidance for the scope of re-exploration after a vision change: "When returning from a vision change, the new Topic Map determines what's explored. Topics that existed in both old and new maps but whose approaches are still valid need only re-confirmation, not full re-exploration. Topics that are new or fundamentally affected by the vision change need full exploration. Old journal entries are preserved (append-only) but new entries supersede them."

---

### C11. Elicitation without enrichment — the skill draws out what the user knows but doesn't add what they don't know but should

[Reviewers: vision-advocate, vision-prober-1, vision-prober-2]

**Strongest convergence across all vision team reviewers. Both probers independently rated this CRITICAL. This is the single most impactful design question from the review.**

The skill's philosophy is "capture the user's vision from their own words" — gate verification requires answers to come "strictly from what the user has said." This is pure **elicitation** (drawing out what the user knows). What's missing is **enrichment** (adding what the user doesn't know but should). For most feature requests, elicitation is sufficient. For security-critical features (login pages), accessibility-critical features (data tables), and architecturally significant features (i18n), elicitation alone is dangerous.

**Specific gaps:**
- Accessibility is completely absent from the entire skill (advocate's exact words)
- Security concerns only caught in Phase 5 Review Loop (4 categories), not during Vision Loop
- No i18n, compliance, or performance prompts during vision capture
- No domain-specific probes — all surfacing relies on LLM general knowledge

**Three partial compensating mechanisms exist:**
1. Research During Vision Gathering catches feasibility issues
2. Phase 5 security lens catches PII/auth/data-storage/external-service
3. Gemini research in Phase 4 may incidentally surface domain concerns

None of these constitute systematic enrichment during vision capture.

**Recommendation:** Add category-triggered enrichment prompts to the Vision Loop or Broad Design Scoping. When the feature domain is identified, prompt for relevant cross-cutting concerns:
- Authentication/authorization → security concerns (rate limiting, CSRF, session management)
- UI components → accessibility concerns (ARIA, keyboard nav, screen readers)
- Data-heavy features → performance/scalability (pagination, indexing, caching)
- User-facing features → i18n/l10n
This converts fragile LLM judgment into reliable procedural checks.

**Note:** This finding was previously listed as I1 in the IMPORTANT section. Promoted to CRITICAL based on cross-reviewer convergence and prober severity ratings. The original I1 entry has been replaced with a reference to this finding.

---

### C12. Synthesis quality — converting conversation to structured vision document is unguided

[Reviewers: vision-prober-2]

**Rated CRITICAL by prober-2 as their strongest unique contribution.** The skill must convert messy conversational input into structured output while preserving fidelity. Specific challenges: distinguishing decisions from speculation ("we need circular dependency detection" vs "maybe a chain icon"), extracting implicit requirements (DAG visualization implies graph data structure), handling user self-corrections, and choosing the appropriate abstraction level (capture intent like "indicate blocked status visually," not casually-discussed specifics like "chain icon").

No guidance exists in support-phases.md §final-design-doc for how to handle these synthesis challenges. The journal's structured entries partially address this (settled decisions are clearly marked), but the design doc compilation step — where conversation becomes document — has no equivalent guidance. Synthesis quality will vary wildly across sessions without it.

**Recommendation:** Add output structure guidance to §final-design-doc: the design doc should distinguish confirmed user decisions from design-level inferences, note open questions explicitly, and capture implied requirements separately from stated requirements. Consider adding a "Design Inferences" section to the design doc template for requirements the Dramaturg derived from conversation that the user didn't explicitly state.

---

## IMPORTANT (27)

### I1. *(Promoted to C11 — see CRITICAL section)*

Cross-cutting concerns / elicitation without enrichment. Both probers rated this CRITICAL; see C11 for the full finding and recommendation.

---

### I2. Three-question gate breaks for non-feature work

[Reviewers: vision-advocate]

"How-used" is meaningless for system qualities (error handling, performance optimization), quality initiatives (test coverage), and infrastructure work. The gate assumes user-facing features.

**Recommendation:** Either (a) explicitly scope the skill to features only and add guidance for recommending alternative approaches for non-feature work, or (b) add an adapted gate for non-feature work: "What change / Why needed / How measured" (replacing How-used with How-measured for quality/infrastructure work).

---

### I3. No fast-path for users with complete, detailed visions

[Reviewers: vision-loop, vision-advocate, vision-prober-2]

Users who arrive with a comprehensive What/Why/How-used already articulated get the same probing process as users with vague ideas. No abbreviated mode, no time-awareness, no graceful exit protocol.

**Recommendation:** Add a fast-path note to vision-loop.md question-strategy: "If the user provides comprehensive What/Why/How-used upfront, verify rather than expand — present the summary and confirm rather than asking additional exploratory questions. The Vision Loop can be as short as one exchange if the user's input is dense and clear."

---

### I4. No decomposition protocol for multi-feature requests

[Reviewers: vision-advocate, vision-prober-2]

"Add payment processing" is 6 features in a trenchcoat. The skill always produces one design document with no mechanism for recognizing when a request should be split into multiple design sessions.

**Recommendation:** Add a complexity threshold check to Broad Design Scoping. If the Topic Map exceeds a threshold (e.g., 10+ topics spanning 3+ distinct systems), the Dramaturg should present the option of decomposing into multiple focused design sessions with clear boundaries.

---

### I5. No design tension concept for contradictory requirements

[Reviewers: vision-advocate]

Everything gets "settled" or "decided." The journal has goals, use-cases, and decisions — all assuming resolution. Real design involves tensions and tradeoffs that coexist productively ("offline AND real-time" is solvable but is a design tension).

**Recommendation:** Add a "tension" journal entry type or a "Tensions" field to existing entry types. Allow the Dramaturg to document requirements that exist in productive conflict without forcing premature resolution. The Arranger receives this as design context.

---

### I6. One-question-at-a-time rule will erode — exception clause too broad

[Reviewers: skillmd-opus]

The exception ("tight, bounded questions expecting short answers may be batched") is broad enough that Claude will gradually expand its interpretation mid-session.

**Recommendation:** Add a concrete test to the exception: "If any question in the batch could produce a multi-paragraph response, it cannot be batched." Apply this tightest definition to the mandatory-rules version (line 27) since it's read first.

---

### I7. Scope protection lacks gray-zone guidance for Phase 4

[Reviewers: skillmd-opus]

"Use SQLite or PostgreSQL?" is architecture. "Use WAL mode?" is gray zone. "What columns does the table need?" could go either way. No guidance for the boundary.

**Recommendation:** Add a judgment heuristic: "The test is whether the detail constrains future implementation choices unnecessarily. 'The data model needs a location field with coordinates' is design. 'Use a REAL column named lat' is implementation. When uncertain, err toward including it — the Arranger can abstract back up more easily than it can infer missing design intent."

---

### I8. "Proceed to Phase Registry" needs a concrete re-read action

[Reviewers: skillmd-opus]

"Return to this section" is conceptually right but mechanistically vague. Claude will skip the re-read and go straight to the reference file, defeating the Phase Registry's re-bootstrap purpose.

**Recommendation:** Make it concrete: "When transitioning, re-read the Phase Registry entry for the target phase in SKILL.md. This is a literal instruction — use the Read tool on SKILL.md — because the framing and constraints may have compacted from context."

---

### I9. Conversation style section buries mandatory rules in style guidance

[Reviewers: skillmd-opus]

§conversation-style contains critical behavioral rules (one-question, scope protection, output length) mixed with tone guidance. Claude will skim the whole section on re-reads. Scope protection — the most important rule — is the last subsection.

**Recommendation:** Reorder §conversation-style to put mandatory behavioral rules first and style guidance after, or move the mandatory rules to §mandatory-rules.

---

### I10. Context Grounding has no concrete exit criteria

[Reviewers: support-phases, overall-haiku]

Purely judgment-based exit, unlike the Vision Loop's strict three-question gate.

**Recommendation:** Add a minimal checklist: "Exit when you can describe: (1) the project's tech stack and primary patterns, (2) existing features related to the user's topic, (3) key integration points the design might touch."

---

### I11. Phase numbering used throughout but never formally defined

[Reviewers: support-phases, skillmd-opus, overall-opus]

The Phase Registry uses names without numbers, but journal templates and all reference files use "Phase 4 — Approach Loop" etc. The 1-7 mapping must be inferred.

**Recommendation:** Add explicit numbers to the Phase Registry table entries.

---

### I12. Ripple assessment re-entry routing is ambiguous

[Reviewers: skillmd-haiku, approach-loop]

When the Review Loop routes to the Approach Loop for a ripple assessment, SKILL.md points to the full approach-loop.md. A Claude instance arriving there starts at §topic-selection (the natural first section) instead of §ripple-assessment.

**Recommendation:** Add conditional entry point guidance to the Phase Registry: "If re-entry is from Review Loop for ripple assessment: Read approach-loop.md §ripple-assessment first. After ripple assessment completes, proceed to §research-discuss-settle with the affected topics. Do not start with §topic-selection."

---

### I13. Cascading ripples (ripple causing further ripples) unhandled

[Reviewers: flow-tracing]

The ripple set is fixed at assessment time and never expanded. If re-settling a ripple topic causes further ripples, there's no protocol.

**Recommendation:** Add to §ripple-assessment: "After re-settling each topic, check whether the new settlement affects other topics not in the original ripple set. If so, expand the ripple set before proceeding."

---

### I14. Multiple approach changes compound context cost — no resumption guidance

[Reviewers: flow-tracing]

Each approach change triggers a full Review Loop → Approach Loop → Review Loop round trip. With 3+ changes, the Dramaturg bounces between phases 6+ times. No guidance on review resumption point after return.

**Recommendation:** Add to review-loop.md approach-change-routing: "On return from the Approach Loop, re-derive the section breakdown from updated settlements and resume section review from the first affected section. Already-approved sections not in the ripple set retain their approval."

---

### I15. SKILL.md implies ripple assessment lives in review-loop.md — it's in approach-loop.md

[Reviewers: review-loop]

SKILL.md Phase Registry says review-loop.md "contains the ripple assessment protocol." It actually contains detection/routing — the execution protocol is in approach-loop.md.

**Recommendation:** Update SKILL.md's Review Loop description: "detection criteria for approach changes (with routing to the Approach Loop's ripple assessment protocol)."

---

### I16. Missing guidance on deriving sections from settled topics (Phase 4 → Phase 5 mapping)

[Reviewers: review-loop]

approach-loop.md notes topics don't map 1:1 to sections. review-loop.md never explains how to construct sections from the Topic Map.

**Recommendation:** Add to review-loop.md §section-presentation: "Before presenting the first section, establish the section breakdown: how settled topics map to design sections. Present this mapping to the user for confirmation."

---

### I17. No handling for partial section approval

[Reviewers: review-loop]

User approves part of a section but defers another part. No protocol.

**Recommendation:** Add: "If a user approves parts but defers others, treat deferred parts as pending detail feedback. Do not write Section Approval journal entry until the full section is settled."

---

### I18. Reconciliation → Review Loop → Approach Loop cycle has no circuit breaker

[Reviewers: support-phases]

Theoretical infinite cycle with no explicit limit.

**Recommendation:** Add soft limit: "If reconciliation routes to Review Loop more than twice, present the underlying tension to the user explicitly rather than continuing to iterate."

---

### I19. Journal archival destination underspecified

[Reviewers: support-phases]

"Archive to `docs/archive/`" — which subdirectory? "Alongside the archived design" — the design doc isn't archived, it stays in `docs/plans/designs/`.

**Recommendation:** Specify exact archive path. Clarify design doc stays in place; journal goes to specified archive location.

---

### I20. Subagent type selection gap in research-strategy.md

[Reviewers: research-strategy]

No guidance on choosing between Explore subagents (codebase) and general-purpose subagents (external research). Claude might default to Explore for external research.

**Recommendation:** Add distinction: Explore subagents for codebase understanding; general-purpose subagents for multi-source external research and synthesis.

---

### I21. Conflict resolution step ordering backwards in research-strategy.md

[Reviewers: research-strategy]

Step 3 (thorough web search) comes before Step 4 (quick Gemini check). Should be quick-first escalation.

**Recommendation:** Reorder: (1) Detect, (2) Present to user, (3) Quick constrained Gemini check, (4) For difficult conflicts, search for real user experiences, (5) Journal.

---

### I22. No research depth calibration by decision importance

[Reviewers: research-strategy, vision-prober-1]

All research treated equally. Foundational architectural decisions deserve significantly more investment than peripheral design details.

**Recommendation:** Add calibration note: "Research depth proportional to architectural impact. Foundational decisions (data model, core patterns) → exhaustive. Peripheral decisions (UI preferences) → quick feasibility check."

---

### I23. Effort mismatch detection absent — no mechanism for recognizing when user underestimates scope

[Reviewers: vision-advocate, vision-prober-1]

The skill has no mechanism for recognizing when user perception of effort ("quick cleanup") drastically mismatches actual effort ("multi-week rewrite with migration"). Context Grounding is "quick and focused" by design and scope protection redirects implementation-depth analysis to the Arranger. For refactoring/restructuring requests (e.g., "refactor the settings page"), the true scope would only be discovered by the Arranger — late and wasteful.

**Recommendation:** Allow Context Grounding to go deeper for refactoring/restructuring requests where the user's framing may understate complexity. Add guidance: "When the user's request involves restructuring or cleaning up existing code, Context Grounding should assess the complexity of the existing implementation. If codebase analysis reveals significantly more complexity than the user's description implies, surface this finding during the Vision Loop."

---

### I24. Context pressure on deep features — post-split journal lacks reasoning context for ripple assessment

[Reviewers: flow-tracing]

For very complex features (8+ topics, extensive research), even with session splitting at Phase 4→5, the journal captures *what* was decided but not the full *reasoning context*. A post-split Phase 5 session encountering an approach change must execute a ripple assessment that needs to understand *why* topics were settled, not just the journal summaries. The journal's "Alternatives discussed" and "Findings" fields help, but they're summaries, not full context.

**Recommendation:** Add to approach-loop.md §session-split: "For complex features, ensure journal entries are detailed enough for a fresh session to understand the reasoning behind settlements — not just the outcome. If a topic required extensive debate or had closely competing alternatives, document the key differentiators that tipped the decision."

---

### I25. Long discussion context consumption — no mechanism to manage pre-design phases

[Reviewers: flow-tracing]

Phases 2-3 have no context management mechanism. Users who explore extensively before committing (extended vision discussion, detailed scoping questions) can exhaust context before reaching Phase 4 where the core design value is generated. The Vision Loop has no upper bound on duration, and while Broad Design Scoping has a 6-question cap, user answers and follow-up clarifications are unbounded.

**Recommendation:** Add guidance to SKILL.md §conversation-style or §mandatory-rules: "If context pressure builds during Phase 2 or 3, the Dramaturg may suggest capturing progress: 'We've covered a lot of ground — let me write the Vision Baseline now and we can continue refining in the next phase.'"

---

### I26. SKILL.md reference pointer omits "research during vision gathering" from vision-loop.md description

[Reviewers: vision-loop]

SKILL.md's reference pointer for vision-loop.md lists "Question strategy, gate verification, vision baseline journal template, loop-back conditions" but omits the "Research During Vision Gathering" sub-section — a substantial and critical portion of the file. A Claude instance reading the pointer during selective loading might not expect to find research guidance in this file and could skip it.

**Recommendation:** Update the reference pointer description to include "research during vision gathering" or "feasibility validation during questioning."

---

### I27. Approach Loop exit lacks explicit completion criteria

[Reviewers: overall-haiku]

The Approach Loop's exit section lists verification checks but does not explicitly state *when* to transition, unlike the Vision Loop's strict gate ("Move to Broad Design Scoping ONLY when..."). A lengthy Phase 4 might have 15 settled topics, and the Dramaturg could reasonably wonder: "Is this enough? Should I keep going?"

**Recommendation:** Revise approach-loop.md §exit to be explicit: "Phase 4 is complete when ALL of the following are true: (1) Every topic from the Topic Map has been explored and settled, (2) Every settled topic has a journal entry, (3) The session split evaluation has been considered. When all three are true, proceed to Phase Registry → Review Loop."

---

### I28. Vision gate enforcement too soft — judgment calls enable skipping feasibility research

[Reviewers: skillmd-haiku]

The mandatory research-during-vision rule is clear in principle, but the judgment calls in vision-loop.md §question-strategy (about when to defer vs. research immediately) are broad enough that a lazy instance can rationalize skipping feasibility research. Distinct from C11 (which is about *what additional concerns* to surface) — this finding is about enforcement of *existing* research rules.

**Recommendation:** Strengthen the decision rule in vision-loop.md §question-strategy from guidance to mandatory: "When the user's vision includes feasibility-dependent technical assumptions (specific positioning methods, real-time sync, platform capabilities, offline behavior), research them immediately before continuing. Defer only well-understood capabilities. When uncertain, research."

---

## MINOR (32)

### M1. "Repetiteur" stale name in design doc — should be "Arranger"
[Reviewers: overall-opus]

### M2. Journal format: design doc says Tier 3 with `<journal>` wrapper, templates use plain markdown
[Reviewers: overall-opus]

### M3. Selective section loading needs re-entry hints in Phase Registry
[Reviewers: overall-opus]
Recommendation: Add section-level loading hints to Phase Registry entries for re-entry cases.

### M4. MEMORY.md breadcrumb stale on session death — needs recovery note
[Reviewers: overall-opus]
Recommendation: Add one sentence: "If a session starts with an existing Research Diversion entry, re-execute the research and delete the stale entry."

### M5. Duplicate exit section in vision-loop.md adds no value
[Reviewers: overall-opus, vision-loop, skillmd-opus]

### M6. research-strategy.md lacks self-documenting loading boundary markers
[Reviewers: overall-opus]
Recommendation: Add note at top indicating which sections load on first diversion vs. conditionally.

### M7. No guidance on sequential vs nested research diversions
[Reviewers: overall-opus]
Recommendation: Add: "Research diversions are sequential, not nested."

### M8. Example questions in vision-loop.md violate the one-at-a-time mandate
[Reviewers: vision-loop]
Line 20 asks "what does that include? What's in and what's out?" — two questions.

### M9. Two overlapping mandatory blocks for research-during-vision should be consolidated
[Reviewers: vision-loop]
Lines 50 and 52 cover the same ground. Merge into one stronger block.

### M10. "Re-verify" after user correction is underspecified
[Reviewers: vision-loop]
Clarify: corrections trigger full re-presentation of updated summary, not just acknowledgment.

### M11. Context Grounding loop-back has no position-tracking mechanism
[Reviewers: vision-loop]
Unlike research diversions (MEMORY.md breadcrumbs), no way to remember where in the vision discussion.

### M12. Re-do of Broad Design Scoping after vision change should be mandatory-tagged
[Reviewers: vision-loop]
Line 138 is important enough to warrant `<mandatory>`.

### M13. Vision Baseline "Status: settled" — no status values enumerated anywhere
[Reviewers: vision-loop]

### M14. Redundant mandatory rules duplicated from SKILL.md in approach-loop.md
[Reviewers: approach-loop]
Lines 47, 60, 68. Creates maintenance risk. Consider adding "Restated for emphasis" note.

### M15. Missing guidance for trivially-settled topics
[Reviewers: approach-loop]
Not every topic needs full research. Add fast-path for topics matching established project patterns.

### M16. Topic Map reference is implicit in approach-loop.md §topic-selection
[Reviewers: approach-loop]
Add explicit: "Read the Topic Map journal entry from the decision journal before beginning topic selection."

### M17. Journal templates missing "Supersedes" field for ripple re-settlement
[Reviewers: approach-loop]
The ripple assessment mandates supersession references but no template field exists.

### M18. Infeasibility findings aren't journaled
[Reviewers: approach-loop]
An infeasible approach is NOT settled, so it falls outside the "every settled topic" journal rule. But the Arranger benefits from knowing what was ruled out.

### M19. Research protocol partially duplicated inline in approach-loop.md
[Reviewers: approach-loop]
Line 49 has a condensed version alongside a pointer to the full version. Frame as reminder or remove.

### M20. Approach Loop exit doesn't handle session-split path
[Reviewers: approach-loop]
If session splits, who "proceeds to Review Loop"?

### M21. Approach Loop → Review Loop return from ripple doesn't specify which sections need re-review
[Reviewers: review-loop, flow-tracing]

### M22. "Remain in place" is ambiguous in vision regression routing
[Reviewers: review-loop]
Clarify: "Section Approval journal entries preserved. Reconciliation re-evaluates against updated vision."

### M23. Security lens "address it naturally" is abstract
[Reviewers: review-loop]
Add: "Specify security-relevant requirements (what data retained, who accesses), not implementation mechanisms."

### M24. No explicit definition of what constitutes user approval
[Reviewers: review-loop]
Add: "Approval is any clear positive signal. If ambiguous, confirm: 'Are you happy with this section as-is?'"

### M25. Topic Map "Status: settled" is misleading — topics haven't been explored yet
[Reviewers: support-phases]
Use "confirmed" instead of "settled" for Topic Map status.

### M26. No explicit instruction for when user says topic coverage is incomplete
[Reviewers: support-phases]
Add: "If user identifies missing topics, add them and re-confirm."

### M27. Degraded Gemini performance not covered — only total failure
[Reviewers: research-strategy]
Add handling for intermittent issues, low-quality results, suspiciously generic responses.

### M28. Scope expansion not proactively surfaced to the user
[Reviewers: flow-tracing]
Scope grows organically but the Dramaturg doesn't flag how much it's grown vs. the original ask.

### M29. Section ordering in Review Loop should be mandatory, not core
[Reviewers: skillmd-haiku]
review-loop.md §section-presentation's section ordering guidance ("foundations first") uses `<core>` not `<mandatory>`. A lazy instance could present sections in any order — potentially approving UI behavior before the data model is settled, causing rework during Reconciliation.

**Recommendation:** Upgrade the section ordering instruction to `<mandatory>`.

### M30. One-section-at-a-time rule has checkability gap for large sections
[Reviewers: skillmd-haiku]
"One section per message" (review-loop.md §section-presentation) conflicts with "never truncate or compress reasoning" (SKILL.md §mandatory-rules) when a section is too large for one message. No guidance for resolving the tension.

**Recommendation:** Clarify: "One logical section per turn. If a section is complex and spans multiple messages, that is one section reviewed across multiple turns. The constraint is that the user sees one section at a time, not that one section must fit in a single message."

### M31. Vision regression detection has false positive risk from memory reconstruction
[Reviewers: skillmd-haiku]
If context compacts between the Vision Loop and Review Loop, the Dramaturg may reconstruct the Vision Baseline from conversation memory instead of reading the journal entry, leading to false-positive regression detections.

**Recommendation:** Add to review-loop.md §vision-regression: "Before testing for vision regression: retrieve the Vision Baseline journal entry from the decision journal. If the entry is missing or unclear, ask the user to clarify the original baseline rather than reconstructing from memory."

### M32. Reference file load instructions inconsistent across phases
[Reviewers: skillmd-opus]
The Approach Loop and Review Loop reference tags in SKILL.md describe file contents extensively but don't specify section restrictions, unlike Context Grounding and Broad Design Scoping which say "Read §X only." This effectively says "read the whole file," contradicting the selective-reading mandatory rule (C3).

**Recommendation:** Either add section restrictions consistently to all reference tags, or acknowledge that approach-loop.md and review-loop.md are intended to be read in full.

---

## SUGGESTIONS (18)

### S1. Add session bootstrap/resumption protocol
[Reviewers: overall-opus, skillmd-opus]
What a new session should do: read journal → identify current phase → verify completeness → summarize for user.

### S2. README should clarify design doc vs SKILL.md authority hierarchy
[Reviewers: overall-opus]

### S3. "Co-visionary" framing could be more explicit in SKILL.md preamble
[Reviewers: overall-opus]

### S4. Add "Next Steps" note at end of Phase 7 mentioning Arranger invocation
[Reviewers: arranger-mapping]

### S5. Vision Baseline template missing research annotation field
[Reviewers: skillmd-opus]
Add: `**Research during vision:** [feasibility checks and results, or "none"]`

### S6. No guidance for user wanting to skip phases
[Reviewers: skillmd-opus]
Add: "Vision Loop gate is non-negotiable. Other phases can be abbreviated if user has done the thinking."

### S7. No error recovery for journal file corruption
[Reviewers: skillmd-opus]
Add: "If journal missing or corrupted, reconstruct from conversation immediately."

### S8. Cross-reference from reference files back to SKILL.md for session-split cases
[Reviewers: skillmd-opus]
Add to session-split section: "A fresh session should load SKILL.md first."

### S9. Sections list in review-loop.md could benefit from grouping comments
[Reviewers: review-loop]

### S10. Exit completeness check should verify Topic Map coverage
[Reviewers: review-loop]

### S11. Loop-back tracking — detect when same phase loop-back occurs twice
[Reviewers: skillmd-haiku]

### S12. Reconciliation recovery path if no Section Approval entries exist
[Reviewers: skillmd-haiku]

### S13. Infeasibility pivot — guidance for when ALL alternatives are infeasible
[Reviewers: skillmd-haiku, flow-tracing]
Escalate to vision change.

### S14. Verbatim capture — guidance for long user statements
[Reviewers: skillmd-haiku, approach-loop]
Capture the design-relevant sentence, not full transcription.

### S15. General-purpose subagents missing from tool-hierarchy section
[Reviewers: research-strategy]

### S16. "Web Wins" heading imprecise — "Ground Truth Wins" better
[Reviewers: research-strategy]

### S17. Define UNRESEARCHED as third Arranger note status
[Reviewers: research-strategy]
For decisions made without any research backing.

### S18. Category-triggered checklists for cross-cutting concerns
[Reviewers: vision-prober-2]
The strongest recommendation from the vision stress test: convert fragile LLM judgment into reliable procedural checks.

---

## Positive Observations

Multiple reviewers noted these as particularly strong:

1. **Phase Registry hub-and-spoke architecture** — prevents spaghetti routing between reference files
2. **Research-during-vision-gathering mandate** — the skill's strongest mechanism
3. **"Drawing Out Why" question strategy** — purpose-built for discovering underlying needs
4. **Feedback classification system** (vision/approach/detail/approved) — clear taxonomy with routing
5. **Reconciliation workflow** — the strongest section in support-phases.md
6. **VERIFIED/PARTIAL Arranger note mechanism** — well-designed cross-skill optimization
7. **Reinforcement principle** — mandatory rules consistently restated at point of use
8. **Tier 3 compliance** — all files properly tagged, no naked markdown
9. **Design doc faithfully implements the design** — minimal drift

---

## Cross-Reviewer Convergence (Highest Confidence Findings)

These findings were independently identified by 3+ reviewers:

| Finding | Reviewers | Severity |
|---------|-----------|----------|
| Selective reading unenforceable | skillmd-opus, skillmd-haiku, overall-opus | CRITICAL |
| MEMORY.md breadcrumbs fragile | skillmd-opus, skillmd-haiku, overall-opus | CRITICAL |
| Vision changes only in Phase 5 | flow-tracing, advocate, skillmd-haiku | CRITICAL |
| Cross-cutting concerns / elicitation gap | advocate, prober-1, prober-2 | CRITICAL (C11) |
| Effort mismatch detection absent | advocate, prober-1 | IMPORTANT |
| Phase numbering not formally defined | support-phases, skillmd-opus, overall-opus | IMPORTANT |
| Duplicate exit in vision-loop.md | overall-opus, vision-loop, skillmd-opus | MINOR |
| Explicit mechanisms > LLM intuition | prober-1, prober-2 (independent convergence) | META |

---

## Statistics

| Severity | Count |
|----------|-------|
| CRITICAL | 12 |
| IMPORTANT | 27 |
| MINOR | 32 |
| SUGGESTION | 18 |
| **Total** | **89** |
