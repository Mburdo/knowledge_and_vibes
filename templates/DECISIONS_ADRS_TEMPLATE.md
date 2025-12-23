---
title: Decisions Template (ADRs)
description: Capture significant decisions - what you decided, alternatives, and what would change your mind.
category: templates
stage: 3
---

# Decisions Template (ADRs)

ADRs (Architecture Decision Records) capture significant decisions so they don't get relitigated later.

The goal is to record what you decided, what the alternatives were, and what would make you change your mind.

---

## Template

Copy this into your project as `PLAN/02_decisions.md`:

```markdown
# Decisions

## ADR-001: [Decision Title]

**Status:** proposed / accepted / superseded
**Date:**

### Context
What problem are we solving? What constraints apply?

### Options

| Option | Summary | Pros | Cons |
|:-------|:--------|:-----|:-----|
| A | ... | ... | ... |
| B | ... | ... | ... |
| C | ... | ... | ... |

### Decision
Which option did we choose?

### Rationale
Why this option? How does it align with the North Star?

### Consequences
What does this decision imply for the rest of the system?

### Reversal Triggers
What would make us reconsider? (Metrics, test failures, new information)

---

## ADR-002: [Decision Title]

...
```

---

## Agent Prompt

Use this when you need help making a decision:

```
I need to make a decision about:
[describe the decision]

Context:
[relevant constraints, requirements, or background]

North Star priority order:
[e.g., Safety > Correctness > UX > Speed]

Help me think through this:
1. What are 2-3 realistic options?
2. What are the tradeoffs of each?
3. Which option best fits my priorities?
4. What would make us reconsider this decision later?

If you need more information to give good options, ask me.
```
