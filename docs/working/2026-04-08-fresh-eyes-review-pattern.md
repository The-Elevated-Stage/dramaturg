# Fresh Eyes Review Pattern — Enrichment from Aletheia Design Review

## Source Session
**Date:** 2026-04-08
**Context:** Post-Dramaturg review of the Aletheia memory system design document. The Dramaturg session had produced a finalized design with 15+ settled decisions and full reconciliation. A subsequent review session using a Claude teammate (behavioral/usage perspective) + Gemini (technical architecture) + user (constraints/intent) discovered 3 critical flaws invisible to the original session.

## The Problem

The Dramaturg session is susceptible to **anchoring bias**. By the time the design document is finalized, the Dramaturg has spent the entire session building the mental model that justifies its decisions. It cannot effectively review its own work because it shares the same assumptions that produced the flaws. This is the "marking your own homework" problem.

Evidence from Aletheia:
- The Dramaturg session decided on a separate sidecar process, anchored by its own WAL research. A fresh Gemini review immediately identified the multi-process collision flaw.
- The Dramaturg session designed OCC with version_id. Neither the session's Gemini consultation nor the user considered context compaction. A fresh Claude reviewer flagged it as breaking in minutes.
- The Dramaturg session designed promote_to_memory as the primary journal-to-memory path. A fresh Claude instance doing honest behavioral self-assessment rated it at ~55-65% reliability — something the original session couldn't assess because it was the designer, not the user.

**Key insight:** The flaws weren't in obscure edge cases. They were in core assumptions that the Dramaturg session had internalized and couldn't question. Fresh perspectives found them quickly because they weren't anchored by those assumptions.

## The Pattern: Three-Agent Design Review

### Participants
1. **Dramaturg** — the session that produced the design. Acts as coordinator, relay, and domain expert. Has full context.
2. **Review Teammate** — a fresh Claude instance spawned with ONLY the design document as context. No conversation history, no decision journal, no awareness of the design process. This lack of context is the feature, not a limitation.
3. **User** — participates interactively in the discussion, providing constraints, empirical evidence, and decisions. Not a passive reviewer.

### Why Teammate, Not Separate Session
- A separate Claude Code session requires the user to manage two terminals, two contexts, and manually relay information between them.
- A teammate within the Dramaturg session allows the Dramaturg to act as the relay/coordinator, making it a seamless 3-agent discussion.
- The Dramaturg has full context to interpret and contextualize the teammate's findings against the decision journal.
- The user participates in one conversation, not two.

### Flow

```
Phase 6 (Review Loop) completes → all sections approved → before Reconciliation:

1. Dramaturg spawns review teammate(s) with ONLY the design document
   - No decision journal (prevents anchoring to the "why" behind decisions)
   - No conversation history (prevents inheriting assumptions)
   - Just the artifact as a cold reader would encounter it

2. Teammate produces independent review
   - From the perspective of the AI that would USE this system (if applicable)
   - From a technical architecture perspective
   - Focusing on: breaking issues, oversights, assumption flaws

3. Dramaturg presents findings to user
   - Organized by severity
   - Cross-referenced against decision journal (Dramaturg knows the "why")
   - Highlights where the reviewer contradicts a settled decision (and why)

4. Three-agent discussion on each finding
   - User provides constraints/evidence the reviewer didn't have
   - Dramaturg provides decision context
   - Reviewer provides independent assessment
   - Resolution is collaborative, not top-down

5. Decisions documented in journal as Phase 9 (or similar post-review phase)
```

### What Makes This Different from Phase 6 Review

Phase 6 (Review Loop) is the Dramaturg presenting its OWN work to the user for approval. The user reviews what the Dramaturg wrote. Both parties share the session's accumulated context and assumptions.

The Fresh Eyes pattern introduces a third party that does NOT share those assumptions. The value is specifically in the lack of shared context — the reviewer questions things the Dramaturg and user have stopped questioning.

## Teammate Configuration

### The Behavioral Reviewer (Claude Teammate)
**Purpose:** Review from the perspective of the AI that would use the designed system.
**Prompt pattern:** Give the teammate the design document and ask it to review AS IF it were the user of the tools/system being designed. Key questions:
- "Would I actually use this consistently?"
- "What's the cognitive overhead?"
- "What happens when my context gets compacted?"
- "What will I default to under time pressure?"

**Why this works:** Claude reviewing its own tool interfaces produces uniquely honest behavioral self-assessment. The Dramaturg session can't do this because it's the designer — it sees the tools as intended. The reviewer sees them as they'd be experienced.

**Evidence:** The Aletheia reviewer's honest self-assessments (~55-65% for promotion, ~15-20% for proactive deletion, ~70% for reactive cleanup) were the most actionable data in the entire review. They directly drove the "Dumb Capture, Smart Digest" redesign.

