---
title: Risks & Spikes Template
description: Timeboxed investigations that produce evidence to reduce uncertainty before committing.
category: templates
stage: 4
---

# Risks & Spikes Template

Spikes are timeboxed investigations that produce evidence. They're not features. They exist to reduce uncertainty before you commit to a path.

A spike ends with a decision, not code.

---

## Template

Copy this into your project as `PLAN/06_risks_and_spikes.md`:

```markdown
# Risks & Spikes

## Top Risks

What could cause rework, failure, or unexpected cost?

| Risk | Why It Matters | Likelihood | Impact | How to Resolve |
|:-----|:---------------|:-----------|:-------|:---------------|
| [Risk 1] | ... | L/M/H | L/M/H | Spike / Research / Accept |
| [Risk 2] | ... | L/M/H | L/M/H | Spike / Research / Accept |

---

## Spikes

### SPIKE-001: [Title]

**Goal:** What question are we trying to answer?

**Timebox:** 1-4 hours

**Steps:**
1. ...
2. ...
3. ...

**Output:** What evidence will this produce? (Docs, prototype, measurements)

**Decision it unlocks:** Which ADR or requirement does this inform?

---

### SPIKE-002: [Title]

...
```

---

## Spike Outcomes

A spike must end with one of:
- **Decision can be made** (evidence is sufficient)
- **Still ambiguous** (what's missing?)
- **Approach rejected** (why?)

---

## Agent Prompt

Use this to identify risks and plan spikes:

```
Review my project plan and identify the top risks:

North Star: [paste or summarize]
Requirements: [paste or summarize]
Proposed approach: [describe]

For each risk:
1. Why does it matter?
2. How likely is it?
3. Can we resolve it with a spike? If so, what would the spike look like?

Focus on things that could cause significant rework if we get them wrong.
```
