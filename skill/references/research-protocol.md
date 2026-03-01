<skill>

<sections>
- trigger-rules
- diversion-flow
- tool-hierarchy
- subagent-delegation
- conflict-resolution
- inconclusive-research
- gemini-failure
</sections>

<!-- Loading guide:
  - §trigger-rules, §diversion-flow: Load on first research diversion
  - §tool-hierarchy, §subagent-delegation: Load on first research execution
  - §conflict-resolution: Load when encountering conflicting sources
  - §inconclusive-research: Load when research returns inconclusive
  - §gemini-failure: Load when Gemini fails mid-session
-->

<section id="trigger-rules">
<core>
## Research Trigger Rules

Research happens when triggered by specific conditions during conversation. It does not happen on a schedule.

**When you recommend an approach:** Research FIRST. Validate the approach, THEN present with evidence. <mandatory>Never recommend based solely on training data for substantive technical decisions. Apply the priority chain: compatibility > reliability > efficiency > security > performance.</mandatory>

**When the user proposes something with clear intent:** Research immediately to validate feasibility. The intent is clear — do not slow the conversation by asking "should I research this?"

**When the user proposes something with ambiguous intent:** Clarify first. Fully understand the goal. Confirm understanding. THEN ask "should I research this approach?"

**When a protocol or pattern is not already implemented in the project:** Research through Gemini before incorporating it into the design. Even if you "know" how something works from training data, validate via current research to catch version-specific gotchas and platform constraints.

**When NOT to research:**
- Trivial decisions that don't affect architecture
- Clarification exchanges still establishing intent
- Details within the project's reliable, existing patterns
- Minor adjustments to already-researched approaches
- When the user explicitly declines research — accept their informed judgment, journal as user-directed decision
</core>
</section>

<section id="diversion-flow">
<core>
## Research Diversion Flow

<mandatory>Research diversions use the decision journal for flow anchoring. Write the entry before research, update to "settled" after return. Research diversions return directly to the recorded step — no Phase Registry re-bootstrap needed.</mandatory>

**Step 1 — Record position in the decision journal:**

<template follow="format">
## Research Diversion (in progress)
**Position:** [reference-file.md] §[section-id] → [step]
**Topic:** [what was being discussed] — [context for why research is needed]
**Trigger:** [user-proposal-clear-intent | dramaturg-recommendation | unimplemented-pattern | user-proposal-ambiguous]
**Status:** in-progress
</template>

**Step 2 — Execute research:**

Tool selection (use the best tool for each job):
1. **Gemini MCP** — Primary for feasibility research, technical questions, approach analysis. Excels at synthesizing tradeoffs and suggesting alternatives.
2. **Brave-search** — Discovering what exists: new libraries, recent articles, community sentiment.
3. **Explore subagents** — Codebase understanding, architectural analysis. Use subagents to keep main context clean.
4. **WebFetch** — Reading specific known URLs.
5. **General-purpose subagents** — Multi-source synthesis, speculative research. Isolates verbose or speculative exploration from the main conversation context — if the path is a dead end, the main session loses nothing.

For detailed tool rationale, subagent delegation patterns, and research depth guidance, see §tool-hierarchy and §subagent-delegation.

**Research should over-investigate, not under-investigate.** Go deep — check edge cases, discover limitations, find alternatives, form recommendations. Shallow research ("X exists and seems to work") is less valuable than thorough research ("X exists, works for Y, has Z limitation, and A might be better because B").

**Provide explicit recommendations**, not just findings. "Here are three options, and I recommend X because Y" is more useful than "here are three options." <mandatory>Recommendations must follow the priority chain: compatibility > reliability > efficiency > security > performance.</mandatory>

**Gemini alternatives:** Gemini naturally suggests alternatives — this is valuable. Briefly assess each against the current context. Deep-dive only on the ones that look genuinely promising or challenge an assumption. Not all alternatives need full investigation.

**Step 3 — Return to recorded step:**

Read the most recent "Research Diversion (in progress)" entry in the decision journal. Return directly to the recorded position in the reference file. Update the journal entry's status from "in-progress" to "settled" and add a **Findings** field with the research results. Resume the discussion with research findings.

<mandatory>Update the journal entry status after returning. In-progress entries must not accumulate — they indicate either active research or a stale diversion from a session interruption. If a session starts with an existing in-progress research diversion entry, re-execute the research from the recorded context.</mandatory>
</core>
</section>

<section id="tool-hierarchy">
<core>
## Tool Hierarchy — Detailed Rationale

### Why Gemini Is Primary

