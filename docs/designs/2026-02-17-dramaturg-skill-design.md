# Dramaturg Skill — Design Document

**Date:** 2026-02-17
**Status:** Design Phase
**Type:** Standalone Claude Code Plugin Skill

---

## 1. Problem Statement & Motivation

Two existing planning tools each solve half the problem:

**Brainstorming** (superpowers plugin) provides fluid, collaborative design discussions — one question at a time, conversational tone, incremental design validation. But it has **zero research capability**. When it proposes "use SSE for device communication" or the user suggests "WiFi-based location," these claims go unvalidated. Recommendations come entirely from Claude's training data, which may be stale or wrong for the specific context.

**Deep-plan** (piercelamb plugin) provides thorough research — codebase exploration, web search, external LLM review. But research happens in a **single upfront pass** (steps 6-7 of a 22-step pipeline), and the subsequent interview and plan writing phases make architectural decisions without fresh research. When the interview reveals a new approach or the plan introduces a design choice, those decisions are uninformed by current reality. The rigid pipeline also prevents the natural back-and-forth that produces better designs.

**The gap:** No tool weaves research into the design discussion itself. The dramaturg skill fills this gap — it researches approaches *as they come up in conversation*, validating recommendations before presenting them and checking user proposals for feasibility in real-time.

### Why "Dramaturg"?

