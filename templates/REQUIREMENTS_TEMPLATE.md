---
title: Requirements Template
description: Define what must be true when done. REQ-* for requirements, AC-* for acceptance criteria.
category: templates
stage: 1
---

# Requirements Template

Requirements define what must be true when you're done. They describe outcomes and constraints, not solutions.

If you can't test it, it's not a requirement. It's a wish.

---

## Conventions

- `REQ-001`, `REQ-002`, etc. for requirements
- `AC-001.1`, `AC-001.2`, etc. for acceptance criteria
- `P0` = must have, `P1` = should have, `P2` = nice to have

---

## Template

Copy this into your project as `PLAN/01_requirements.md`:

```markdown
# Requirements

## Scope
- Project:
- Date:
- Rigor Tier:

---

## REQ-001 (P0): [Short title]

**What must be true:**
[Observable outcome or constraint]

**Acceptance Criteria:**
- AC-001.1: [How you'll prove it's true]
- AC-001.2: [Another way to prove it]

**Assumptions:**
- [Any assumptions, labeled HIGH/MED/LOW confidence]

**Dependencies:**
- [External systems, APIs, data sources]

---

## REQ-002 (P1): [Short title]

**What must be true:**
[Observable outcome or constraint]

**Acceptance Criteria:**
- AC-002.1: [How you'll prove it's true]

---

## Open Questions
- [Things that still need clarification]
```

---

## Agent Prompt

Use this to have an agent draft requirements from your North Star:

```
Draft requirements for my project based on this North Star Card:

[paste your North Star Card]

Rules:
1. Requirements describe outcomes, not solutions
2. Every P0 requirement needs at least one testable acceptance criterion
3. If something is an assumption, label it (HIGH/MED/LOW confidence)
4. If you need clarification to write a requirement, ask me

Output a requirements list using the REQ/AC format.
```
