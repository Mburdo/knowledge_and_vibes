# Knowledge & Vibes

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository is my **personal source of truth** for building software with AI coding agents.  
It contains the exact tools, workflows, patterns, and rules I use across *every* project — and I keep it continuously updated as I discover better ways to work.

Think of this as the **official, up-to-date playbook** for my agentic development stack:  
how I break down requirements, how agents coordinate, how tasks persist across sessions, and how the whole system stays reliable, searchable, and fast.

If you're exploring multi-agent coding, this repo shows the stack that actually works in production:
a combination of persistent memory, structured task graphs, cross-agent learning, automated QA, and real-time knowledge retrieval.  
These workflows are not theoretical — they’re refined daily through real builds, real debugging, and real delivery.

Everything here evolves as I refine my process.  
If the file is in this repo, it’s because I actively use it.

## Install

Tell your AI agent:

```
Follow https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/SETUP_GUIDE.md to set up Knowledge & Vibes for this project.
```

Or manually: [SETUP_GUIDE.md](./SETUP_GUIDE.md)

---

## Table of Contents

- [Quick Start for Agents](#quick-start-for-agents)
- [The Toolkit](#the-toolkit)
- [Tool Reference](#tool-reference)
  - [Beads (Task Tracking)](#beads-task-tracking)
  - [Beads Viewer (Graph Analysis)](#beads-viewer-graph-analysis)
  - [CASS (Session Search)](#cass-session-search)
  - [cass-memory (Cross-Agent Learning)](#cass-memory-cross-agent-learning)
  - [UBS (Bug Scanner)](#ubs-bug-scanner)
  - [NTM (Multi-Agent Tmux Manager)](#ntm-multi-agent-tmux-manager)
  - [Agent Mail (Multi-Agent Coordination)](#agent-mail-multi-agent-coordination)
  - [Warp-Grep (Parallel Code Search)](#warp-grep-parallel-code-search)
  - [Exa (AI Web & Code Search)](#exa-ai-web--code-search)
- [Operator Guide](#operator-guide)
  - [Installation](#installation)
  - [Project Setup](#project-setup)
  - [MCP Servers](#mcp-servers)
  - [Health Checks](#health-checks)
- [Documentation](#documentation)
- [Troubleshooting](#troubleshooting)

---

## Quick Start for Agents

### The Three Commands You Need

```bash
bd ready --json              # 1. What should I work on?
cm context "task" --json     # 2. What do I already know?
ubs --staged                 # 3. Is my code safe to commit?
```

### Session Workflow

```bash
# Start
bd ready --json                              # See available tasks
bd update bd-123 --status in_progress --assignee AgentName  # Claim one
cm context "implement feature X" --json      # Get relevant context

# Name your session (Claude Code 2.0.64+)
# Format: {project}-{AgentMailName}-{YYYYMMDD-HHMMSS}
/rename myapp-GreenCastle-20251210-143022

# End
ubs --staged                                 # Scan for bugs
bd close bd-123 --reason "Implemented X"     # Complete task
git add -A && git commit && git push         # Save everything
```

Resume sessions later with `claude --resume myapp-GreenCastle-20251210-143022`

### When Stuck

```bash
cass search "similar problem" --robot        # Search past sessions
bv --robot-triage                            # Full recommendations bundle
# or: bv --robot-next                        # Minimal “what next?”
bv --robot-alerts                            # What looks risky/stale right now?
bv --search "your query" --robot-search      # Semantic search over beads
# Use Exa MCP for current docs/APIs          # Real-time web search
```

### Rules

- **Always** use `--robot-*` flags with `bv` (bare `bv` launches a TUI and will hang)
- **Always** use `--robot` or `--json` flags with `cass`
- **Never** delete files without explicit user approval
- **Never** run destructive git commands (`--force`, `--hard`)

---

## The Toolkit

| Tool | Purpose | Interface |
|------|---------|-----------|
| [Beads](#beads-task-tracking) | Task tracking across sessions | `bd` CLI |
| [Beads Viewer](#beads-viewer-graph-analysis) | Dependency graph analysis | `bv --robot-*` CLI |
| [CASS](#cass-session-search) | Search past AI sessions | `cass` CLI |
| [cass-memory](#cass-memory-cross-agent-learning) | Learn from past sessions | `cm` CLI |
| [UBS](#ubs-bug-scanner) | Scan for 1000+ bug patterns | `ubs` CLI |
| [NTM](#ntm-multi-agent-tmux-manager) | Multi-agent tmux orchestration | `ntm` CLI |
| [Agent Mail](#agent-mail-multi-agent-coordination) | Multi-agent coordination | MCP server |
| [Warp-Grep](#warp-grep-parallel-code-search) | Parallel codebase search | MCP server |
| [Exa](#exa-ai-web--code-search) | Real-time web & code search | MCP server |

---

## Tool Reference

### Beads (Task Tracking)

```bash
# Core workflow
bd ready --json                    # What's available?
bd update ID --status in_progress --assignee YOUR_AGENT_NAME  # Claim task
bd close ID --reason "Done"        # Complete it
bd create "Title" -t bug -p 1      # Create new task

# Dependencies
bd dep add bd-child bd-blocker --type blocks
bd dep tree bd-42                  # Visualize deps
bd blocked                         # What's waiting?

# Maintenance
bd doctor --fix                    # Health check
```

| Concept | Values |
|---------|--------|
| Types | `bug`, `feature`, `task`, `epic`, `chore` |
| Priority | `0` (critical) → `4` (backlog) |
| Child beads | `bd-a1b2.1`, `bd-a1b2.3.1` |

**Rule**: Always commit `.beads/` with your code changes.

**Never:**
- Use markdown TODO lists (use beads)
- Use other trackers (beads is authoritative)
- Claim only the parent bead (always claim ALL sub-beads)
- Skip `[CLAIMED]`/`[CLOSED]` announcements
- Edit `.beads/*.jsonl` directly (only via `bd`)

### Beads Viewer (Graph Analysis)

```bash
# Default session kickoff
bv --robot-next                    # Minimal “what should I do next?”
bv --robot-triage                  # Full bundle (blockers, quick wins, commands)

# Multi-agent partitioning
bv --robot-triage --robot-triage-by-track
bv --robot-plan

# Risk & hygiene (use before big work)
bv --robot-alerts
bv --robot-suggest

# Debugging / regression analysis
bv --robot-diff --diff-since HEAD~5
bv --as-of HEAD~10 --robot-triage

# Handoff (when needed)
bv --robot-history --bead-history bd-123
```

**Graph metrics explained:**
- **PageRank**: Foundational blockers (tasks that enable many others)
- **Betweenness**: Bottlenecks (must pass through these)
- **Critical path**: Longest dependency chain

**Rule**: Always use `--robot-*` flags. Never run bare `bv`.

### CASS (Session Search)

```bash
cass search "query" --robot --limit 5        # Find past solutions
cass search "query" --robot --fields minimal # Lean output
cass search "query" --robot --max-tokens 2000  # Token budget
cass view /path/to/session.jsonl --json      # View a session
cass expand /path -n 42 -C 3 --json          # Expand context
cass timeline --today --json                 # Today's activity
cass export /path/session.jsonl --format markdown
```

**Rule**: Always use `--robot` or `--json`. Never run bare `cass`.

### cass-memory (Cross-Agent Learning)

```bash
cm context "task description" --json    # Get playbook + history + anti-patterns
cm doctor                               # Health check
```

The system learns automatically. You just query it.

### UBS (Bug Scanner)

```bash
ubs --staged                       # Scan staged changes (pre-commit)
ubs --staged --fail-on-warning     # Strict mode
ubs --diff                         # Scan working tree changes
ubs path/to/file.ts                # Scan specific file
ubs --profile=strict .             # Fail on warnings
ubs --only=typescript .            # Language filter
ubs . --format=sarif               # GitHub Code Scanning format
```

| Feature | Details |
|---------|---------|
| Languages | javascript, typescript, python, c, c++, rust, go, java, ruby |
| Suppress | `// ubs:ignore` |

### NTM (Multi-Agent Tmux Manager)

Orchestrates multiple AI coding agents (Claude, Codex, Gemini) in tiled tmux panes.

```bash
# Spawn a multi-agent session
ntm spawn myproject --cc=2 --cod=1    # 2 Claude + 1 Codex agents

# Send prompts to agents
ntm send myproject --cc "fix TypeScript errors"   # Send to all Claude agents
ntm send myproject --all "analyze this code"      # Broadcast to all agents

# Manage sessions
ntm list                               # Show active sessions
ntm attach myproject                   # Attach to session
ntm kill myproject                     # End session

# Interactive command palette
ntm palette myproject                  # Fuzzy-search commands with TUI
```

| Feature | Details |
|---------|---------|
| Agents | Claude Code (`--cc`), Codex (`--cod`), Gemini (`--gem`) |
| TUI | Command palette with fuzzy search, Catppuccin themes |
| Hooks | Pre/post-command automation for logging/notifications |
| Output | Capture and filter agent outputs |

**Tip**: Use NTM to run multiple agents in parallel, then coordinate via Agent Mail.

### Agent Mail (Multi-Agent Coordination)

```python
# Register yourself
ensure_project(human_key="/path/to/project")
register_agent(project_key="/path/to/project", program="claude-code", model="opus-4.5")

# Reserve files before editing
file_reservation_paths(project_key, agent_name, ["src/**"], exclusive=True)
renew_file_reservations(project_key, agent_name, extend_seconds=1800)

# Communicate with other agents
send_message(project_key, sender_name, to=["OtherAgent"],
             subject="...", body_md="...", importance="high")
fetch_inbox(project_key, agent_name, urgent_only=True)
search_messages(project_key, query="authentication")

# Build coordination
acquire_build_slot(project_key, agent_name, slot="main")
release_build_slot(project_key, agent_name, slot="main")

# Quick start macro
macro_start_session(human_key="/path", program="claude-code", model="opus-4.5",
                    file_reservation_paths=["src/**"])
```

Web UI: http://127.0.0.1:8765/mail

### Multi-Agent Coordination Rules

**These rules are MANDATORY to prevent conflicts when running multiple agents.**

#### Rule 1: Claim ALL Sub-Beads Together

When you claim a parent bead, you MUST claim ALL its sub-beads immediately:

```bash
bd update bd-123 --status in_progress --assignee YOUR_AGENT_NAME
bd update bd-123.1 --status in_progress --assignee YOUR_AGENT_NAME
bd update bd-123.2 --status in_progress --assignee YOUR_AGENT_NAME
```

**Why:** If you only claim the parent, another agent sees sub-beads as "ready" → CONFLICT.

#### Rule 2: Reserve Files Before Editing

```python
file_reservation_paths(
    project_key, agent_name,
    paths=["src/module/**", "tests/test_module.py"],
    ttl_seconds=3600, exclusive=True, reason="bd-123"
)
```

#### Rule 3: Announce `[CLAIMED]` When Starting

```python
send_message(project_key, sender_name, to=["all"],
    subject="[CLAIMED] bd-123 - Feature Title",
    body_md="Starting work on **bd-123** (plus sub-beads .1, .2).\n\nFile reservations: `src/module/**`",
    thread_id="bd-123")
```

#### Rule 4: Announce `[CLOSED]` When Finishing

```python
send_message(project_key, sender_name, to=["all"],
    subject="[CLOSED] bd-123 - Feature Title",
    body_md="Completed **bd-123**.\n\nFiles created: ...\nReleasing reservations.",
    thread_id="bd-123")
```

#### Rule 5: Check Inbox BEFORE Claiming

Before running `bd ready`, check your inbox for recent `[CLAIMED]` messages.

### Bead Claiming Checklist

```
□ 1. Check inbox for recent [CLAIMED] messages
□ 2. Run `bd ready --json` to find unblocked work
□ 3. Run `bv --robot-next` (or `bv --robot-triage`) to pick the best next task
□ 4. Check current file reservations (avoid conflicts)
□ 5. Claim PARENT bead: `bd update <id> --status in_progress --assignee YOUR_NAME`
□ 6. Claim ALL SUB-BEADS: `bd update <id.1> --status in_progress --assignee YOUR_NAME` (repeat for all)
□ 7. Reserve ALL file paths you will touch (exclusive when appropriate)
□ 8. Send `[CLAIMED]` message (use `thread_id="<id>"`, list reserved paths)
□ 9. Work on the bead (keep updates in-thread)
```

### Bead Finish Checklist

```
□ 1. Run tests / builds relevant to your change
□ 2. Run `ubs --staged` (fix issues in files you touched; rerun until clean)
□ 3. Commit your work (include `.beads/issues.jsonl`): `git add -A && git commit`
□ 4. Close ALL sub-beads first: `bd close <id>.1 ...` (repeat for all)
□ 5. Close the parent bead: `bd close <id> --reason "Completed: ..."`
□ 6. Release file reservations
□ 7. Send `[CLOSED]` message in the same thread (what changed, tests run, reservations released)
□ 8. `git push`
```

**Note:** Beads auto-exports to JSONL. Share via normal git commits (include `.beads/issues.jsonl` with code changes).

### Warp-Grep (Parallel Code Search)

MCP tool that runs 8 parallel searches per turn. Activates automatically for natural language code questions.

| Use Case | Tool |
|----------|------|
| "How does X work?" | Warp-Grep |
| Data flow analysis | Warp-Grep |
| Known function name | `rg` (ripgrep) |
| Known file path | Just open it |

### Exa (AI Web & Code Search)

```
web_search_exa        # Real-time web search
get_code_context_exa  # Search GitHub, docs, StackOverflow
deep_search_exa       # Deep research with query expansion
crawling              # Extract content from specific URLs
```

| Use Case | Tool |
|----------|------|
| Current documentation | Exa |
| Latest API changes | Exa |
| Info in codebase | CASS |
| Historical context | cass-memory |

---

## Operator Guide

### Installation

```bash
# Agent Mail (installs bd, bv, am)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/mcp_agent_mail/main/scripts/install.sh | bash -s -- --yes

# CASS (session search)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/coding_agent_session_search/main/install.sh | bash -s -- --easy-mode

# cass-memory (cross-agent learning)
curl -L https://github.com/Dicklesworthstone/cass_memory_system/releases/latest/download/cass-memory-macos-arm64 -o ~/.local/bin/cm && chmod +x ~/.local/bin/cm

# UBS (bug scanner)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ultimate_bug_scanner/master/install.sh | bash -s -- --easy-mode

# NTM (multi-agent tmux manager)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ntm/main/install.sh | bash
```

See [SETUP_GUIDE.md](./SETUP_GUIDE.md) for full setup including MCP servers.

### Project Setup

```bash
cd your-project
bd init
cp /path/to/knowledge_and_vibes/AGENTS_TEMPLATE.md ./AGENTS.md
# Edit AGENTS.md for your project
```

Or tell your agent: `"Set up Knowledge & Vibes for my project using SETUP_GUIDE.md"`

### MCP Servers

**Warp-Grep** (requires [Morph API key](https://morphllm.com)):
```bash
claude mcp add morph-fast-tools -s user \
  -e MORPH_API_KEY=your-key \
  -e ALL_TOOLS=true \
  -- npx -y @morphllm/morphmcp
```

**Exa** (requires [Exa API key](https://dashboard.exa.ai)):
```bash
claude mcp add exa -s user \
  -e EXA_API_KEY=your-key \
  -- npx -y @anthropic-labs/exa-mcp-server
```

**Agent Mail**:
```bash
am  # Start the server
# Web UI at http://127.0.0.1:8765/mail
```

### Health Checks

```bash
bd doctor              # Beads
ubs doctor             # UBS
cm doctor              # cass-memory
cass health            # CASS
curl localhost:8765/health  # Agent Mail
ntm doctor             # NTM
```

---

## Documentation

### Start Here

| If you want to... | Read |
|-------------------|------|
| Understand the philosophy | [PHILOSOPHY.md](./PHILOSOPHY.md) → [TUTORIAL.md](./TUTORIAL.md) |
| Set up a new project | [SETUP_GUIDE.md](./SETUP_GUIDE.md) |
| Onboard an existing project | [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) |
| Deep dive on planning & beads | [PLANNING_AND_DECOMPOSITION.md](./PLANNING_AND_DECOMPOSITION.md) |
| Configure Claude Code (.claude/) | [CLAUDE_CONFIG_GUIDE.md](./CLAUDE_CONFIG_GUIDE.md) |

### Configuration Locations

| Location | Purpose | When to Use |
|----------|---------|-------------|
| `CLAUDE.md` | Project context, architecture | Understanding the codebase |
| `AGENTS.md` | Workflow instructions | Session startup, tool usage |
| `.claude/rules/` | Constraints and conventions | Auto-loaded, always follow |
| `.claude/skills/` | Detailed guides and capabilities | Reference when relevant |
| `.claude/commands/` | Slash commands | Invoke with `/command-name` |

Included commands in this repo (copy into your project):
- `/prime` → `.claude/commands/prime.md`
- `/next-bead` → `.claude/commands/next-bead.md`
- `/ground` → `.claude/commands/ground.md`
- `/decompose-task` → `.claude/commands/decompose-task.md`

### Philosophy & Approach
| Document | Description |
|----------|-------------|
| [PHILOSOPHY.md](./PHILOSOPHY.md) | The 4-phase framework: Requirements → Plan → Implement → Reflect |
| [CODEMAPS_TEMPLATE.md](./CODEMAPS_TEMPLATE.md) | Architecture documentation templates for AI agents |

### Planning & Decomposition
| Document | Description |
|----------|-------------|
| [PLANNING_AND_DECOMPOSITION.md](./PLANNING_AND_DECOMPOSITION.md) | Planning workflow + breaking work into beads |
| [TUTORIAL.md](./TUTORIAL.md) | Complete workflow walkthrough |

### Setup & Templates
| Document | Description |
|----------|-------------|
| [SETUP_GUIDE.md](./SETUP_GUIDE.md) | Fresh project setup (new projects) |
| [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) | Onboarding existing projects to K&V stack |
| [AGENTS_TEMPLATE.md](./AGENTS_TEMPLATE.md) | Template for AGENTS.md workflow |
| [CLAUDE_CONFIG_GUIDE.md](./CLAUDE_CONFIG_GUIDE.md) | Organizing rules, skills, and commands |


---

## Troubleshooting

### Common Issues

| Issue | Fix |
|-------|-----|
| `bd: command not found` | Add `~/.local/bin` to PATH: `export PATH="$HOME/.local/bin:$PATH"` |
| `bv: command not found` | Same PATH fix, or reinstall via agent-mail installer |
| `bv` or `cass` hangs | For `bv`: always use `bv --robot-*` flags. For `cass`: always use `--robot` or `--json`. |
| CASS finds nothing | Run `cass index --full` to index sessions |
| `cm context` returns empty | Check CASS indexed, run `cm doctor`. Update to cm v0.2.0+ |
| `cm` errors | Upgrade `cm`/`cass` and retry; `cm context` is the primary entrypoint |
| Agent Mail won't start | Check port 8765 is free: `lsof -i :8765` |
| Agent Mail MCP not connecting | Run `am` to start server, check `curl localhost:8765/health` |
| UBS module errors | Run `ubs doctor --fix` |
| Warp-Grep not working | Check `/mcp` shows morph-fast-tools, verify MORPH_API_KEY |
| Exa not working | Check `/mcp` shows exa, verify EXA_API_KEY |
| `git push` conflicts on `.beads/issues.jsonl` | Resolve like any text merge conflict, then `bd sync --import-only` to reload |
| Permission denied on install | Use `~/.local/bin` instead of `/usr/local/bin`, or use `sudo` |
| `ntm: command not found` | Run installer or add to PATH: `export PATH="$HOME/.local/bin:$PATH"` |
| NTM can't find tmux | Install tmux: `brew install tmux` (macOS) or `apt install tmux` (Linux) |
| NTM agents not spawning | Check agent CLIs are installed (`claude --version`, etc.) |

### Agent-Specific Rules

**Critical**: These tools have TUI modes that will hang AI agents:
- `bv` → Always use `bv --robot-*` flags
- `cass` → Always use `cass --robot` or `--json` flags

### Health Checks

```bash
# Quick health check for all tools
bd doctor && cm doctor && ubs doctor && cass health && curl -s localhost:8765/health && ntm doctor
```

---

## Repository Structure

```
knowledge_and_vibes/
├── README.md                # This file
├── SETUP_GUIDE.md           # Fresh project setup
├── MIGRATION_GUIDE.md       # Onboarding existing projects
├── AGENTS_TEMPLATE.md       # Template for your projects
├── CLAUDE_CONFIG_GUIDE.md   # Organizing .claude/ directory
├── PHILOSOPHY.md            # 4-phase development framework
├── PLANNING_AND_DECOMPOSITION.md  # Planning workflow + task breakdown
├── CODEMAPS_TEMPLATE.md     # Architecture documentation
├── TUTORIAL.md              # Detailed workflow guide
└── LICENSE                  # MIT
```
