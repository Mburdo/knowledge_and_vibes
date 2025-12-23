<div align="center">

# Knowledge & Vibes

### The production-grade framework for multi-agent AI development

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Research-Backed](https://img.shields.io/badge/Research-73%20Papers-green.svg)](./research/README.md)
[![Claude Code](https://img.shields.io/badge/Built%20for-Claude%20Code-blueviolet.svg)](https://claude.ai)

---

**Build real software with AI—reliably.**

This framework turns AI-assisted coding from a gamble into a repeatable process.<br>
Every protocol is research-backed. Every tool catches a specific failure mode.<br>
The result: **a system where it's hard to be wrong.**

[Get Started](#-quick-start) · [Documentation](#-documentation) · [Research](./research/README.md) · [Follow @YachtsmanCap](https://x.com/YachtsmanCap)

</div>

---

## 🎯 What You Get

| Challenge | Solution |
|:----------|:---------|
| Keeping AI aligned with your intent | **North Star Cards** anchor goals before coding starts |
| Tracking complex, multi-step work | **Beads** manage tasks with dependencies and verification |
| Coordinating multiple agents | **Agent Mail** prevents conflicts with reservations and messaging |
| Catching security issues | **UBS** scans every commit automatically |
| Proving things actually work | **TDD-first protocol** with mandatory test coverage |
| Maintaining context across sessions | **CASS** searches past solutions; **cm** retrieves patterns |

> **Workflow beats prompting.** You can't prompt your way to reliability—but you can build a system where failures get caught before they matter.

---

## 🚀 Quick Start

### 1. Install the Tools

See the **[Setup Guide](./docs/guides/SETUP_GUIDE.md)** for complete installation instructions for all tools and MCP servers.

### 2. Initialize a Project

```bash
cd your-project
bd init
curl -o AGENTS.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/main/templates/AGENTS_TEMPLATE.md
git add .beads/ AGENTS.md && git commit -m "Initialize Knowledge & Vibes"
```

### 3. Run a Session

```bash
/prime              # Register agent, check inbox, discover tasks
/next-bead          # Claim work, reserve files, announce [CLAIMED]
# ... implement with TDD ...
ubs --staged        # Security scan (mandatory)
bd close <id>       # Complete task, release files, announce [CLOSED]
/calibrate          # Between phases: check for drift
```

---

## 💡 The Core Idea

> **Truth lives outside the model.**

The AI's confident output is not truth. Truth is tests that pass, code that compiles, documentation that exists. Everything else is a hypothesis requiring verification.

This framework enforces that distinction through:

| Principle | Implementation |
|:----------|:---------------|
| **Explicit artifacts** | North Star, requirements, decisions recorded *before* coding |
| **Mandatory verification** | Tests before implementation, security scans before commit |
| **Structured coordination** | File reservations, claim/close announcements, calibration gates |
| **Bounded iteration** | Max 3 repair attempts, then decompose or escalate |

---

## 📚 Documentation

<table>
<tr>
<td width="33%" valign="top">

### Start Here

| Doc | Learn |
|:----|:------|
| [**Setup Guide**](./docs/guides/SETUP_GUIDE.md) | Install & configure |
| [**START_HERE**](./START_HERE.md) | Reading order |
| [**Glossary**](./GLOSSARY.md) | Terms defined |

</td>
<td width="33%" valign="top">

### The Workflow

| Doc | Learn |
|:----|:------|
| [**Evidence-Based Guide**](./docs/workflow/EVIDENCE_BASED_GUIDE.md) | 10-stage pipeline |
| [**Protocols**](./docs/workflow/PROTOCOLS.md) | 18 procedures |
| [**Decomposition**](./docs/workflow/DECOMPOSITION.md) | Task breakdown |

</td>
<td width="33%" valign="top">

### Going Deeper

| Doc | Learn |
|:----|:------|
| [**Planning Deep Dive**](./docs/workflow/PLANNING_DEEP_DIVE.md) | Idea → plan |
| [**Philosophy**](./docs/workflow/PHILOSOPHY.md) | Why it works |
| [**Research**](./research/README.md) | 73 papers |

</td>
</tr>
</table>

---

## 🛠 The Toolkit

### CLI Tools

| Tool | Purpose | Key Command |
|:-----|:--------|:------------|
| **Beads** (`bd`) | Task tracking with dependencies | `bd ready --json` |
| **Beads Viewer** (`bv`) | Graph analysis, recommendations | `bv --robot-next` |
| **UBS** | Security scanner | `ubs --staged` |
| **CASS** | Session search | `cass search "query" --robot` |
| **cm** | Context memory | `cm context "task" --json` |

### MCP Servers

| Server | Purpose |
|:-------|:--------|
| **Agent Mail** | Multi-agent coordination, file reservations, messaging |
| **Warp-Grep** | Fast parallel codebase search |
| **Exa** | Web search for current documentation |

---

## 📋 Templates

| Template | When to Use |
|:---------|:------------|
| [**North Star Card**](./templates/NORTH_STAR_CARD_TEMPLATE.md) | Start of every project |
| [**Requirements**](./templates/REQUIREMENTS_TEMPLATE.md) | Defining what to build |
| [**Decisions (ADRs)**](./templates/DECISIONS_ADRS_TEMPLATE.md) | Recording architectural choices |
| [**AGENTS.md**](./templates/AGENTS_TEMPLATE.md) | Agent instructions for your repo |

📖 **Complete index:** [TEMPLATES.md](./TEMPLATES.md)

---

## 👥 Multi-Agent Coordination

```
┌─────────────────────────────────────────────────────────────────┐
│                     ORCHESTRATOR                                │
│   Assigns work · Resolves conflicts · Triggers calibration      │
└─────────────────────────┬───────────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Worker A │    │ Worker B │    │ Worker C │
    │ Track 1  │    │ Track 2  │    │ Track 3  │
    └──────────┘    └──────────┘    └──────────┘
          │               │               │
          └───────────────┼───────────────┘
                          ▼
                   Agent Mail
            (reservations, messages)
```

**The Protocol:**

1. **Reserve files before editing** — `file_reservation_paths()` prevents conflicts
2. **Announce claims** — `[CLAIMED] bd-123` tells others what's taken
3. **Announce completions** — `[CLOSED] bd-123` releases work
4. **Check inbox before claiming** — Avoid duplicate effort
5. **Calibrate between phases** — Catch drift before it compounds

📖 **Full pattern:** [Orchestrator-Worker Guide](./docs/guides/ORCHESTRATOR_SUBAGENT_PATTERN.md)

---

## 🔧 Troubleshooting

| Problem | Solution |
|:--------|:---------|
| `bd: command not found` | `export PATH="$HOME/.local/bin:$PATH"` |
| `bv` hangs | Use `--robot-*` flags (never bare `bv`) |
| CASS finds nothing | `cass index --full` |
| UBS errors | `ubs doctor --fix` |

**Health check:**
```bash
bd doctor && cm doctor && ubs doctor && cass health
```

---

## 📁 Repository Structure

```
knowledge_and_vibes/
├── docs/
│   ├── guides/           # Setup, tutorial, migration
│   └── workflow/         # Protocols, planning, philosophy
├── templates/            # North Star, requirements, ADRs
├── research/             # 73 paper summaries
└── .claude/
    ├── commands/         # /prime, /calibrate, /next-bead
    ├── rules/            # Safety guardrails
    ├── skills/           # On-demand playbooks
    └── templates/        # Runtime templates
```

---

## 👤 About

This framework is the distillation of three years of building with AI—starting from zero.

When GPT-3.5 launched, I was in high finance. I couldn't write a line of code. But I recognized immediately that AI was going to fundamentally change how things get built, and I wanted to be part of it.

So I went all in. Not with tutorials or bootcamps, but with a first principles approach: *What can these models actually do? Where do they fail? How do you extract every ounce of capability while catching the inevitable mistakes?*

The early days were rough. The models were weaker. The tooling didn't exist. Every technique had to be discovered through trial and error. But I stayed in it—learning, building, refining—session after session, project after project.

Three years later, I'm shipping complex applications with real users and real revenue. Not because I became a traditional developer, but because I learned how to work *with* AI in a way that produces reliable results.

This framework is everything I've learned, systematized. The protocols that prevent the common failures. The tools that catch mistakes before they ship. The workflow that turns "AI-assisted coding" from a gamble into a repeatable process.

**If you're technical, this will make you faster. If you're not, this is proof that you can build real things anyway.**

---

<div align="center">

**[Follow my work →](https://x.com/YachtsmanCap)**

[![Twitter Follow](https://img.shields.io/twitter/follow/YachtsmanCap?style=social)](https://x.com/YachtsmanCap)

---

MIT License

</div>
