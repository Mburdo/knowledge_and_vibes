---
title: Evidence-Based Guide
description: The operational manual for Knowledge & Vibes - a gated, artifact-driven pipeline that maximizes the probability of correct outcomes.
category: workflow
---

<div align="center">

# Evidence-Based Guide

### The Complete Pipeline

</div>

The operational manual for Knowledge & Vibes, a gated, artifact-driven pipeline that maximizes the probability of correct outcomes.

> **Evidence-based means:** Every stage is justified by research. Every gate exists because skipping it causes predictable issues.

---

## 📖 Table of Contents

| Section | What You'll Learn |
|:--------|:------------------|
| [The Six Principles](#-the-six-principles) | The core ideas that shape everything |
| [Roles](#-roles-who-does-what) | What operators and agents each handle |
| [The Tool Stack](#-the-tool-stack) | What each tool addresses |
| [The 10-Stage Pipeline](#-the-10-stage-pipeline) | The complete workflow from idea to ship |
| [What To Read Next](#-what-to-read-next) | Where to go from here |

---

## 💡 The Six Principles

<table>
<tr><td>

### 0. Give AI Few Decisions

**Plan as much as appropriate. Give the AI as few decisions as possible.**

Any decision you don't claim, you implicitly delegate. Gaps become assumptions. Assumptions become architecture. Bad architecture becomes a rewrite.

**Implication:** The plan is complete when there's nothing left to interpret. See `DISCOVERY.md`.

</td></tr>
<tr><td>

### 1. Truth Lives Outside the Model

The AI's confident output is not truth. Truth is:
- Tests that pass or fail
- Code that compiles or doesn't
- Documentation that exists or doesn't
- Measurements you can observe

**Implication:** Every claim that matters must be verified, not trusted.

</td></tr>
<tr><td>

### 2. Planning is Search + Selection

Don't bet on one plan. Generate multiple candidates, then select using evidence.

**Implication:** Stages 3-5 produce options before committing.

</td></tr>
<tr><td>

### 3. Tests Adjudicate Disagreements

When agents disagree, the answer is not more debate.

**Implication:** Write discriminating tests, run them, let results decide.

</td></tr>
<tr><td>

### 4. Minimal Plans Beat Massive Plans

> "The smallest set of high-signal tokens that enables correct behavior." — Anthropic

**Implication:** Plan size matches project complexity. "Lossless" = no guessing, not "huge."

</td></tr>
<tr><td>

### 5. Decompose When Reality Demands It

Don't pre-decompose "just in case." Start coarse, split only when execution fails.

**Implication:** ADaPT pattern in Stage 7-8.

</td></tr>
</table>

---

## 👤 Roles: Who Does What

<table>
<tr>
<td width="50%" valign="top">

### Operator (You)

You do **value decisions** and **gate enforcement**:

| Responsibility | Why It's Yours |
|:---------------|:---------------|
| Define goals, stakes, priorities | Only you know what success means |
| Answer clarifying questions | Only you know what you actually want |
| Choose between options | Only you can make value tradeoffs |
| Enforce "no tests = not done" | Core quality gate |
| Approve plan changes | Drift must be intentional |

You do NOT need to: validate code, review implementations, debug technical problems, or decide architecture.

</td>
<td width="50%" valign="top">

### Agents

Agents do **technical work + evidence production**:

| Responsibility | Why Agents |
|:---------------|:-----------|
| Convert intent to `REQ-*` / `AC-*` | Technical translation |
| Ground claims (Warp-Grep, Exa) | Access to verification tools |
| Implement in verifiable beads | Technical execution |
| Run verification loops | Automated checking |
| Coordinate via Agent Mail | Multi-agent orchestration |
| Maintain traceability | Coverage tracking |

</td>
</tr>
</table>

---

## 🛠 The Tool Stack

Each tool addresses a specific concern:

| Tool | Command | Addresses |
|:-----|:--------|:----------|
| **Beads** | `bd` | Work tracking, dependency management |
| **BV** | `bv --robot-*` | Execution ordering, blocker detection |
| **Agent Mail** | MCP tools | Multi-agent coordination, file reservations |
| **Warp-Grep** | MCP tool | Codebase truth |
| **Exa** | MCP tool | Web truth (current docs, APIs) |
| **UBS** | `ubs --staged` | Security scanning (mandatory) |
| **CASS/cm** | `cass`, `cm` | History, learned patterns |

📖 **For detailed usage:** See [Tool Stack & Operations](../guides/TOOL_STACK_AND_OPERATIONS.md)

---

## 📋 The 10-Stage Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   Stage 0: North Star        → What are we building? Why?                  │
│   Stage 1: REQ/AC            → What must be true when done?                │
│   Stage 2: Requirements QA   → Is it unambiguous?                          │
│   Stage 3: Decision Search   → What are the options?                       │
│   Stage 4: Risks & Spikes    → What could go wrong?                        │
│   Stage 5: Plan Pack         → Complete spec for execution                 │
│   Stage 6: Phase Breakdown   → Manageable chunks                           │
│   Stage 7: Bead Decomposition → Executable tasks                           │
│   Stage 8: Execution         → Build with verification                     │
│   Stage 9: Calibration       → Check and correct                           │
│   Stage 10: Release          → Ship with confidence                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

> **Critical rule:** Don't skip stages. The gates exist because skipping them causes predictable issues.

---

### Stage 0 — North Star (Ground Truth)

| | |
|:--|:--|
| **Purpose** | Establish what we're building and why before anyone writes code |
| **Artifact** | `templates/NORTH_STAR_CARD_TEMPLATE.md` pinned at top of plan |
| **Gate** | Card is explicit enough that agents can resolve tradeoffs without guessing |

**Must include:**
- Success metrics (how will you know it worked?)
- Stakes (what happens if it fails?)
- Non-goals (what are we explicitly NOT doing?)
- Stop/ask rule (when should agents pause for human input?)

---

### Stage 1 — Requirements That "Compile" (REQ/AC)

| | |
|:--|:--|
| **Purpose** | Convert intent into testable requirements |
| **Artifact** | `PLAN/01_requirements.md` |
| **Template** | `templates/REQUIREMENTS_TEMPLATE.md` |
| **Gate** | Every P0 requirement has at least one falsifiable AC |

**Contains:**
- `REQ-*` — Observable outcomes or constraints
- `AC-*` — Acceptance criteria (ideally test-shaped)
- Priority (P0/P1/P2)

> If you can't test it, it's not a requirement, it's a wish.

---

### Stage 2 — Requirements QA

| | |
|:--|:--|
| **Purpose** | Eliminate ambiguity before it becomes expensive rework |
| **Artifact** | Updated requirements |
| **Template** | `templates/REQUIREMENTS_QA_TEMPLATE.md` |
| **Gate** | No P0 requirement remains ambiguous or untestable |

---

### Stage 3 — Decision Search

| | |
|:--|:--|
| **Purpose** | Don't bet on one approach. Generate options, then select with evidence. |
| **Artifact** | `PLAN/02_decisions_adrs.md` |
| **Template** | `templates/DECISIONS_ADRS_TEMPLATE.md` |
| **Gate** | Every architecturally significant requirement is decided or "TBD pending spike" |

**ADRs contain:**
- Options A/B/C
- Tradeoffs for each
- Risks
- Reversal triggers (what would change our mind)

---

### Stage 4 — Risks & Spikes

| | |
|:--|:--|
| **Purpose** | Identify unknowns and resolve them before committing to a path |
| **Artifact** | `PLAN/06_risks_and_spikes.md` |
| **Template** | `templates/RISKS_AND_SPIKES_TEMPLATE.md` |
| **Gate** | High-risk unknowns are converted into spikes with explicit outputs |

---

### Stage 5 — Plan Pack (Lossless Specs)

| | |
|:--|:--|
| **Purpose** | Produce specs complete enough that implementation requires no guessing |
| **Gate** | A competent engineer could implement without guessing |

**Artifacts:**
- `PLAN/03_system_map.md`
- `PLAN/04_data_model.md`
- `PLAN/05_api_and_error_model.md`
- `PLAN/08_testing_spec.md`
- `PLAN/09_ops_security.md`
- `PLAN/10_dependencies_and_phases.md`

> Plan size should match project complexity. "Lossless" means no guessing, not "comprehensive."

---

### Stage 6 — Phase Breakdown

| | |
|:--|:--|
| **Purpose** | Break the plan into chunks that fit in working memory without loss |
| **Artifact** | Phase documents using `.claude/templates/planning/phase-document.md` |
| **Heuristic** | ~500-1000 lines per phase (based on semantic coherence) |
| **Gate** | No phase is large enough to cause silent omission |

---

### Stage 7 — Bead Decomposition (TDD-First)

| | |
|:--|:--|
| **Purpose** | Convert phases into executable tasks with built-in verification |
| **Template** | `.claude/templates/beads/bead-structure.md` |
| **Command** | `/decompose-task [phase]` |
| **Gate** | Every bead has tests in the description before implementation begins |

**Beads contain:**
- **Tests FIRST** (written before implementation)
- Edit locus (exactly what files/functions to change)
- REQ/AC linkage
- Verification plan

**Key rules:**
- **TDD-first:** Tests before code
- **Adaptive decomposition:** Start coarse, split only when execution fails

```bash
# Decompose a phase into beads
/decompose-task [phase]

# Validate the dependency graph
bv --robot-suggest   # Missing deps, duplicates
bv --robot-plan      # Sane execution order?
bv --robot-alerts    # Drift signals, risky topology
```

---

### Stage 8 — Execution Loops

| | |
|:--|:--|
| **Purpose** | Implement with verification, not hope |
| **Gate** | No bead closes without all tests passing and `ubs --staged` clean |

**Key rules:**
- **Max 3 repair iterations** — Fail fast, escalate early
- **If still failing after 3:** STOP, spawn sub-bead for failing part, notify operator

```bash
# Session startup
/prime                           # Register with Agent Mail, check inbox

# Find and claim work
/next-bead                       # Full discovery + claim flow

# During work
pytest                           # Run tests
ubs --staged                     # Security check (mandatory)

# Close work
bd close <id>.1 --reason "..."   # Close sub-beads FIRST
bd close <id> --reason "..."     # Then parent
```

---

### Stage 9 — Calibration

| | |
|:--|:--|
| **Purpose** | Check for drift and correct course before continuing |
| **Command** | `/calibrate [phase_or_milestone]` |
| **Gate** | No next phase begins until calibration produces falsifiable decisions |

**Key rules:**
- Disagreements must be resolved by **tests**, not rhetoric
- If tests don't discriminate, preserve both positions for human decision
- Update plan with any changes

---

### Stage 10 — Release Readiness

| | |
|:--|:--|
| **Purpose** | Ensure "done" means "actually shippable" |
| **Gate** | System meets success metrics at chosen rigor tier |

**Release checklist:**
- All tests passing
- Security scan clean
- Observability in place
- Rollback plan documented

**Mandatory gates:**
- **Security Gate:** `ubs --staged` clean
- **Human Verification:** Operator sign-off on P0 implementations

---

## 📚 What To Read Next

| If you want... | Read... |
|:---------------|:--------|
| Pre-pipeline discovery (start here for new projects) | [DISCOVERY.md](./DISCOVERY.md) |
| Exact protocol checklists | [PROTOCOLS.md](./PROTOCOLS.md) |
| Deep planning tutorial | [PLANNING_DEEP_DIVE.md](./PLANNING_DEEP_DIVE.md) |
| Decomposition mechanics | [DECOMPOSITION.md](./DECOMPOSITION.md) |
| Tool operations & multi-agent | [Tool Stack & Operations](../guides/TOOL_STACK_AND_OPERATIONS.md) |
| Research summaries | [research/README.md](../../research/README.md) |
| Why this approach works | [PHILOSOPHY.md](./PHILOSOPHY.md) |

---

## Key Refrains

**Plan as much as appropriate. Give the AI as few decisions as possible.**

**The plan is complete when there's nothing left to interpret.**

**Same hours. Different outcomes.**