In theater, the dramaturg is the person who researches historical context, source material, and thematic background to inform the creative team's vision. They don't direct the play (that's the conductor). They don't perform (that's the musician). They ensure the creative vision is grounded in reality — which is exactly what this skill does for software design.

This creates a natural pipeline metaphor: **dramaturg** (researches and shapes the vision) → **arranger** (translates vision into executable plan) → **conductor** (coordinates the execution) → **musician** (implements the work). The copyist creates individual parts from the arranger's score.

---

## 2. Skill Identity

- **Name:** `dramaturg`
- **Purpose:** Research-augmented vision/design skill — explores what *could* be built through fluid discussion with active research
- **Location:** Standalone plugin, alongside conductor and musician
- **Output:** Design document only. No automatic handoff to any implementation skill. The natural next step is the **arranger** (translates the design into an executable implementation plan), but the user decides — they may also use writing-plans, deep-plan, conductor, or manual implementation.
- **Scope:** This skill is concerned with *what could be designed* — the vision, the possibilities, the feasibility. Implementation details (how to make it real in this specific codebase) are the concern of subsequent skills. This scope boundary exists because mixing vision exploration with implementation constraints kills creativity — "we can't do X because the database doesn't support it" shuts down a conversation that should be "X would be ideal, and we can figure out the database later." The dramaturg explores the ideal; the implementation planner figures out how to approximate it.

### Core Design Principles

These two principles are foundational. They appear here first and are **reinforced throughout every section where they apply** — not mentioned once and assumed remembered.

**Priority chain for technical decisions:**

**compatibility > reliability > efficiency > security > performance**

Prefer modern but not bleeding-edge approaches. Future-favoring: prefer current code, APIs, and protocols that are well-established and well-documented. When research presents multiple viable options, this chain breaks ties. When Claude recommends an approach, the recommendation should reflect this ordering — a more compatible but slightly less performant solution beats a faster but less compatible one.

**Reinforcement principle:**

Critical constraints stated once get forgotten as context grows. One or two authoritative statements DO NOT survive a long session — they get buried under research findings, discussion exchanges, and section drafts. The skill file must use constant reminders at the point of relevance: when referencing templates, when giving instructions for research, when describing phase transitions. This design document follows the same principle — mandatory constraints appear in every section where they're relevant, not just their "home" section.

### What Dramaturg Is NOT (and Why Each Boundary Exists)

- **Not a replacement for brainstorming.** Brainstorming remains appropriate for simple features that don't need research validation. Adding research overhead to "where should this button go?" wastes time. Dramaturg is for designs where *getting the approach wrong has significant downstream cost* — architectural decisions, technology choices, integration patterns. If the design wouldn't benefit from external research, use brainstorming.

- **Not a replacement for deep-plan.** Deep-plan remains appropriate when you need a full implementation plan with TDD, section splitting, and external LLM review. Dramaturg intentionally stops at the design doc because the implementation planning concern (task decomposition, test scaffolding, dependency ordering) is a different problem with different requirements. Trying to do both in one skill would recreate deep-plan's rigidity.

- **Not an arranger.** It produces a design doc, not an implementation plan. This boundary exists because the skill is about *what could be designed* — exploring possibilities, validating feasibility, shaping vision. Implementation planning requires different context (specific file paths, test infrastructure, deployment constraints) that would pollute the design discussion. The user chooses the right implementation tool after the design is done. **Scope protection:** When users push for implementation details (specific file paths, function signatures, variable names, package versions), redirect to architecture-level language. "We'll define the file structure in the implementation plan. For now, this belongs in the data layer." This is not evasion — it's preserving the design discussion's altitude. Implementation details answered prematurely become constraints that limit design exploration.

---

## 3. Workflow Phases

The workflow has 7 phases with defined loop-back points. Phases are conversational (not rigid pipeline steps), but transitions have explicit triggers.

```
1. CONTEXT GROUNDING
   Explore subagents for project architecture/scope
   Quick, focused — "what exists today"
          │
          ↓
2. VISION DISCUSSION
   One question at a time, open-ended
   Gate: Claude can answer What/Why/How-used strictly from user input
   ←── loops to 1 if vision reveals unexplored project context
          │
          ↓
3. BROAD DESIGN SCOPING
   High-level framing questions (max 6 total, max 2 per area)
   Open-ended, designed to draw out rich context
          │
          ↓
4. APPROACH DISCUSSION
   Topic-guided, research-backed exploration of approaches
   Research-discuss loop: explore options, validate, settle
   Explicitly confirms when each topic is settled
   ←── loops to research as discussion needs
          │
          ↓
5. PER-SECTION REVIEW (foundations first)
   Present detailed design sections based on settled approaches
   One section at a time for focused review
   - Vision change → back to phase 2
   - Approach change → back to phase 4
   - Detail change → revise section
   - Approved → next section
          │ (all sections approved)
          ↓
6. RECONCILIATION PASS
   Re-review all sections for cross-section consistency
   Present affected sections for re-approval
          │
          ↓
7. FINAL DESIGN DOC
   Compile all reconciled sections
   Save to docs/plans/designs/YYYY-MM-DD-<topic>-design.md
```

### Phase 1: Context Grounding

**Purpose:** Understand the current project before discussing what to build on top of it.

**Execution:** Launch Explore subagents to understand:
- Project architecture and scope
- Existing patterns and conventions
- Relevant existing features that the new design might interact with

**Decision:** This phase is quick and focused — not a deep codebase audit. The goal is "what exists today" so the vision discussion is grounded, not "analyze every file." The research execution strategy (Section 5) applies here: use Explore subagents, keep it lightweight.

**Transition:** Move to Phase 2 when Claude has enough project context to have an informed conversation. This is Claude's judgment call.

### Phase 2: Vision Discussion

**Purpose:** Understand what the user wants, why they want it, and how they'll use it — entirely from the user's own words.

**Execution:** Conversational, one question at a time, open-ended questions. Draw the user out. This is brainstorming-style dialogue — no forms, no checklists, no multi-question dumps.

**Transition gate (strict):** Claude can move to Phase 3 **only when** it can answer all three of these from user input alone:

1. **What** does the user want to build?
2. **Why** do they want to build it? (the motivation, the problem it solves)
3. **How** will they use it? (concrete use cases, user scenarios)

These answers must come strictly from what the user has said. Claude must not infer the "why" from project context or fill in use cases from its own training data. If Claude can't answer one of these, it keeps asking.

**Decision rationale:** This strict gate prevents the most common planning failure: Claude races ahead with an assumed understanding, designs something the user didn't actually want, and the entire session is wasted. The three questions are deliberately simple — if you can't answer them from user input, you don't understand the vision yet.

**Loop-back to Phase 1:** If the vision discussion reveals unexplored project context (e.g., "this needs to integrate with the notification system" and Claude hasn't explored that), loop back to Context Grounding to explore the relevant area, then return to Vision Discussion.

### Phase 3: Broad Design Scoping

**Purpose:** Establish a high-level map of the design territory before diving into detailed per-section work. This frames what sections need to exist and gives the session a shared understanding of the big picture.

**Execution:** A focused set of high-level, open-ended questions.

**Constraints (and why each limit exists):**
- **Max 2 questions per area** (backend, frontend, data model, etc.) unless the feature is particularly heavy in that area. *Why 2:* One question often isn't enough to frame an area, but three starts becoming a deep dive — which is Phase 4's job. Two questions hit the sweet spot: enough to establish direction without getting lost in details.
- **Max 6 total questions** across all areas. *Why 6:* This forces prioritization. A feature that touches backend, frontend, data model, and API design could generate 20 questions. Capping at 6 means Claude must identify the *most design-shaping* questions — the ones where the answer changes what sections need to exist. This also respects the user's time and keeps the scoping phase from becoming the interview phase.
- **Open-ended questions only** — designed to make the user talk, not answer yes/no or pick from a list. *Why:* The purpose of this phase is to draw out the user's mental model and priorities. "Should the backend use REST or GraphQL?" gets a one-word answer. "How do you see the frontend and backend communicating for this feature?" gets context about the user's assumptions, preferences, and constraints that inform the entire design.
- Not deep dives — just enough to frame the subsequent discussion sections. *Why:* Detailed exploration happens in Phase 4 with research backing. The scoping phase's value is in establishing the map, not exploring every territory on it.

**Decision rationale:** Without this step, Claude would dive into detailed per-section research without knowing the big picture. You'd research SSE deeply for device communication, then discover in a later section that the user actually wants a simpler polling approach for their use case. The scoping step surfaces these high-level decisions cheaply before investing research time.

The strict limits (max 6 questions, max 2 per area) force Claude to ask *good* questions. When you can only ask 2 backend questions, you pick the ones that most shape the design. This also encourages user verbosity — open-ended questions with limited count signal "tell me everything important about this area."

**Transition:** Move to Phase 4 when the broad shape is established. Before transitioning, Claude explicitly lists the topics it plans to explore in Phase 4 (derived from the scoping answers) and asks the user to confirm the list is complete. This list becomes the **Topic Map** entry in the decision journal (see Section 6). The topic map serves as both a coverage check ("did we miss an area?") and a recoverable checkpoint if the session is interrupted.

### Phase 4: Approach Discussion

**Purpose:** The core of the skill — explore, research, and settle on the right approach for each major design topic through fluid, research-backed conversation. This is where the big architectural decisions happen (SSE vs FCM vs refined FCM), informed by active research.

**Why this phase exists (and why it's separate from Per-Section Review):** Without this phase, Phase 5 (Per-Section Review) would be doing two jobs at once: hashing out the fundamental approach *and* reviewing the detailed writeup. This overloads section reviews — the first "review" of a section becomes a debate about the whole direction, followed by multiple rounds of feedback on the actual details. By settling approaches here first, the Per-Section Review becomes what it should be: "here's the detailed writeup of what we already agreed on — does the detail look right?"

**Real example from this project's location feature:** Broad Scoping asked "what devices will you connect to?" and "how do you see yourself using the location feature?" — establishing the territory. The Approach Discussion then explored the *actual protocol*: started with SSE, research revealed HTTP/2 issues with Express, pivoted to FCM, user refined the FCM approach. By the time the "FCM Integration" section reached Per-Section Review, the *what* was settled — only the details (message schema, delivery guarantees, background behavior) needed review.

**Execution:** Topic-guided but flexible conversation:

**Step a — Pick a topic.** Claude identifies the next design topic to explore, guided by the Broad Design Scoping output and ordered by dependency (foundations first). Topics map to the major decisions that shape the design — not to document sections (those come in Phase 5).

**Step b — Research.** Using the research execution strategy (Section 5), investigate the topic. The research trigger rules (Section 4) determine when research happens within the discussion. **Reminder: research FIRST, recommend AFTER. Never recommend based solely on training data for substantive technical decisions. Apply the priority chain (compatibility > reliability > efficiency > security > performance) when evaluating options.**

**Step c — Discuss findings.** Present research results conversationally. Discuss implications, trade-offs, and how the findings affect the approach. The user may redirect, suggest alternatives, or raise related concerns.

**Steps b↔c loop** as the discussion needs. If discussing findings raises new questions, research those. If research reveals an unexpected angle, discuss it. This loop is the heart of the skill's value — research and discussion are interleaved, not sequential. Natural tangents are allowed when topics are related (e.g., discussing the communication protocol naturally touches on error handling). **Tangent bounding:** if a tangent would be its own topic in the design document, table it rather than exploring it inline. Briefly note it ("error handling deserves its own topic — I'll add it to the list") and return to the current topic. This prevents aggregate drift where each tangent feels natural but the conversation wanders far from the original topic.

**Step b↔c — Pivot on infeasibility.** If research proves the current approach is not feasible, do not try to force it. Immediately present the infeasibility finding to the user and loop back to the start of this topic's exploration with alternatives. "Research shows [approach] won't work because [finding]. Here are the alternatives I'd recommend exploring..." The pivot is a natural part of the b↔c loop, not a failure — the skill's value is catching these before they become design flaws.

**Step d — Settle the topic.** When Claude and the user have converged on an approach, Claude explicitly confirms: "So we're going with [approach] for [reason]. Moving on?" This prevents topics from being left ambiguous. The user either confirms or clarifies, and only then does the conversation move to the next topic.

**Decision rationale (explicit settlement):** Without explicit settlement, it's unclear when a topic is "done." The user might think they were just exploring options while Claude moves on assuming a decision was made. The brief confirmation takes a second but prevents misaligned assumptions.

**Disproportionate complexity.** If research reveals that an approach is feasible but dramatically over-engineered for the use case, mention it conversationally — "this works, but it's significantly more complex than [simpler alternative] for what you need." This is not a formal gate or a loop step — just Claude being a good collaborator. The detailed cost/effort analysis is the Arranger's job, but catching obvious over-engineering early prevents wasting a full design cycle on an approach that the Arranger would later flag as impractical.

**Transition:** Move to Phase 5 when all major design topics are settled. Claude should be able to list every settled approach and its rationale before transitioning.

**Session split evaluation.** The Phase 4→5 boundary is the designed natural cut point for session management. Before transitioning to Phase 5, evaluate whether the session should split:
- **If the feature has many topics (8+) or the discussion was lengthy**, recommend the user check context usage and consider starting a fresh session for Phase 5. The decision journal preserves all settled approaches, research findings, and the vision baseline — a new session reads the journal and proceeds directly to section writing with full context available.
- **If the feature is moderate (4-6 topics)**, mention the option but don't push. "We've settled all the approaches. Want to continue to section review, or start fresh? The journal has everything we need either way."
- **If the feature is small**, continue without comment.

This is not about compressing output or rushing Phase 4 to save context — thorough Phase 4 analysis is the skill's core value. The split is about *starting Phase 5 with a clean context budget* so section writing and review get the same thoroughness that the approach discussion did.

### Phase 5: Per-Section Review

**Purpose:** Present the detailed design as structured sections for focused review. By this point, the *approaches* are already settled in Phase 4 — this phase is about the *details* within those approaches.

**Why this phase is streamlined:** The heavy lifting — approach exploration, research, pivots, debates — happened in Phase 4. The Per-Section Review presents the writeup of already-agreed approaches, so reviews should be mostly smooth sailing. Feedback here is typically detail-level ("add a note about retry backoff") rather than approach-level ("actually, let's not use FCM").

**Execution:** For each section (foundations first, Claude judges ordering):

**Present one section at a time.** Each section is a detailed writeup of a settled approach from Phase 4, including specifics like data schemas, error handling, edge cases, and interaction patterns. One section per message — never present multiple sections in a single reply.

**Decision rationale (one section at a time):** Multi-topic reviews lose detail. When a user needs to respond to 8 topics, the 2-3 that need detailed responses get lost in the volume. Single-section reviews keep the scope focused and ensure every important point gets proper attention.

**User reviews.** Four possible outcomes, determined by the feedback type:

| Feedback type | Detection | Action |
|---|---|---|
| **Vision change** | Alters the answer to what/why/how-used | Loop to Phase 2 (Vision) |
| **Approach change** | Same goal, different technical method | Loop to Phase 4 (Approach Discussion) for this topic. **Ripple assessment:** Claude identifies whether the new approach affects other settled topics. If it does, list the affected topics and get user confirmation before invalidating those sections. Unaffected sections retain their approval. |
| **Detail change** | Refinement within the current approach | Revise the section in place |
| **Approved** | User accepts the section | Move to next section |

**Section ordering:** Claude sequences sections by dependency, preferring foundations first (architecture, data model before UI behavior, error handling). The ordering is less critical here than in Phase 4's topic order, since the approaches are already settled — but foundations-first still prevents unnecessary rework if a detail change in a foundational section ripples outward.

**Security lens:** When presenting each section, Claude should briefly assess whether the section involves PII handling, authentication/authorization, data storage, or external service communication. If it does, flag the security-relevant aspects explicitly — "this section stores user location data, which is PII — the design should address data retention and access control." This is not a full security audit (that's an implementation concern), but design-level security considerations should be surfaced during review rather than discovered downstream. If no security concerns apply to a section, don't mention it — the check is silent when clean.

### Phase 6: Reconciliation Pass

**Purpose:** Ensure all approved sections are internally consistent before compiling the final document.

**Execution:**
1. List all approved sections (use decision journal section approval entries for a compact view)
2. For each section, check: did any later section's revisions affect this section's assumptions?
3. Classify each inconsistency by severity:

| Severity | Example | Action |
|---|---|---|
| **Cosmetic** | Terminology drift ("device" vs "client"), formatting differences | Fix silently — update the section without user re-approval. Note the fix in passing. |
| **Clarification** | A section references a detail that a later section made more specific (e.g., "stores location" → later section settled on "GeoJSON point") | Update the earlier section to match. Present the change to the user as an informational note, not a re-approval gate. "I updated Section A to say GeoJSON point, matching what we settled in Section D." |
| **Substantive** | A section's approach conflicts with a later decision (e.g., Section A assumed polling but Section C switched to WebSockets). The section's logic or architecture would change. | Present the conflict and the proposed revision. Require user re-approval of the affected section. |

4. Present substantive inconsistencies to the user for re-approval. Cosmetic and clarification fixes are reported but don't gate progress.
5. Only proceed to Phase 7 when all substantive inconsistencies are resolved.

**Why severity thresholds matter:** Without them, every minor terminology drift triggers a re-approval cycle. For a design with 6-8 sections, even 2-3 cosmetic fixes per section means 12-18 unnecessary approval gates. The user gets stuck in a "reconciliation loop" re-approving sections that haven't meaningfully changed. The severity classification ensures user attention is spent on real conflicts, not rubber-stamping minor updates.

**Decision rationale:** Later sections inevitably refine earlier decisions. Section A might say "the backend stores location as lat/lng" and Section D's discussion might reveal "we actually need a full GeoJSON point for future extensibility." Without reconciliation, the final doc contains contradictions. This step catches those before they become implementation bugs.

**Transition gate — Implementation readiness:** Before moving to Phase 7, the Dramaturg evaluates whether the design is complete enough to begin implementation planning. The test: *could the Arranger take this design and produce an executable implementation plan without needing to make design-level decisions?* If yes, the design is ready — the Dramaturg tells the user "the design is ready when you are" and proceeds to Phase 7 when the user confirms. If no, the Dramaturg identifies what's still unresolved and keeps probing — asking more questions, researching open areas, or revisiting sections that feel underspecified.

This is a gate, not a formality. The Dramaturg does not rush to Phase 7 because all sections are reconciled — reconciliation ensures consistency, but consistency does not imply completeness. A perfectly consistent design with a vague data model or an unexplored error handling story is not ready for implementation planning. The Dramaturg's job is to keep working until the design is genuinely ready to hand off, and to be honest with the user when it isn't yet.

### Phase 7: Final Design Doc

**Purpose:** Compile all reconciled sections into a single design document.

**Execution:** Write the compiled document to `docs/plans/designs/YYYY-MM-DD-<topic>-design.md`. This is the only file the skill produces. The user decides what happens next.

**Output format:** The design doc is intentionally open-ended. There is no rigid template — the Dramaturg writes whatever best captures the design vision it developed with the user. The document is more akin to a large, structured prompt for the Arranger than a formal specification. Most decisions are documented throughout the design naturally, as part of the narrative.

The Dramaturg's purpose is to develop, define, and refine a loose idea or group of ideas from the user into a coherent design vision. Restricting its output format would constrain the creative exploration that is the skill's entire value. The decision journal handles the structured decision trail separately — the design doc doesn't need to duplicate it.

**Required element — Project Goals:** The design doc must open with a clear, verbose articulation of the user's goals — what they want the feature to accomplish from their perspective, independent of technical decisions made during the design. This is the one structural element that the Dramaturg must include, because it is the anchor that every downstream skill relies on.

Goals should be rich with context. For a location feature, this is not "location-aware app" — it's "the app should be location-aware, with notifications that respect the user's physical locations. The user wants specific notification behaviors per location: silent mode at work, full alerts at home, workout reminders that activate automatically at the gym. The notification system should know where the user is and adjust its behavior accordingly, without the user needing to manually toggle modes."

Every goal the user stated during the design session belongs here, including specific behaviors, types, and capabilities they mentioned — even if those details also appear in later design sections. The goals section is a self-contained statement of intent that can be read without the rest of the document. If the Repetiteur needs to fundamentally redesign the implementation approach mid-pipeline, the goals section tells it exactly what the user wants to achieve — and nothing about how to achieve it.

The journal also captures goals with verbatim user input (see Section 6), but the design doc's goals section serves a different purpose: it's a synthesized, narrative statement of the complete vision, not a log of individual goal entries. Both are needed — the journal preserves the user's exact words; the goals section weaves them into a coherent picture.

**Required appendix — Arranger Notes:** The one structural requirement is an appended section at the end of the design doc that flags items for the Arranger's attention. This is an afterthought to the design work, not a structural constraint on it:

```markdown
## Arranger Notes

### New Protocols / Unimplemented Patterns
- [Protocol/pattern not currently in the codebase that the design relies on]
- [Each should have been researched during Phase 4 — reference the journal entry]

### Open Questions
- [Anything that couldn't be fully resolved during the design phase]
- [Project-specific details that need investigation against the actual codebase]
- [Should be few — most questions are resolved during the design discussion]

### Key Design Decisions
- [Brief pointers to the most architecturally significant decisions]
- [Not a full list — the journal has the complete trail]
```

This section gives the Arranger a quick orientation: what's new and needs verification, what's still uncertain, and where the biggest decisions are. The Arranger reads the full design doc regardless, but the notes section tells it where to focus its feasibility audit.

**What the Arranger receives:**
1. The design doc (goals section + vision, architecture, creative narrative)
2. The decision journal (Tier 3, structured decision trail with goal/use-case/decision categorization, verbatim user input, VERIFIED/PARTIAL research flags)
3. The Arranger Notes appendix (quick-orientation flags)

This is a context-aware split of duties. Rather than spending 40 minutes refining a vague idea and *then* trying to prepare to plan, the pipeline provides a clean, structured break between creative exploration and implementation planning. The Dramaturg thinks freely; the Arranger inherits a well-formed vision and a structured decision trail to work from.

---

## 4. Research Trigger Rules

Research doesn't happen on a schedule — it's triggered by specific conditions during the conversation. These rules define when and how.

### When Claude Recommends an Approach

**Rule:** Research FIRST, validate the approach, THEN present with evidence. Never recommend based solely on training data for substantive technical decisions. When presenting options, apply the priority chain: **compatibility > reliability > efficiency > security > performance.** Prefer modern, well-established approaches over bleeding-edge or legacy ones.

**Rationale:** Claude's training data has a cutoff, and technical ecosystems change. "Use SSE with Express" sounds reasonable from training data, but current research might reveal Express 4.x doesn't support HTTP/2 natively (a real issue this project encountered). Researching before recommending prevents the user from building on a faulty foundation.

**Example:**
- BAD: "I recommend using SSE for device communication because it's lightweight and unidirectional."
- GOOD: *[Claude researches SSE + Express + Flutter compatibility first]* "Based on research, SSE works well for this — here's what I found about the specific tech stack, including a consideration about HTTP/2 multiplexing..."
- PRIORITY CHAIN IN ACTION: If research shows Option A is faster but Option B has broader platform compatibility, recommend B — compatibility outranks performance.

### When User Proposes Something (Clear Intent)

**Rule:** If the user's proposal has clear intent and a defined direction, research immediately to validate feasibility.

**Rationale:** The user knows what they want — don't slow down the conversation by asking "should I research this?" when the answer is obviously yes. Validate and come back with findings.

**Example:** User says "Let's use WiFi positioning instead of GPS for the indoor use case." The intent is clear (WiFi over GPS for indoor location). Research WiFi positioning APIs on Android immediately.

### When User Proposes Something (Ambiguous Intent)

**Rule:** Clarify first. Fully understand the goal. Confirm understanding. THEN ask "should I research this approach?"

**Rationale:** Researching a misunderstood proposal wastes time and context. If the user says "we should make this work offline" — do they mean offline-first with sync? Cached data only? Local-only mode? Clarify before investing research effort.

**Example:**
1. User: "This should work without internet."
2. Claude: "Do you mean it should cache data for brief outages, or operate fully independently with sync when reconnected?"
3. User: "Full offline with sync."
4. Claude: "Got it — so a full offline-first architecture with conflict resolution on reconnect. Should I research offline sync patterns for Flutter + SQLite?"

### When a Protocol or Pattern Is Not Already Implemented

**Rule:** Any protocol, pattern, or technology that is not already implemented in the project should be researched through Gemini before being incorporated into the design.

**Rationale:** Claude's training data may describe a protocol's ideal behavior, but real-world integration involves version-specific gotchas, platform constraints, and ecosystem compatibility issues that only current research reveals. If the project doesn't already have working code for a pattern, Claude should not assume its training-data understanding is sufficient — validate via Gemini, which can synthesize current documentation and known issues.

**Example:** The project uses Express but has never implemented SSE. Even if Claude "knows" how SSE works, it should research SSE + Express + the specific stack to catch issues like the HTTP/2 incompatibility that training data might present as solved.

### When NOT to Research

- Trivial decisions that don't affect architecture ("should the button be blue or green?")
- Clarification exchanges that are still establishing intent
- Details that are clearly within Claude's reliable knowledge *and* the project already has working implementations of the relevant pattern
- Minor adjustments to already-researched approaches
- **When the user explicitly declines research.** If the user states a firm decision and says "I already know I want X, don't research alternatives," accept it. The skill augments the user's design process — it does not override the user's informed judgment. Document it in the decision journal as a user-directed decision (no Arranger note — the Arranger decides independently whether to verify user-directed decisions).

**Claude uses judgment, erring toward research for anything it's uncertain about.** The cost of unnecessary research is a few seconds of delay. The cost of a wrong recommendation is a flawed design. **When in doubt: research. When recommending: priority chain (compatibility > reliability > efficiency > security > performance). These are not suggestions — they are the skill's operating constraints.**

---

## 5. Research Execution Strategy

Research happens at two scales, with clear rules for which to use.

### Research Depth Principle

**Research should over-investigate, not under-investigate.** When a subagent or Gemini query is exploring a topic, it should go deep — checking edge cases, discovering limitations, finding alternatives, and forming recommendations. Shallow research ("X exists and seems to work") is less valuable than thorough research ("X exists, works for Y use case, has Z limitation, and A might be a better fit because B"). The cost of over-investigating is extra context; the cost of under-investigating is a flawed design built on surface-level understanding.

Research agents should also **provide explicit recommendations**, not just report findings. "Here are three options" is less useful than "Here are three options, and I recommend X because Y." This gives the discussion concrete proposals to react to rather than open-ended lists to evaluate from scratch. **Recommendations must follow the priority chain: compatibility > reliability > efficiency > security > performance.** Prefer modern, well-established approaches — not bleeding-edge, not legacy.

### Gemini Availability (Hard Requirement)

**Gemini MCP is required.** The Dramaturg does not launch without it. If Gemini becomes unavailable mid-session (timeout, rate limit, error), pause research and prompt the user: "Gemini is unavailable — want to wait, or continue without research on this topic?" Do not silently fall back to training data or brave-search as a Gemini substitute. The user decides.

### Inline Research (Main Session)

**When to use:** Claude can say "I just need to check this one thing."

**Tools:**
- `gemini-search` — Quick factual lookups, current best practices
- `gemini-query` — Analyze a specific technical question with Gemini
- `WebFetch` — Read a specific URL the user or research mentioned
- Quick codebase checks via direct Read/Grep

**Examples:**
- "Does Flutter's `geolocator` package support WiFi positioning?" → `gemini-search`
- "What's the current recommended SSE library for Express?" → `gemini-search`
- "Let me check the Flutter pubspec for existing location packages" → direct Read

### Subagent Research (Delegated)

**When to use:** Research might hit dead ends, requires checking multiple sources, or could waste main context on a path that turns out to be irrelevant.

**Tools:**
- `Explore` subagents for codebase analysis (understanding architecture, finding patterns)
- General-purpose subagents for multi-source web research
- `brave-search` via subagents for broad web queries

**Rationale:** The main session's context window is precious. If Claude thinks "this might be a dead end but I should check," that's a subagent job. The subagent wastes its own context exploring, and reports back a summary. If the path was productive, the summary is all the main session needs. If it was a dead end, the main session lost nothing.

**Examples:**
- "I'm not sure if this architectural pattern even works with our stack — let me have a subagent investigate" → Explore subagent
- "There are several competing approaches and I need to compare them across multiple sources" → general-purpose subagent
- "The user suggested an approach I've never heard of — let me research whether it's viable" → subagent (might be a dead end)

### Tool Selection Hierarchy

No rigid preference — use the best tool for each job. The ordering below reflects general priority, not strict rules:

1. **Gemini MCP** — Primary for possibility/feasibility research, technical questions, analyzing approaches. *Why primary:* Gemini excels at synthesizing technical knowledge, understanding tradeoffs, and providing structured analysis of approaches. It's already integrated via MCP and handles the most common dramaturg research pattern (feasibility checks and approach comparison) better than raw web search. **Gemini's alternative-recommending strength:** Gemini naturally suggests alternatives when analyzing an approach — this is highly valuable for the dramaturg's exploration phase. Not all suggested alternatives need deep investigation; briefly assess each against the current context and deep-dive only on the ones that look genuinely promising or that challenge an assumption in the current approach.
2. **Brave-search** — General web queries, finding recent articles, discovering libraries. *Why separate from Gemini:* Brave-search is better for discovering *what exists* (new libraries, recent articles, community sentiment) while Gemini is better for *analyzing what you've found*. They complement each other.
3. **Explore subagents** — Codebase understanding, architectural analysis. *Why subagent:* Codebase exploration can be verbose and context-heavy. Subagents explore, compress findings, and return summaries — keeping the main context clean.
4. **WebFetch** — Reading specific known URLs. For when research points to a specific resource that needs reading.
5. **General-purpose subagents** — Multi-source synthesis, speculative research. *Why last resort:* These are the most expensive in terms of latency and tokens. Use when the research genuinely requires synthesizing multiple sources or when the path is uncertain enough to justify the isolation.

### Conflicting Research Resolution

When research sources disagree, follow this hierarchy:

**Web wins.** Gemini excels at finding leads, synthesizing possibilities, and confirming known patterns — but when Gemini's synthesis contradicts real user experiences found through web search, the web evidence wins. Gemini's training data can smooth over real-world gotchas that actual developers have hit.

**Resolution protocol:**

1. **Detect the conflict.** If Gemini says "X works" but a web search reveals developers reporting "X breaks in scenario Y," that's a conflict. Don't silently pick a winner.
2. **Present the conflict to the user.** Be explicit: "Gemini says X, but I found user reports saying Y. Here's what each side says."
3. **For difficult conflicts:** Search for real user experiences — Stack Overflow answers, GitHub issues, blog posts from developers who actually implemented the pattern. User experiences outweigh synthesized analysis.
4. **Quick conflict checks:** Ask Gemini with **strict instructions to only cite real user experiences** in its answer — not to synthesize or extrapolate, but to report what actual users have encountered. This leverages Gemini's search capability while constraining it to evidence-based answers.
5. **Journal the conflict.** The decision journal entry should capture both sides, the resolution, and what evidence settled it. Mark as `PARTIAL` in the Arranger note if the conflict wasn't fully resolved — the Arranger should re-verify.

**Why this ordering:** Gemini is a synthesis engine. It's brilliant at "here are the three approaches and their tradeoffs" and at confirming that an approach is sound. But synthesis can paper over edge cases that real users discover in production. When a developer writes "I spent 3 days debugging this because the docs don't mention X," that's ground truth that no amount of synthesis outweighs.

**Reminder:** This hierarchy applies at all research scales — inline Gemini queries, subagent research, and cross-source comparison. **The priority chain (compatibility > reliability > efficiency > security > performance) still applies to the final recommendation after the conflict is resolved.**

### Inconclusive Research

When research comes back but doesn't settle the question — vague results, genuinely unsettled topics, or partial answers that leave important aspects unaddressed:

1. **Exhaust reasonable options first.** If Gemini returned something vague, try a more specific query. If a web search was too broad, narrow it. If an inline check was insufficient, consider a subagent for deeper investigation. **But do not chase wild geese** — if two different research approaches both come back inconclusive, the topic is genuinely uncertain and further digging won't help.
2. **Be transparent.** Tell the user exactly what was found and what wasn't: "I researched X through Gemini and web search. I found [partial findings], but couldn't get a clear answer on [specific gap]. Here's what I know and what remains uncertain."
3. **Prompt for guidance.** Give the user the choice: "Should I keep digging with a different approach, or should we make a decision with what we have and flag this as something the Arranger should verify?" The user knows their risk tolerance — some uncertainties are acceptable at the design level, others are blockers.
4. **Journal the gap.** If the user decides to proceed with incomplete information, the decision journal entry must capture: what was researched, what was inconclusive, and why the user chose to proceed. Mark the Arranger note as `PARTIAL` with the specific gap described — "PARTIAL: Could not confirm whether Android 14 restricts background WiFi scans differently from Android 13. Arranger should verify before committing to the polling interval."

**The goal is honest, bounded research — not exhaustive research.** The Dramaturg should try hard but know when to stop and surface the uncertainty rather than spinning on an unanswerable question.

---

## 6. Decision Journal

The decision journal is the Dramaturg's progressive external memory — it captures decisions, research findings, and vision context as the session progresses, ensuring no work is lost to context compaction or session interruption.

### Format

The journal is a **Tier 3 document** (per hybrid document structure conventions) using the `<journal>` wrapper tag. All text must be inside authority or structural tags — no naked markdown. This is Claude-only documentation; human readability is not a concern. The structured tagging maximizes downstream parseability for the Arranger and Repetiteur.

**User input is captured verbatim.** When the user explains their reasoning, states a goal, or describes a use case, the journal records their exact words — not a paraphrase or summary. Downstream skills (particularly the Repetiteur) need the user's actual language to understand intent, not Claude's interpretation of it. The `User verbatim` field in each entry captures this.

### Purpose

1. **Context preservation.** Long design sessions exhaust context. The journal externalizes settled decisions and research conclusions so Claude can re-read the journal rather than relying on the full conversation history remaining in context. This is critical for Phase 5 (section writing) which needs Phase 4 decisions but may have lost the original discussion to context pressure.
2. **Session resilience.** If a session dies mid-Phase-4, the journal preserves all settled approaches, vision answers, and research findings. A new session starts by reading the journal and picking up where it left off.
3. **Pipeline handoff.** Research-backed decisions documented in the journal carry forward to the Arranger skill. The Arranger's feasibility audit can skip re-verifying topics that the Dramaturg already researched — but only if the journal captures *what was researched, how, and what was found*, not just the conclusion. This eliminates redundant work across the pipeline.
4. **Audit trail.** The journal shows how the design evolved — not just the final answer but the path taken. This is valuable if later skills encounter issues and need to understand the Dramaturg's reasoning.

### Entry Types

The journal captures two categories of decisions, each with distinct entry formats:

**User-discussed entries** — settled through conversation with the user:

```markdown
## Decision: [Topic]
**Phase:** [Phase number and name]
**Category:** [goal | use-case | decision]
**Decided:** [What was decided]
**User verbatim:** [The user's exact words — not paraphrased. Capture the full statement that established this goal, use case, or decision.]
**User context:** [Additional reasoning, constraints, or motivation the user expressed across the discussion]
**Alternatives discussed:** [What else was considered, why rejected]
**Status:** settled
```

**Category tagging:**

- **goal** — What the user wants the feature to accomplish. Goals are the anchors that survive implementation replanning. "I want the app to be location-aware with notifications that respect my physical locations" is a goal. "I also want specific notification types — silent when at work, full alerts at home" is a goal. When the user describes desired outcomes, behaviors, or capabilities, those are goals. Be verbose — capture the full scope of what the user wants, including specific behaviors and types they mention. If a user statement contains both a goal and a technical decision, journal them as separate entries.
- **use-case** — How the user plans to use the feature. "When I arrive at the gym, I want my workout reminders to activate automatically" is a use case. Use cases provide extremely strong context for downstream skills — they reveal the user's mental model of how the feature works in practice, which constrains implementation in ways that goals alone don't capture.
- **decision** — A technical or design choice. This is the default — any settled discussion point that isn't a goal or use case. "Use FCM for push notifications" is a decision.

Goals and use cases are the entries that the Repetiteur treats as inviolable constraints. Decisions can be reworked if implementation proves them unworkable, but goals and use cases represent the user's intent and must be preserved across any replanning.

**Research-backed decisions** — validated through Gemini, web search, or subagent investigation:

```markdown
## Research: [Topic]
**Phase:** [Phase number and name]
**Question:** [What was being validated]
**Tools used:** [gemini-search / gemini-query / brave-search / subagent / WebFetch]
**Findings:** [What was discovered — specific enough for the Arranger to determine whether re-verification is needed]
**Decision:** [What was decided based on findings]
**Arranger note:** [VERIFIED — skip re-audit] or [PARTIAL — Arranger should verify X specifically]
**Status:** settled
```

The **Arranger note** field is the cross-skill optimization. When the Dramaturg has thoroughly researched a protocol or platform behavior, it marks the entry `VERIFIED` so the Arranger's feasibility audit can skip it. When research was partial or inconclusive, `PARTIAL` flags what still needs verification — preventing the Arranger from assuming completeness.

### Special Entries

**Vision baseline (Phase 2 exit):**

```markdown
## Vision Baseline
**What:** [What the user wants to build]
**Why:** [Motivation, problem it solves]
**How used:** [Concrete use cases, user scenarios]
```

This entry is the regression test anchor. When Phase 5 feedback triggers vision regression detection, Claude compares against this entry — not against its memory of a conversation that may have compacted away.

**Topic map (Phase 3 exit):**

```markdown
## Topic Map
**Areas to explore in Phase 4:**
- [Topic 1] — [why it needs exploration]
- [Topic 2] — [why it needs exploration]
- ...
**User confirmed coverage:** yes
```

This addresses the Phase 3 coverage gap — the topic map is explicit, user-confirmed, and recoverable.

**Section approval (Phase 5):**

```markdown
## Section Approved: [Section Name]
**Key decisions reflected:** [references to earlier journal entries]
**User feedback incorporated:** [any revisions made during review]
```

Compact — just enough for the reconciliation pass (Phase 6) to work from the journal without re-reading full sections.

### Checkpoint Triggers

The Dramaturg writes to the journal at:
- Phase 2 exit: vision baseline entry
- Phase 3 exit: topic map entry
- Every settled topic in Phase 4 (decision or research entry)
- Every section approval in Phase 5
- Any point where significant autonomous work (research, analysis) is about to be discussed with the user — journal first, then discuss. This ensures the work survives even if the discussion takes an unexpected turn.

### File Location & Lifecycle

- **Location:** `<planning_dir>/dramaturg-journal.md` (same directory as the design doc output)
- **Append-only.** Entries are never edited. If a later decision supersedes an earlier one, the new entry references the old one.
- **Archived** to `docs/archive/` when the design doc is finalized. The journal is a working artifact, not a deliverable — but it persists alongside the archived design for the Arranger to consume.
- **Not committed** with the final design doc — archived separately.

### Context Management Role

The journal is the Dramaturg's answer to context exhaustion:
- **During Phase 5:** If Phase 4 discussions have compacted, Claude re-reads the journal to recall settled approaches rather than relying on conversation history.
- **During Phase 6 (Reconciliation):** The journal's section approval entries provide a compact view of all approved sections without re-reading full section text.
- **On session resumption:** A new session reads the journal first, then the most recent conversation context, and can determine: what phase was active, what's been settled, what research has been done.

The journal does not *solve* the 200k context limit — a complex feature will still pressure context. But it ensures that the *most important* information (settled decisions, validated research, vision baseline) is recoverable regardless of what slides out of the conversation window.

**The Phase 4→5 boundary is the designed session split point.** The workflow intentionally separates the discussion-research loop (Phase 4) from section writing (Phase 5) so the user can cut the session between them. A new session reads the journal and proceeds directly to Phase 5 with full context available — no loss of settled decisions, no need to re-discuss approaches. This architectural choice means the skill never needs to compress output or rush analysis to save context. Phase 4 gets thorough analysis; Phase 5 gets a clean context budget.

---

## 7. Vision Regression Detection

The most subtle behavior in the skill: detecting when user feedback during section review actually changes the fundamental vision, not just the implementation approach.

### The Three-Question Test

If user feedback would change Claude's answer to any of these three questions (originally established in Phase 2), the conversation has regressed to vision territory:

1. **What** does the user want to build?
2. **Why** do they want to build it?
3. **How** will they use it?

### Detection Hierarchy

| Feedback type | Example | How to detect | Loop to |
|---|---|---|---|
| **Vision change** | "Actually, this should also work offline" | Changes *how they'll use it* — the use cases are now different | Phase 2 (Vision) |
| **Approach change** | "Use WebSockets instead of SSE" | Same goal, same use cases, different technical method | Phase 4 (Approach Discussion) |
| **Detail change** | "Make that timeout 30s instead of 10s" | Same approach, minor parameter adjustment | Revise section in place |

### Behavior When Regression Is Detected

1. **Claude proactively surfaces it:** "This sounds like it changes the original vision — you initially wanted X for Y reason, and this feedback suggests Z. Should we revisit the vision?"
2. **User confirms or denies:** The user decides whether regression is real. Sometimes what looks like a vision change is just poor phrasing of a detail change.
3. **If confirmed:** Previously approved sections remain in place but will be re-evaluated during the Reconciliation Pass (Phase 6). Claude returns to Phase 2 to re-establish the vision.
4. **If denied:** Claude continues in the current phase, adjusting the section as a detail or approach change per the user's intent.

**Decision rationale:** The explicit confirmation step prevents false positives. Claude might misinterpret "oh, and it should also send notifications" as a vision change when the user considers it an obvious detail within the existing vision. But without detection at all, a genuine vision shift gets silently absorbed into a section revision, leading to an incoherent design.

---

## 8. Conversation Style

These aren't suggestions — they're hard rules that define the skill's character and differentiate it from deep-plan's rigid pipeline.

### One Question at a Time (With an Exception)

**Default:** One question per message. If Claude needs to understand three things, that's three messages.

**Exception:** Multiple questions are acceptable in a single message **when all questions are tight, focused, and expect short answers** — not open-ended explorations. This is appropriate for quick confirmations, binary choices, or parameter-style questions where each answer is a few words. If any question in the batch could reasonably produce a multi-paragraph response, it should be asked alone.

**Why the default:** Multi-question messages with open-ended questions get partial answers. The user addresses the first two, forgets the third, and Claude has to follow up anyway. Worse, the user might combine answers in ways that create ambiguity about which answer goes with which question.

**Why the exception:** Asking "Should the timeout be 30s or 60s?" and "Do you want retry on failure?" as separate messages wastes the user's time. These are tight, bounded questions — batching them respects the conversation's pace.

### Output Length Guidance

Claude's output should be as long as the topic demands — **never truncate or compress reasoning to fit an artificial limit.** If Claude's analysis, recommendations, and rationale for a single topic run to 200 lines, that is perfectly acceptable and should never be discouraged. Thorough reasoning is the skill's entire value.

The ~75 line mark is a useful heuristic for **when to split content into separate messages**, not a cap on total output. If Claude is presenting findings and the output is approaching this range, consider whether the content covers multiple distinct topics that would be better served as separate messages. But a single complex topic that genuinely needs 200 lines of analysis should get those 200 lines in one message.

**This rule is not in tension with context management.** The workflow is architecturally designed so that thorough output never needs to be sacrificed for context preservation. The decision journal externalizes settled decisions, and the Phase 4→5 boundary provides a natural session split point. Claude should never think "I need to be brief here to save context for later" — that defeats the skill's purpose. If context pressure is a concern, the answer is a session split at the Phase 4→5 boundary, not compressed analysis.

### One Section at a Time for Review

When presenting design sections for review (Phase 5), present exactly one section per message.

**Why:** When reviewing 8 topics at once, the 2-3 that need detailed responses get buried in the volume. Single-section reviews keep scope focused. The user can give each section the attention it deserves.

### Open-Ended Questions Preferred

Design questions to draw out the user's thinking, not to get a quick yes/no.

**Why:** The user's unspoken context — their mental model, their priorities, their past experience — is the most valuable input to the design process. Open-ended questions surface this. "Tell me about how users will discover this feature" yields far more design insight than "Should there be a button on the home screen?"

### Conversational, Collaborative Tone

This is a design discussion between colleagues, not an interview or a form to fill out. Claude should:
- Share its own thinking and reasoning
- Explain why it's asking a question
- React to user input with genuine analysis
- Admit uncertainty ("I'm not sure about this — let me research it")

### Fluid Transitions

Phase transitions should feel natural in conversation, not like "PHASE 2 COMPLETE, ENTERING PHASE 3." The phases are a structural backbone, not a visible UI. The only exception is the Vision Discussion gate (Phase 2 → Phase 3), which is strict because the consequences of moving forward without understanding are severe.

### Reinforcement in the Skill File

This design document establishes the reinforcement principle in Section 2 (Core Design Principles) and follows it throughout — the priority chain and research-first mandate appear in every section where they're relevant. **The eventual skill file must do the same.** A skill file is a system prompt, and system prompt instructions fade as conversation context grows. The skill file must:
- Repeat the priority chain wherever recommendations are discussed
- Repeat the research-first mandate wherever Claude might skip research
- Repeat the one-question rule wherever Claude generates questions
- Include reminders in any templates, checklists, or phase transition instructions

One or two authoritative statements at the top of a skill file DO NOT survive a long design session. The constraint must be present at the point of action, every time.

---

## 9. Plugin Structure

The dramaturg skill lives in a standalone plugin alongside the conductor and musician skills, reinforcing the pipeline metaphor.

### Location

```
~/.claude/skills/dramaturg/    (or equivalent plugin directory)
```

### Pipeline Position

```
dramaturg → arranger → conductor → musician
(vision)    (plan)     (coordination)  (implementation)
                        copyist creates individual parts from arranger's score
```

The dramaturg produces a design doc that the arranger consumes to create an executable plan. The dramaturg's decision journal carries forward research-backed decisions marked VERIFIED, eliminating redundant feasibility work in the arranger's audit phase.

The user controls the pipeline — no automatic handoff. After dramaturg completes, they may:
- Feed the design to the arranger (natural next step)
- Use writing-plans or deep-plan as alternatives
- Feed directly to the conductor if the design is detailed enough
- Implement manually

---

## 10. Design Decisions Log

Decisions made during the brainstorming process and their rationale:

| Decision | Alternatives Considered | Rationale |
|---|---|---|
| **Discussion loop with research interrupts** over rigid pipeline | Phased approach (like deep-plan), pure free-form (like brainstorming) | The looping structure gives brainstorming's fluidity while the research triggers add deep-plan's rigor. Rigid phases recreate deep-plan's weakness. |
| **Design doc only** (no automatic handoff) | Handoff to writing-plans (like brainstorming), full plan (like deep-plan) | Keeps the skill focused on vision/design. Decouples it from implementation planning, which has its own complexities. |
| **Standalone plugin** over superpowers integration | Adding to superpowers plugin | Reinforces the pipeline metaphor (dramaturg → conductor → musician). Keeps the official superpowers plugin clean. |
| **Vision gate (What/Why/How-used)** as strict transition | Claude's judgment for transition, explicit "ready?" question | Three concrete questions are testable and prevent the most common failure (racing ahead with assumed understanding). |
| **One question/section at a time** | Batched questions, full-document review | Multi-topic replies lose detail. Single-item focus ensures every important point gets proper attention. |
| **Max 6 broad scoping questions** | Unlimited scoping, no scoping phase | Limits force high-quality questions. Without limits, Claude asks 15 mediocre questions. Without the phase, detailed work starts without a big-picture map. |
| **Subagents for speculative research** | All research inline, all research delegated | Protects main context from dead ends while keeping quick lookups fast. "Will this be a dead end?" → subagent. "Quick factual check" → inline. |
| **Separate Approach Discussion (Phase 4) from Per-Section Review (Phase 5)** | Combined research-discuss-review in one phase | Without separation, Per-Section Review does two jobs: hashing out the approach AND reviewing details. This overloads reviews — the first "review" becomes a direction debate followed by detail feedback. Separating them means Phase 4 settles the *what* through research-backed discussion, and Phase 5 reviews the *details* of already-settled approaches. Real example: location feature went SSE → FCM → refined FCM in discussion; by section review, only FCM details needed attention. |
| **Topic-guided but flexible discussion** in Phase 4 | Per-topic strict loops, free-flowing with no structure | Strict per-topic loops feel rigid and prevent natural tangent exploration. Free-flowing risks losing track of what's settled. Topic-guided with explicit settlement confirmations balances fluidity with clarity. |
| **Reconciliation pass** before final doc | No reconciliation, continuous reconciliation | Without it, later changes create contradictions in the final doc. Continuous reconciliation would interrupt the discussion flow. A final pass is the right balance. |
| **User-confirmed vision regression** | Auto-detect and regress, ignore regression | Auto-detection has false positives (user meant a detail, Claude thinks it's a vision change). Ignoring regression leads to incoherent designs. Explicit confirmation gives the user control while Claude surfaces the concern. |
| **Over-investigate by default** | Proportional research, minimal research | Shallow research leads to flawed designs built on surface-level understanding. The cost of extra context is low; the cost of a wrong recommendation is high. Research agents should go deep and provide explicit recommendations, not just report options. |
| **Multiple tight questions allowed** | Strict one-question-per-message always | Asking "timeout 30s or 60s?" and "retry on failure?" as separate messages wastes time. Tight, bounded questions can be batched. Open-ended questions still get asked alone to ensure quality answers. |
| **~75 line split heuristic, no output cap** | Hard output limits, no guidance | Claude should never truncate reasoning to fit an artificial limit. 200 lines for a complex topic is perfectly acceptable. The ~75 line mark guides when to split across messages, not when to stop thinking. |
| **Pipeline: dramaturg → arranger → conductor → musician** | dramaturg → conductor directly | An arranger between dramaturg and conductor translates vision into executable plans. Both skills designed before building either, enabling tight integration. Journal entries with VERIFIED/PARTIAL flags enable the arranger to skip redundant research. |
| **Gemini for unimplemented protocols** | Trust training data for known patterns | Even "known" patterns have version-specific gotchas. If the project hasn't implemented it, Gemini should validate before the design assumes it works. Real example: SSE + Express HTTP/2 incompatibility. |
| **Gemini alternatives: triage, don't deep-dive all** | Investigate every alternative equally, ignore alternatives | Gemini naturally suggests alternatives — valuable for exploration. But not all need deep investigation. Brief assessment against context, deep-dive only on promising ones that challenge assumptions. |
| **Name: "dramaturg"** | "composer," "overture," "vision-planner," "research-planner" | The theater dramaturg literally does research to inform creative vision — the most accurate metaphor. Also creates a natural arts/performance pipeline with conductor and musician. |
| **Decision journal as progressive external memory** | No state preservation, inline summaries only, temp file checkpoints | Addresses three critical review findings at once: context exhaustion (settled decisions survive compaction), session resilience (journal is a resumption checkpoint), and research preservation (Arranger can skip re-verification of VERIFIED entries). Adapted from the Arranger skill's journal design, with entry types tuned for vision/design-level decisions rather than implementation specifics. |
| **Dual entry types (user-discussed + research-backed)** | Single entry format for all decisions | User-discussed entries capture conversational context and reasoning. Research-backed entries capture tools, findings, and an explicit Arranger note (VERIFIED/PARTIAL) enabling the Arranger to skip redundant feasibility audits. Different concerns, different formats. |
| **Pipeline: dramaturg → arranger → conductor → musician** | dramaturg → impl planner → conductor → musician | The "implementation planner" is now named "arranger" — takes a composition and creates the detailed arrangement for the orchestra. Creates a complete arts pipeline with the copyist creating individual parts from the arranger's score. |
| **Priority chain: compatibility > reliability > efficiency > security > performance** | No explicit ordering, case-by-case | Explicit ordering prevents ambiguous trade-off decisions during research-backed recommendations. Modern but not bleeding edge. Future-favoring. Shared with the Arranger to maintain consistent decision-making across the pipeline. |
| **Reinforcement principle: repeat constraints at point of relevance** | State constraints once authoritatively | 1-2 strong statements at the top of a skill file DO NOT survive long sessions — they get buried under growing context. Critical constraints must appear everywhere they're relevant: in phase descriptions, research rules, templates, and transition instructions. This design doc follows the same principle. |
| **Web wins research conflicts** | Gemini wins, majority rules, ask user every time | Gemini excels at finding leads and confirmations but its synthesis can paper over real-world edge cases. Real user experiences (Stack Overflow, GitHub issues, dev blogs) are ground truth. Quick conflict checks: ask Gemini with strict rules to only cite user experiences. Difficult conflicts: web search for developer experiences wins. |
| **Reconciliation severity threshold** (cosmetic/clarification/substantive) | Re-approve everything, no reconciliation | Without thresholds, every terminology drift triggers re-approval. 6-8 sections with 2-3 cosmetic fixes each = 12-18 unnecessary approval gates. Severity classification ensures user attention is spent on real conflicts. Cosmetic: fix silently. Clarification: inform. Substantive: require re-approval. |
| **Topic map at Phase 3 exit** | Claude's judgment for transition, no coverage check | Explicit topic list + user confirmation catches missing areas before research investment. Journaled as recoverable checkpoint. Cheap (one extra exchange) with high coverage value. |
| **Tangent bounding in Phase 4** | Unlimited natural tangents, strict per-topic only | If a tangent would be its own design doc section, table it. Prevents aggregate drift while allowing genuinely related discussion. Strict per-topic kills natural exploration; unlimited tangents cause unbounded Phase 4. |
| **Pivot on infeasibility** | Force the approach, ask user first, abandon topic | When research proves infeasibility, present the finding and loop back with alternatives. Not a failure — the skill's value is catching these. No need to ask permission to pivot; the finding speaks for itself. |
| **Ripple assessment on Phase 5→4 loop-back** | Invalidate all sections, no ripple check | When re-entering Phase 4 for one topic, assess whether the new approach affects other settled topics. List affected topics, get user confirmation before invalidating. Unaffected sections retain approval. Proportional response. |
| **Accept user-declined research** | Always research regardless, negotiate | The skill augments the user's process, not overrides it. User says "I know I want X" = user-directed decision, journaled without Arranger note. Arranger decides independently whether to verify. |
| **Scope protection for implementation details** | Answer implementation questions, defer silently | Redirect to architecture-level language when users push for file paths, function names, package versions. Premature implementation details become constraints that limit design exploration. Explicit about why: "we'll define that in the implementation plan." |
| **Security lens in Phase 5** | Full security audit, no security check, separate security phase | Design-level security surfacing during section review. Flag PII, auth, data storage, external service concerns. Not a full audit (that's implementation). Silent when clean — no checkbox noise on non-security sections. |
| **Gemini is a hard requirement** | Graceful degradation, launch without it | The skill does not launch without Gemini. If unavailable mid-session, pause and prompt user. No silent fallback to training data — the user decides. |
| **Inconclusive research: exhaust then surface** | Keep digging indefinitely, default to training data, skip and move on | Exhaust reasonable options (requery, narrow, try subagent) but don't chase wild geese. Be transparent about what was found and what wasn't. Prompt user: keep digging or proceed with uncertainty? Journal the gap with PARTIAL Arranger note. Honest, bounded research — not exhaustive. |
| **Complexity as conversational note, not formal gate** | Formal complexity phase, ignore complexity, Arranger handles all | Obvious over-engineering should be surfaced during Phase 4 discussion — "this works but it's dramatically more complex than the simpler option." Not a loop step or checklist item. Detailed cost/effort analysis is the Arranger's job; Dramaturg just catches the obvious cases early. |
| **Phase 4→5 as designed session split point** | Compress output in later phases, no split guidance, always split | The Phase 4/5 separation is dual-purpose: approach vs detail review AND natural session boundary. Journal preserves all state across the split. For large features (8+ topics), recommend user evaluate splitting. Never compress analysis to save context — split the session instead. |
| **Open-ended output with Arranger Notes appendix** | Rigid template, fully unstructured, mirror journal format | The design doc is a creative artifact — a structured prompt for the Arranger, not a form. No template constraints on the narrative. Required appendix flags new protocols, open questions, and key decisions for the Arranger's orientation. The journal handles the structured decision trail. Three artifacts together (doc + journal + notes) give the Arranger everything it needs without constraining the Dramaturg's creative exploration. |
| **Required Goals section in design doc** | Goals only in journal, no explicit goals section, structured requirements list | The Goals section is a verbose, narrative statement of what the user wants to achieve — independent of technical decisions. Downstream skills (especially Repetiteur) need to distinguish between user intent and implementation choices. The journal captures individual goal entries with verbatim user words; the design doc's goals section synthesizes them into a coherent picture. Both are needed for different purposes. |
| **Journal as Tier 3 documentation** | Tier 2, Tier 1 markdown | The journal is exclusively Claude-consumed — no human readability concern. Tier 3 `<journal>` wrapper with all text inside tags maximizes structural signal for downstream parsing by the Arranger and Repetiteur. Verbatim user input preservation ensures downstream skills get the user's actual words, not Claude's interpretation. |
| **Goal and use-case category tags in journal** | Separate entry types, no categorization, all entries equal | Simple `Category: goal \| use-case \| decision` field on user-discussed entries. Goals and use cases are inviolable for the Repetiteur — they represent user intent that survives implementation replanning. Decisions can be reworked. This distinction is the core mechanism for the Repetiteur to know what must be preserved vs what can change. |
| **Implementation readiness gate at Phase 6→7** | Readiness in Phase 2, no explicit gate, user decides readiness | Gate at Phase 6→7 asks: could the Arranger produce an executable plan without making design-level decisions? If no, Dramaturg keeps probing. Reconciliation ensures consistency but not completeness — a consistent but vague design isn't ready. The gate prevents premature handoff to implementation planning. |