A single Gemini query is fundamentally different from a single web search. Gemini runs multiple searches internally, processes the results through a logic filter, and returns synthesized analysis. One Gemini call replaces what would otherwise be 5-10 individual Brave queries plus your own synthesis work — all without consuming main conversation context on intermediate results.

This makes Gemini the most context-efficient research tool available. You get a synthesized answer with tradeoffs analyzed, alternatives surfaced, and edge cases identified — in one call. Using Brave-search to achieve the same result would mean: run query → read results → run refined query → read more results → synthesize findings — consuming conversation context at every step.

**Where Gemini excels:**
- Feasibility checks — "Does X work with Y in this configuration?"
- Approach comparison — "What are the tradeoffs between A, B, and C?"
- Technical deep-dives — "How does this protocol handle edge case Z?"
- Alternative discovery — Gemini naturally suggests alternatives when analyzing an approach, which is highly valuable for Phase 5 exploration

**Where Gemini is weaker:**
- Discovering *what exists* in the current ecosystem — new libraries, recent releases, community sentiment. Brave-search is better for "what's out there?" while Gemini is better for "analyze what I've found."
- Ground truth on real-world gotchas — Gemini's synthesis can smooth over edge cases that actual developers have hit. See §conflict-resolution.

### Brave-Search

Complements Gemini. Use for:
- Discovering new libraries, tools, or frameworks
- Finding recent articles and release announcements
- Gauging community sentiment ("are people actually using X?")
- Surfacing real user experiences for conflict resolution (§conflict-resolution)

### WebFetch

For reading specific known URLs — documentation pages, GitHub READMEs, API references. Use when research or the user points to a specific resource that needs detailed reading.

### Explore Subagents

For codebase understanding and architectural analysis. Subagents explore, compress findings, and return summaries — keeping the main conversation context clean for the design discussion. Use for:
- Understanding project architecture and patterns
- Finding existing implementations of relevant patterns
- Checking how the codebase currently handles related features

### General-Purpose Subagents

For multi-source synthesis, speculative research, and complex external investigation. General-purpose subagents can use all available tools (Gemini, web search, WebFetch) and are suited for:
- Comparing multiple external approaches across several sources
- Speculative research that might be a dead end (protects main context)
- Synthesizing findings from diverse sources into a recommendation

Where Explore subagents are optimized for codebase navigation, general-purpose subagents are optimized for breadth of external research.
</core>
</section>

<section id="subagent-delegation">
<core>
## Subagent Delegation — When and Why

### The Core Principle

Subagents protect the main conversation context. Your primary context is the design discussion with the user — research that might be verbose, speculative, or a dead end should not consume that context directly.

### When to Use Subagents

**Speculative research — might be a dead end.** If you think "this might not lead anywhere but I should check," that's a subagent job. The subagent explores in isolation. If productive, the summary is all the main session needs. If a dead end, the main session lost nothing.

**Multi-source comparison.** Comparing three frameworks or five approach options across multiple dimensions generates verbose intermediate analysis. A subagent synthesizes and returns a focused comparison.

**Verbose codebase exploration.** Understanding "how does the existing notification system work?" might require reading 10+ files. An Explore subagent reads them all and returns a summary — the design discussion gets the insight without the file-reading noise.

### When NOT to Use Subagents

**Quick factual checks.** "Does Flutter's geolocator package support WiFi positioning?" — one Gemini query, answered inline. No subagent overhead needed.

**Research with clear direction.** When you know exactly what to check and the answer will be concise, inline research is faster and lower-overhead.

### Choosing Subagent Type

- **Explore subagents** — for codebase understanding, architectural analysis, finding existing implementations. These subagents have fast file search tools and are optimized for navigating code.
- **General-purpose subagents** — for multi-source external research, speculative investigation, synthesis across web sources. These subagents can use all research tools and are better for open-ended exploration outside the codebase.

