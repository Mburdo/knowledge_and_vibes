# Knowledge & Vibes

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**The premier agentic stack. December 2025.**

Eight battle-tested tools that give AI coding agents what they've been missing: persistent memory, task tracking, multi-agent coordination, quality assurance, and real-time knowledge.

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
bd update bd-123 --status in_progress        # Claim one
cm context "implement feature X" --json      # Get relevant context

# End
ubs --staged                                 # Scan for bugs
bd close bd-123 --reason "Implemented X"     # Complete task
git add -A && git commit && git push         # Save everything
```

### When Stuck

```bash
cass search "similar problem" --robot        # Search past sessions
bv --robot-priority                          # Get recommendations
# Use Exa MCP for current docs/APIs          # Real-time web search
```

### Rules

- **Always** use `--robot` or `--json` flags with `bv` and `cass` (TUIs will hang)
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
| [Agent Mail](#agent-mail-multi-agent-coordination) | Multi-agent coordination | MCP server |
| [Warp-Grep](#warp-grep-parallel-code-search) | Parallel codebase search | MCP server |
| [Exa](#exa-ai-web--code-search) | Real-time web & code search | MCP server |

---

## Tool Reference

### Beads (Task Tracking)

```bash
# Core workflow
bd ready --json                    # What's available?
bd update ID --status in_progress  # Claim a task
bd close ID --reason "Done"        # Complete it
bd create "Title" -t bug -p 1      # Create new task

# Dependencies
bd dep add bd-child bd-blocker --type blocks
bd dep tree bd-42                  # Visualize deps
bd blocked                         # What's waiting?

# Maintenance
bd doctor --fix                    # Health check
bd sync                            # Force sync
```

| Concept | Values |
|---------|--------|
| Types | `bug`, `feature`, `task`, `epic`, `chore` |
| Priority | `0` (critical) → `4` (backlog) |
| Child beads | `bd-a1b2.1`, `bd-a1b2.3.1` |

**Rule**: Always commit `.beads/` with your code changes.

### Beads Viewer (Graph Analysis)

```bash
bv --robot-priority                # What should I work on?
bv --robot-plan                    # Parallel execution tracks
bv --robot-insights                # Graph metrics (PageRank, betweenness, HITS)
bv --robot-diff --diff-since "1 hour ago"  # Recent changes
bv --robot-recipes                 # Available filter presets
```

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

### Agent Mail (Multi-Agent Coordination)

```python
# Register yourself
ensure_project(project_key="/path/to/project")
register_agent(project_key, program="claude-code", model="opus-4.5")

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

# cass-memory (see Troubleshooting if issues)
curl -L https://github.com/Dicklesworthstone/cass_memory_system/releases/latest/download/cass-memory-darwin-arm64 -o ~/.local/bin/cm && chmod +x ~/.local/bin/cm

# UBS (bug scanner)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ultimate_bug_scanner/master/install.sh | bash -s -- --easy-mode
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
```

---

## Documentation

### Philosophy & Approach
| Document | Description |
|----------|-------------|
| [PHILOSOPHY.md](./PHILOSOPHY.md) | The 4-phase framework: Requirements → Plan → Implement → Reflect |
| [CODEMAPS_TEMPLATE.md](./CODEMAPS_TEMPLATE.md) | Architecture documentation templates for AI agents |

### Planning & Decomposition
| Document | Description |
|----------|-------------|
| [DECOMPOSITION.md](./DECOMPOSITION.md) | Breaking work into beads + 5 planning patterns |
| [TUTORIAL.md](./TUTORIAL.md) | Complete workflow walkthrough |

### Setup & Templates
| Document | Description |
|----------|-------------|
| [SETUP_GUIDE.md](./SETUP_GUIDE.md) | Agent-driven setup instructions |
| [AGENTS_TEMPLATE.md](./AGENTS_TEMPLATE.md) | Template for your projects |

### Examples
| Document | Description |
|----------|-------------|
| [cass_memory_system/AGENTS.md](./cass_memory_system/AGENTS.md) | Comprehensive AGENTS.md example |

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `bd: command not found` | Add `~/.local/bin` to PATH |
| CASS finds nothing | Run `cass index --full` |
| `cm context` returns empty | Check CASS is indexed, run `cm doctor`. If still empty, apply patches (see below) |
| `cm reflect` broken | Known CASS bug, use `cm context` instead |
| Agent Mail won't start | Check port 8765 is free |
| UBS module errors | Run `ubs doctor --fix` |
| Warp-Grep not working | Check `/mcp` shows morph-fast-tools |
| Exa not working | Check `/mcp` shows exa, verify API key |

### CASS + cass-memory Compatibility

If `cm context` returns empty results even with indexed sessions, you may need to apply patches for two upstream bugs:

1. **Search parsing bug**: CASS returns `{hits:[...]}` but cass-memory expects `[...]`
2. **Nullable field bug**: Some CASS hits have `created_at: null` which fails validation

**To fix**, use our patched cass-memory:
```bash
cd /path/to/knowledge_and_vibes/cass_memory_system
bun install && bun run build
sudo mv ./dist/cass-memory /usr/local/bin/cm
```

Or apply patches to a fresh clone:
```bash
git clone https://github.com/Dicklesworthstone/cass_memory_system.git
cd cass_memory_system
/path/to/knowledge_and_vibes/patches/fix-cass-memory.sh .
bun install && bun run build
sudo mv ./dist/cass-memory /usr/local/bin/cm
```

**Tracking issues**:
- [cass_memory_system#2](https://github.com/Dicklesworthstone/cass_memory_system/issues/2)
- [coding_agent_session_search#7](https://github.com/Dicklesworthstone/coding_agent_session_search/issues/7)

---

## Repository Structure

```
knowledge_and_vibes/
├── README.md                # This file
├── SETUP_GUIDE.md           # Agent-driven setup
├── AGENTS_TEMPLATE.md       # Template for your projects
├── PHILOSOPHY.md            # 4-phase development framework
├── DECOMPOSITION.md         # Task breakdown + planning patterns
├── CODEMAPS_TEMPLATE.md     # Architecture documentation
├── TUTORIAL.md              # Detailed workflow guide
├── LICENSE                  # MIT
├── patches/                 # Upstream bug fixes
└── cass_memory_system/      # Patched cass-memory
```
