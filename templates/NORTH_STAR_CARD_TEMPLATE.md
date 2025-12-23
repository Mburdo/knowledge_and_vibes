---
title: North Star Card Template
description: Anchors everything - what success looks like, what's out of scope, and when to stop and ask.
category: templates
stage: 0
---

# North Star Card

The North Star Card anchors everything. Before anyone writes code, this document pins down what success looks like, what's out of scope, and when to stop and ask.

If the North Star is vague, agents will drift.

---

## Build Profile (Pick One)

| Profile | Context | Rigor | What "Good" Looks Like |
|:--------|:--------|:------|:-----------------------|
| **Personal / Hobby** | Solo, low stakes, learning | Tier 1 | Fast iteration, minimal architecture, smoke tests |
| **Startup MVP** | Early product, speed matters | Tier 2 | Critical-path tests, clear boundaries, pragmatic security |
| **Business Tool** | Used by a team, real impact | Tier 2-3 | Strong correctness, integration tests, observability |
| **Enterprise** | Compliance, audits, high stakes | Tier 3 | Traceability, threat modeling, formal change control |

---

## Rigor Tiers

| Tier | Focus | Minimum Bar |
|:-----|:------|:------------|
| **1** | Speed | Smoke tests on critical paths |
| **2** | Balanced | Unit + integration tests, ADRs for major decisions |
| **3** | Assurance | Full test matrix, threat model, traceability |

---

## Template

Copy this into your project as `PLAN/00_north_star.md`:

```markdown
# North Star Card

## The Goal
What does success look like? (One sentence, concrete)

## Who It's For
Primary user and the core problem you're solving.

## Context
What kind of project is this? (Startup MVP, internal tool, hobby project, etc.)

## Build Profile
Which profile from the table above? Why?

## Rigor Tier
1, 2, or 3? Why this level?

## Success Metrics
How will you know it worked? (Observable, ranked)

## Non-Goals
What are you explicitly NOT building?

## Constraints
Time, budget, tech stack, compliance requirements.

## Stop/Ask Rules
When should agents pause and ask instead of guessing?

## Open Questions
What still needs to be clarified?
```

---

## Agent Prompt

Use this to have an agent help draft your North Star Card:

```
Help me create a North Star Card for my project.

Here's what I'm trying to build:
[describe your project]

Here's the context:
[who it's for, why you're building it, any constraints]

Create a North Star Card that includes:
- A concrete one-sentence goal
- The build profile and rigor tier (with rationale)
- Success metrics I can actually measure
- Non-goals to prevent scope creep
- Stop/ask rules so agents know when to pause

Be direct. If something is unclear, ask me.
```