Do not use Explore subagents for external research (they lack web search tools) or general-purpose subagents for simple codebase queries (they're slower than Explore for file navigation).

### Subagent Prompt Quality

Give subagents specific questions, not vague directives:
- **Good:** "Research whether Express 4.x supports HTTP/2 natively, and if not, what the workaround options are. Check for known issues with SSE over HTTP/2 in this stack."
- **Bad:** "Research SSE for our project."

Specific questions get focused answers. Vague prompts get broad, unfocused exploration that may miss the actual question.

### Research Depth in Subagents

<mandatory>Subagent research should over-investigate, not under-investigate. Subagents should go deep — check edge cases, discover limitations, find alternatives, and form explicit recommendations with rationale. "X exists and seems to work" is less valuable than "X exists, works for Y, has Z limitation, and A might be better because B." Recommendations must follow the priority chain: compatibility > reliability > efficiency > security > performance.</mandatory>

### Research Diversion Sequencing

Research diversions are sequential, not nested. Complete the current diversion and return to the conversation before starting another. If a diversion reveals a new question that needs research, complete the current diversion first, discuss findings with the user, then initiate a new diversion for the follow-up question.
</core>
</section>

<section id="conflict-resolution">
<core>
## Conflict Resolution

When research sources disagree, follow this hierarchy.

### Ground Truth Wins

Gemini excels at finding leads, synthesizing possibilities, and confirming known patterns. But when Gemini's synthesis contradicts real user experiences found through web search, the web evidence wins.

<mandatory>Gemini's synthesis can smooth over real-world gotchas that actual developers have hit. When a developer writes "I spent 3 days debugging this because the docs don't mention X," that's ground truth that no amount of synthesis outweighs.</mandatory>

### Resolution Protocol

1. **Detect the conflict.** If Gemini says "X works" but web search reveals developers reporting "X breaks in scenario Y" — that's a conflict. Do not silently pick a winner.
2. **Present the conflict to the user.** Be explicit: "Gemini says X, but I found user reports saying Y. Here's what each side says."
3. **Quick conflict check:** Ask Gemini with strict instructions to only cite real user experiences — not to synthesize or extrapolate, but to report what actual users have encountered. This leverages Gemini's search capability while constraining it to evidence-based answers.
4. **For difficult conflicts:** Search for real user experiences — Stack Overflow answers, GitHub issues, blog posts from developers who actually implemented the pattern. Real experiences outweigh synthesized analysis.
5. **Journal the conflict.** The decision journal entry must capture both sides, the resolution, and what evidence settled it. Mark as PARTIAL in the Arranger note if the conflict wasn't fully resolved.

<mandatory>The priority chain (compatibility > reliability > efficiency > security > performance) still applies to the final recommendation after the conflict is resolved.</mandatory>
</core>
</section>

<section id="inconclusive-research">
<core>
## Inconclusive Research

When research returns but doesn't settle the question — vague results, genuinely unsettled topics, or partial answers that leave important aspects unaddressed.

### Step 1: Exhaust Reasonable Options

If Gemini returned something vague, try a more specific query. If a web search was too broad, narrow it. If an inline check was insufficient, consider a subagent for deeper investigation.

<mandatory>Do not chase wild geese. If two different research approaches both come back inconclusive, the topic is genuinely uncertain. Further digging won't help — surface the uncertainty instead of spinning.</mandatory>

### Step 2: Be Transparent

Tell the user exactly what was found and what wasn't: "I researched X through Gemini and web search. I found [partial findings], but couldn't get a clear answer on [specific gap]. Here's what I know and what remains uncertain."

### Step 3: Prompt for Guidance

Give the user the choice: "Should I keep digging with a different approach, or should we make a decision with what we have and flag this for the Arranger to verify?"

The user knows their risk tolerance. Some uncertainties are acceptable at the design level, others are blockers.

### Step 4: Journal the Gap

If the user decides to proceed with incomplete information, the decision journal entry must capture:
- What was researched
- What was inconclusive
- Why the user chose to proceed

Mark the Arranger note as PARTIAL with the specific gap described: "PARTIAL: Could not confirm whether Android 14 restricts background WiFi scans differently from Android 13. Arranger should verify before committing to the polling interval."
</core>
</section>

<section id="gemini-failure">
<core>
## Gemini Failure Mid-Session

If Gemini becomes unavailable during an active design session (timeout, rate limit, error):

1. **Pause research.** Do not silently fall back to training data or substitute Brave-search as a Gemini replacement.
2. **Inform the user.** "Gemini is unavailable right now — [error description]."
3. **Offer the choice:** "Want to wait and retry in a few minutes, or continue without research on this topic?"
4. **If continuing without research:** Clearly mark any subsequent recommendations as unresearched. The user should know which decisions have research backing and which don't. Journal any unresearched decisions without an Arranger note — the Arranger must verify them independently.

<mandatory>Never silently fall back to training data. The user decides what happens when Gemini is unavailable. Your value is research-backed design — removing the research without the user's knowledge undermines the entire skill.</mandatory>

### Degraded Gemini Performance

If Gemini returns but with intermittent issues (timeouts on some queries, suspiciously generic responses, low-quality results that don't address the specific question):

1. **Retry once** with a more specific query. Generic results often mean the query was too broad.
2. **If still degraded:** Inform the user: "Gemini's responses seem generic/unreliable right now. I can supplement with targeted web searches, or we can wait."
3. **Mark any results from degraded Gemini as lower confidence** in the decision journal: "Note: Gemini response quality was degraded during this research."
</core>
</section>

</skill>
