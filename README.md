# Dramaturg

A research-augmented design skill for Claude Code. The Dramaturg explores what *could* be built through fluid discussion with active research — validating recommendations before presenting them and checking user proposals for feasibility in real-time. Produces a design document through conversational, research-backed exploration.

## Pipeline Position

```
dramaturg → arranger → conductor → musician
(vision)    (plan)     (coordination)  (implementation)
                        copyist creates individual parts from arranger's score
```

The Dramaturg produces a design document that the Arranger consumes to create an executable implementation plan. The user controls the pipeline — no automatic handoff.

## Repository Structure

```
dramaturg/
├── README.md
├── docs/
│   └── designs/
│       └── 2026-02-17-dramaturg-skill-design.md   ← Authoritative design document
└── skill/
    ├── SKILL.md                                    ← Hub file (Tier 3)
    └── references/
        ├── support-phases.md                       ← Phases 1, 4, 7, 8
        ├── vision-loop.md                          ← Phase 2
        ├── vision-expansion.md                     ← Phase 3 (enrichment and proactive ideation)
        ├── approach-loop.md                        ← Phase 5 (core research-discuss-settle loop)
        ├── review-loop.md                          ← Phase 6
        └── research-strategy.md                    ← Research tool hierarchy and delegation
```

## Installation

Copy the skill directory to the Claude Code skills location:

```bash
cp -r skill/ ~/.claude/skills/dramaturg/
```

## Status

- Design document: complete
- SKILL.md: complete, reviewed
- Reference files: complete
- Deployment: staged (not yet deployed)

## Authority

- **SKILL.md** is the authoritative runtime document — Claude follows it during sessions
- **Design documents** (`docs/designs/`) record the rationale and decisions that shaped SKILL.md
- If SKILL.md and a design document disagree, SKILL.md wins — design docs may be outdated
