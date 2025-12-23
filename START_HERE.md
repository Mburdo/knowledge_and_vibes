<div align="center">

# Start Here

### The fastest path to understanding and using Knowledge & Vibes

</div>

---

## What This Framework Enables

Knowledge & Vibes gives you a structured system for building software with AI:

| Challenge | How This Framework Addresses It |
|:----------|:--------------------------------|
| Keeping AI aligned with your goals | Explicit North Star cards anchor intent |
| Managing complex, multi-step projects | Beads track tasks with dependencies |
| Coordinating multiple agents | Agent Mail prevents conflicts |
| Catching issues before they compound | Verification gates and calibration checkpoints |
| Maintaining evidence that things work | TDD-first with mandatory test coverage |

> **The core insight:** Truth lives outside the model. Tests that pass, code that compiles, documentation that exists: these are truth. Everything else is hypothesis requiring verification.

---

## The Solution in 60 Seconds

1. **Plan explicitly**: North Star cards, requirements with acceptance criteria, recorded decisions
2. **Track everything**: Beads (tasks) with dependencies, status, verification requirements
3. **Coordinate agents**: File reservations, claim/close announcements, message routing
4. **Verify continuously**: Tests before code (TDD), security scans before commit, calibration between phases
5. **Bound iteration**: Max 3 repair attempts, then decompose or escalate

**The result:** A repeatable process that produces working software.

---

## Reading Order

<table>
<tr>
<td width="50%" valign="top">

### Phase 1: Get Running

| Document | Goal |
|:---------|:-----|
| [**Setup Guide**](./docs/guides/SETUP_GUIDE.md) | Install all tools, verify they work |
| [**Glossary**](./GLOSSARY.md) | Bookmark for unfamiliar terms |

*After this:* You can run `bd`, `bv --robot-next`, `ubs`, `cass`, and `cm`

</td>
<td width="50%" valign="top">

### Phase 2: Understand the System

| Document | Goal |
|:---------|:-----|
| [**Evidence-Based Guide**](./docs/workflow/EVIDENCE_BASED_GUIDE.md) | See the complete 10-stage pipeline |
| [**Protocols**](./docs/workflow/PROTOCOLS.md) | Learn the 18 procedures |
| [**Philosophy**](./docs/workflow/PHILOSOPHY.md) | Understand why verification matters |

*After this:* You understand how the system works

</td>
</tr>
<tr>
<td width="50%" valign="top">

### Phase 3: Learn to Use It

| Document | Goal |
|:---------|:-----|
| [**Tool Stack & Operations**](./docs/guides/TOOL_STACK_AND_OPERATIONS.md) | What each tool and command does |
| [**Tutorial**](./docs/guides/TUTORIAL.md) | Detailed walkthroughs with examples |

*After this:* You can run `/prime`, `/next-bead`, `/calibrate`

</td>
<td width="50%" valign="top">

### Phase 4: Deep Dives (As Needed)

| Document | When to Read |
|:---------|:-------------|
| [**Planning Deep Dive**](./docs/workflow/PLANNING_DEEP_DIVE.md) | Starting a new project |
| [**Decomposition**](./docs/workflow/DECOMPOSITION.md) | Breaking plans into tasks |
| [**Research**](./research/README.md) | Understanding why protocols exist |

</td>
</tr>
</table>

---

## If You Only Read 3 Things

| Priority | Document | Why |
|:--------:|:---------|:----|
| 1 | [**Setup Guide**](./docs/guides/SETUP_GUIDE.md) | Get the tools working |
| 2 | [**Evidence-Based Guide**](./docs/workflow/EVIDENCE_BASED_GUIDE.md) | The complete pipeline |
| 3 | [**Protocols**](./docs/workflow/PROTOCOLS.md) | The exact moves to make |

Everything else is reference material.

---

## The Core Principles

<table>
<tr>
<td width="50%" valign="top">

### 1. Truth Lives Outside the Model

AI output is not truth. Truth is:
- Tests that pass or fail
- Code that compiles or doesn't
- Documentation that exists or doesn't

Every claim must be verified, not trusted.

### 2. Tests Adjudicate, Not Rhetoric

When agents disagree, write tests that distinguish the approaches. Run the tests. Let results decide.

Tests are the tiebreaker.

</td>
<td width="50%" valign="top">

### 3. Decompose When Reality Demands It

Don't pre-plan everything. Start with coarse tasks (beads). Attempt execution. If it fails after 3 tries, *then* decompose. Only the failing part.

### 4. Minimal Viable Planning

A 200-line script doesn't need a 50-page plan. Match plan size to project complexity.

"Lossless" means no guessing required, not "exhaustively long."

</td>
</tr>
</table>

---

## What a Session Looks Like

```bash
# Start session
/prime                      # Register, check inbox, discover tasks

# Claim work
/next-bead                  # Find task, reserve files, announce [CLAIMED]

# Implement (TDD-first)
pytest                      # Run tests (expect red)
# ... write implementation ...
pytest                      # Run tests (expect green)
ubs --staged                # Security scan (mandatory)

# Close work
git add -A && git commit
bd close <id> --reason "..."
# Agent releases files, sends [CLOSED]

# Between phases
/calibrate                  # Hard stop, check for drift
```

---

## Quick Links

| I want to... | Go to... |
|:-------------|:---------|
| Install everything | [**Setup Guide**](./docs/guides/SETUP_GUIDE.md) |
| Look up a term | [**Glossary**](./GLOSSARY.md) |
| See all templates | [**TEMPLATES.md**](./TEMPLATES.md) |
| Understand the research | [**Research**](./research/README.md) |
| Start a new project | [**Setup Guide**](./docs/guides/SETUP_GUIDE.md) then [**North Star Template**](./templates/NORTH_STAR_CARD_TEMPLATE.md) |
| Migrate an existing project | [**Migration Guide**](./docs/guides/MIGRATION_GUIDE.md) |
