<div align="center">

# Glossary

### Every term used in Knowledge & Vibes, defined

</div>

---

## 📖 Navigation

| Section | What's Covered |
|:--------|:---------------|
| [Core Concepts](#-core-concepts) | Beads, North Star, Lossless, Grounding, Calibration, Drift |
| [Requirements & Planning](#-requirements--planning) | REQ, AC, ADR, Rigor Tier, Priorities, Spikes, Gates |
| [Execution](#-execution) | TDD, 3-Iteration Limit, ADaPT, Verification |
| [Multi-Agent Coordination](#-multi-agent-coordination) | Agent Mail, File Reservations, Claims, Orchestrator Pattern |
| [Tools](#-tools) | bd, bv, ubs, cass, cm, MCP, Warp-Grep, Exa |
| [Slash Commands](#-slash-commands) | /prime, /next-bead, /calibrate, /decompose-task, /ground |
| [Phrases](#-phrases) | Common expressions and their meaning |

---

## 🎯 Core Concepts

<table>
<tr><td>

### Beads

The unit of work in this system. A bead is a task with a description, status, priority, dependencies, and verification requirements. Managed with the `bd` command.

Beads can have sub-beads (e.g., `bd-123.1`, `bd-123.2`). Sub-beads must complete before their parent closes.

</td></tr>
<tr><td>

### North Star Card

A one-page document that anchors the entire project. Contains: goal, success metrics, rigor tier, non-goals, and stop/ask rules.

> When in doubt, check the North Star. Everything traces back to it.

</td></tr>
<tr><td>

### Lossless

A plan or spec where nothing is left to interpretation. Another agent could implement it without asking clarifying questions.

"Lossless" means no guessing required—not "exhaustively long."

</td></tr>
<tr><td>

### Grounding

Verifying claims against real sources before acting on them. Three types:

| Type | Question | Tool |
|:-----|:---------|:-----|
| **Codebase truth** | What does the actual code say? | Warp-Grep |
| **Web truth** | What do current docs say? | Exa |
| **History truth** | What worked before? | CASS, cm |

</td></tr>
<tr><td>

### Calibration

A hard stop between phases to check alignment. Detects drift, resolves disagreements, updates the plan.

Triggered by `/calibrate`. Disagreements are resolved by tests, not debate.

</td></tr>
<tr><td>

### Drift

When implementation gradually diverges from the original plan. Causes: long context, forgotten requirements, accumulated decisions.

Calibration catches drift before it compounds.

</td></tr>
</table>

---

## 📋 Requirements & Planning

<table>
<tr><td>

### REQ (Requirement)

A numbered requirement describing what the system must do. Format: `REQ-001`, `REQ-002`.

Requirements describe **outcomes**, not solutions.

*Example:* `REQ-001: Users can reset their password via email`

</td></tr>
<tr><td>

### AC (Acceptance Criteria)

Testable conditions that prove a requirement is met. Format: `AC-001.1`, `AC-001.2`.

> If you can't test it, it's not an AC—it's a hope.

*Example:* `AC-001.1: Password reset email arrives within 30 seconds`

</td></tr>
<tr><td>

### ADR (Architecture Decision Record)

A log entry documenting a significant decision. Contains:
- What was decided
- Alternatives considered
- Why this option was chosen
- What would change our mind (reversal triggers)

</td></tr>
<tr><td>

### Rigor Tier

How much process a project needs:

| Tier | Use For | Testing |
|:-----|:--------|:--------|
| **Tier 1 (Speed)** | Prototypes, experiments | Smoke tests only |
| **Tier 2 (Balanced)** | Most projects | Unit + integration tests, ADRs |
| **Tier 3 (High Assurance)** | Regulated, high-stakes | Full test matrix, threat modeling |

</td></tr>
<tr><td>

### P0 / P1 / P2 (Priority)

| Level | Meaning |
|:------|:--------|
| **P0** | Must have. Project fails without it. |
| **P1** | Should have. Important but not blocking. |
| **P2** | Nice to have. Do if time permits. |

</td></tr>
<tr><td>

### Spike

A timeboxed investigation to reduce uncertainty. Run before committing to an approach.

*Example:* "Spike: Can we integrate with their API?" (output: yes/no + notes)

</td></tr>
<tr><td>

### Gate

A checkpoint that must pass before proceeding:

| Gate | Requirement |
|:-----|:------------|
| **Security gate** | `ubs --staged` clean |
| **Tests gate** | All tests pass |
| **Calibration gate** | Drift addressed |

</td></tr>
</table>

---

## ⚡ Execution

<table>
<tr><td>

### TDD (Test-Driven Development)

Writing tests before implementation. In this system, TDD is mandatory—tests go in the bead description before code is written.

</td></tr>
<tr><td>

### 3-Iteration Limit

Never attempt more than 3 repair cycles on failing code. After 3 failures: stop, decompose, or escalate.

</td></tr>
<tr><td>

### ADaPT (Adaptive Decomposition)

Only break tasks into smaller pieces when execution fails. Don't pre-decompose based on guesses.

**Flow:** Start coarse → Attempt execution → If fails after 3 tries → Decompose only the failing part

</td></tr>
<tr><td>

### Verification

Evidence that a bead is complete:
- Tests pass
- Security scan clean
- Traceability updated
- Commands and outputs recorded

> "It looks right" is not verification.

</td></tr>
</table>

---

## 👥 Multi-Agent Coordination

<table>
<tr><td>

### Agent Mail

The coordination system for multiple AI agents. Provides:
- Agent registration
- Messaging between agents
- File reservations
- Discovery of active agents

</td></tr>
<tr><td>

### Agent Name

A randomly generated identifier (e.g., "BlueLake", "GreenCastle"). Assigned at registration.

Names are memorable identifiers, not role descriptions.

</td></tr>
<tr><td>

### File Reservation

A lock on files or directories that prevents other agents from editing simultaneously. Set before editing, released when done.

Prevents merge conflicts and duplicate work.

</td></tr>
<tr><td>

### [CLAIMED] / [CLOSED]

Message subjects announcing when an agent starts or finishes a task:

| Message | Meaning |
|:--------|:--------|
| `[CLAIMED] bd-123 - Task title` | Work started |
| `[CLOSED] bd-123 - Task title` | Work completed |

All agents see these to avoid stepping on each other.

</td></tr>
<tr><td>

### Orchestrator-Worker Pattern

A coordination pattern where one agent (orchestrator) assigns work to others (workers) and synthesizes results.

</td></tr>
</table>

---

## 🛠 Tools

<table>
<tr><td>

### bd (Beads CLI)

Command-line tool for task management.

```bash
bd create "Task name"      # Create a bead
bd ready --json            # See unblocked beads
bd update ID --status ...  # Update a bead
bd close ID --reason "..." # Close a bead
bd dep add child blocker   # Add dependency
```

</td></tr>
<tr><td>

### bv (Beads Viewer)

Analyzes the bead graph and recommends next tasks.

```bash
bv --robot-next            # Single best next task
bv --robot-triage          # Full analysis
bv --robot-alerts          # Risks and stale items
```

⚠️ **Always use `--robot-*` flags. Bare `bv` hangs.**

</td></tr>
<tr><td>

### ubs (Universal Bug Scanner)

Security scanner that catches vulnerabilities before commit.

```bash
ubs --staged               # Scan staged changes
ubs --staged --fail-on-warning  # Strict mode
```

⚠️ **Mandatory before every commit.**

</td></tr>
<tr><td>

### cass (Coding Agent Session Search)

Searches past Claude Code sessions to find similar solutions.

```bash
cass search "query" --robot
cass timeline --today --json
```

⚠️ **Always use `--robot` or `--json`. Bare `cass` hangs.**

</td></tr>
<tr><td>

### cm (Context Memory)

Retrieves learned patterns and anti-patterns for your current task.

```bash
cm context "what I'm doing" --json
```

</td></tr>
<tr><td>

### MCP (Model Context Protocol)

A standard for giving AI assistants additional capabilities. MCP servers provide tools to Claude Code.

</td></tr>
<tr><td>

### Warp-Grep

MCP tool for fast parallel codebase search. Use to understand how things work across multiple files.

</td></tr>
<tr><td>

### Exa

MCP tool for web search. Use to verify documentation, find best practices, research libraries.

</td></tr>
</table>

---

## ⌨️ Slash Commands

| Command | Purpose |
|:--------|:--------|
| `/prime` | Start a new session. Registers with Agent Mail, checks inbox, discovers tasks. |
| `/next-bead` | Claim the next task. Handles: close current → discover → claim → reserve files → announce. |
| `/calibrate` | Run a calibration checkpoint. Checks coverage, detects drift, resolves disagreements with tests. |
| `/decompose-task` | Break a phase into beads and sub-beads. Creates the task graph for execution. |
| `/ground` | Verify claims against real sources before making decisions. |

---

## 💬 Phrases

<table>
<tr><td>

### "Truth lives outside the model"

AI output is not truth. Truth is tests that pass, code that compiles, behavior you can observe.

</td></tr>
<tr><td>

### "Tests adjudicate, not rhetoric"

When agents disagree, they write tests that distinguish the approaches. Test results decide—not arguments.

</td></tr>
<tr><td>

### "External feedback only"

Don't ask "are you sure?"—that makes the AI second-guess correct answers. Instead, run tests and use results as feedback.

</td></tr>
<tr><td>

### "Stop/ask rule"

A predefined trigger for when the AI should pause and ask the user. Defined in the North Star Card.

</td></tr>
<tr><td>

### "Context sufficiency"

Checking whether you have enough information before starting. If you'd have to guess, gather more context first.

</td></tr>
<tr><td>

### "Workflow beats prompting"

You can't prompt your way to reliability. You need a system where failures get caught before they matter.

</td></tr>
</table>
