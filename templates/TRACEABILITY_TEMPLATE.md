---
title: Traceability Matrix Template
description: Map requirements to implementation. Prevents drift and makes gaps obvious.
category: templates
---

# Traceability Matrix Template

A traceability matrix maps requirements to implementation. It prevents drift and makes gaps obvious.

Without explicit traceability, requirements get lost as context grows. A single coverage map catches gaps before they become bugs.

---

## Conventions

- `NS-#` for North Star items
- `REQ-#` for requirements
- `AC-#` for acceptance criteria
- Beads use bead IDs (e.g., `bd-123.2`)
- Tests use file paths or test identifiers
- Status: `planned` > `in_progress` > `code_done` > `tests_done` > `accepted`

---

## Template

Copy this into your project as `PLAN/05_traceability.md`:

```markdown
# Traceability Matrix

## Scope
- Project:
- Date:
- Rigor Tier:

---

## Coverage Matrix

| ID | Requirement | Phase | Beads | Tests | Acceptance | Status | Evidence |
|----|-------------|-------|-------|-------|------------|--------|----------|
| NS-1 | [North Star bullet] | Phase 0 | plan.0.1 | [tests] | AC-1 | planned | [link] |
| REQ-3 | [Requirement text] | Phase 2 | bd-123.2 | tests/auth_sso.py | AC-7 | tests_done | run: #127 |

---

## Gap Check

| Pattern | Symptom | Fix |
|---------|---------|-----|
| Orphan requirement | REQ with no beads | Create bead or mark REQ as deferred |
| Untested bead | Bead with no tests | Add tests before closing |
| Unverified AC | AC with no evidence | Run verification, add evidence |
| Scope creep | Bead not linked to any REQ | Either link to REQ or question if needed |
| Missing P0 coverage | P0 REQ without `accepted` status | Block release until verified |
```

---

## When to Update

| Trigger | Action |
|---------|--------|
| After `/decompose-task` | Add bead IDs to REQ rows |
| After creating tests | Add test paths to bead rows |
| After closing a bead | Update Status + Evidence |
| During `/calibrate` | Scan for gaps |
| After requirements change | Add new rows, mark old as deprecated |

---

## Agent Prompt

Use this to have an agent check traceability:

```
Review the traceability between my requirements and implementation:

Requirements: [paste or reference PLAN/01_requirements.md]
Beads: [paste bd list output or reference .beads/]
Tests: [describe test coverage or list test files]

For each requirement:
1. Is there a bead that implements it?
2. Is there a test that verifies it?
3. Is there evidence the test passed?

Flag any gaps:
- Requirements without implementation
- Implementation without tests
- Tests without passing evidence

Output a gap report with specific fixes for each issue.
```