### The Technical Reviewer (Gemini via MCP)
**Purpose:** Technical architecture review — infrastructure, concurrency, failure modes, scalability.
**Already available:** Gemini is a mandatory Dramaturg dependency. The enrichment is using it specifically as a FRESH reviewer of the finalized design, not as a research tool during design.

**Key difference from research-phase Gemini usage:** During the Approach Loop, Gemini answers specific research questions in the Dramaturg's frame. During Fresh Eyes review, Gemini reviews the COMPLETE design as an independent architect. Different prompt, different value.

### The Interactive Brainstorm Pattern
**For complex issues:** Rather than getting a one-shot review from Gemini, the Dramaturg can act as middleman in an interactive brainstorm between Gemini and the user. The Dramaturg:
- Sends the issue to Gemini with full context
- Presents Gemini's analysis to the user
- Relays the user's response/constraints back to Gemini
- Iterates until resolution

This worked exceptionally well in the Aletheia review for the architecture (sidecar vs daemon vs hybrid) and OCC (monolithic vs hybrid strategy) discussions.

## When to Use

### Always Recommended
- Designs for systems that Claude will interact with as a tool user (MCP servers, hook systems, tool interfaces). The behavioral self-assessment is uniquely valuable here.
- Designs with concurrency, multi-process, or distributed system concerns. Fresh technical review catches assumption-based architecture flaws.
- Designs that went through significant evolution during the session (many approach changes, vision revisions). More evolution = more accumulated assumptions = more value from fresh eyes.

### Recommended for Complex Designs
- Designs with 4+ settled approaches that interact with each other. Interaction effects are invisible to individual approach reviews.
- Designs where the user provided empirical evidence ("I tried X and it didn't work") that may not be fully reflected in the design. The reviewer can catch when the design doesn't account for the user's experience.

### Optional / Skip
- Simple designs with 1-2 approaches and no concurrency/multi-agent concerns.
- Designs where the user has deep domain expertise and has already caught the major issues during Phase 6.

## Scaling Pattern: Per-Domain Teammates

For large designs, multiple review teammates can be spawned in parallel, each focused on a specific domain:
- One teammate per major design section
- One cross-cutting teammate for interaction effects between sections
- Results synthesized by the Dramaturg before presenting to user

For even larger designs (e.g., a system with many tag domains or entry types), teammates can be further specialized by tag/topic type, with a coordinator handling cross-domain concerns. This narrows each teammate's scope for more focused analysis.

## Results from Aletheia Session

| Finding | Found By | Original Session Missed It Because |
|---------|----------|-----------------------------------|
| Multi-process SQLite collision | Gemini (fresh) | Dramaturg anchored by its own WAL research |
| OCC + context compaction | Claude teammate | Neither Dramaturg nor Dramaturg's Gemini modeled LLM behavior |
| promote_to_memory unreliability | Claude teammate | Dramaturg was the designer, not the user |
| Tool count concerns | Claude teammate | Dramaturg accepted tool count based on deferred loading |
| Data lifecycle gap | Both | Dramaturg internalized the "TTL rejected" decision |
| Hook latency concerns | Gemini (fresh) | Dramaturg didn't model latency as user concern |

3 of 3 critical flaws were found. All were invisible to the original session. All required either fresh technical perspective or honest behavioral self-assessment that the designing session couldn't provide.

## Integration Recommendation

This pattern should be integrated into the Dramaturg skill as an optional phase between Review Loop (Phase 6) completion and Reconciliation (Phase 7). Suggested name: **Phase 6.5 — Fresh Eyes Review** or incorporate into an expanded Phase 6 as a post-approval review step.

The phase should:
1. Be recommended (not mandatory) based on design complexity heuristics
2. Use teammate spawning, not separate sessions
3. Include both behavioral (Claude teammate) and technical (Gemini) perspectives
4. Be interactive (user participates), not passive (user reviews conclusions)
5. Document findings as journal entries that feed into Reconciliation


---

## 2026-04-27 — Integration Note

This pattern has been folded into the Dramaturg skill at `skills/dramaturg/references/fresh-eyes-review.md` as **Phase 6.5 — Fresh Eyes Review**, an optional phase between Review Loop (Phase 6) and Reconciliation (Phase 7) gated by explicit, binary, auditable evaluation criteria. The teammate-within-session framing (preserved from this doc) carries forward; the skill's Phase Registry routes through Phase 6.5 for complex designs per the criteria.

This working doc remains the long-form discovery record. Future Dramaturg sessions reach for the integrated reference; future skill-rework cycles reach for this discovery doc when revisiting the pattern's rationale.
