# Knowledge & Vibes

**The premier agentic stack. December 2025.**

Eight battle-tested tools that give AI coding agents what they've been missing: persistent memory, task tracking, multi-agent coordination, quality assurance, and real-time knowledge. This is the infrastructure that turns Claude Code from a helpful assistant into an autonomous engineering force.

> **New here?** Point your agent at this repo and say: "Set up Knowledge & Vibes for my project using the SETUP_GUIDE.md"
>
> **For Agents**: You're the primary user. See [Quick Start for Agents](#quick-start-for-agents).
>
> **For Operators**: You configure and monitor. See [Operator Guide](#operator-guide).

---

## Quick Start for Agents

### The Three Commands You Need

```bash
# 1. What should I work on?
bd ready --json

# 2. What do I already know about this?
cm context "your task here" --json

# 3. Is my code safe to commit?
ubs --staged
```

That's it. Everything else is optional.

### Start Every Session

```bash
bd ready --json                              # See available tasks
bd update bd-123 --status in_progress        # Claim one
cm context "implement feature X" --json      # Get relevant context
```

### End Every Session

```bash
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

### What NOT to Do

- **Never** run `bv` or `cass` without `--robot` or `--json` flags (interactive TUIs will hang)
- **Never** delete files without explicit user approval
- **Never** run destructive git commands (`--force`, `--hard`)

---

## What's in the Toolkit

| Tool | What It Does | Interface |
|------|--------------|-----------|
| **Beads** | Track tasks across sessions | `bd` CLI |
| **Beads Viewer** | Analyze task dependencies | `bv --robot-*` CLI |
| **CASS** | Search past AI sessions | `cass` CLI |
| **cass-memory** | Learn from past sessions | `cm` CLI |
| **UBS** | Scan code for 1000+ bug patterns | `ubs` CLI |
| **Agent Mail** | Coordinate multiple agents | MCP server |
| **Warp-Grep** | 8× parallel codebase search | MCP server |
| **Exa** | Real-time web & code search | MCP server |

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

Types: `bug`, `feature`, `task`, `epic`, `chore`
Priority: `0` (critical) to `4` (backlog)
Child beads: `bd-a1b2.1`, `bd-a1b2.3.1`

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

Languages: javascript, typescript, python, c, c++, rust, go, java, ruby

Suppress false positives: `// ubs:ignore`

### Agent Mail (Multi-Agent Coordination)

When multiple agents work on the same project:

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
reply_message(project_key, message_id, sender_name, body_md="...")

# Check inbox and search
fetch_inbox(project_key, agent_name, urgent_only=True)
search_messages(project_key, query="authentication")
summarize_thread(project_key, thread_id="bd-123")

# Build coordination
acquire_build_slot(project_key, agent_name, slot="main")
release_build_slot(project_key, agent_name, slot="main")

# Quick start macro
macro_start_session(human_key="/path", program="claude-code", model="opus-4.5",
                    file_reservation_paths=["src/**"])

# Release when done
release_file_reservations(project_key, agent_name)
```

Web UI: http://127.0.0.1:8765/mail

### Warp-Grep (Parallel Code Search)

MCP tool that runs 8 parallel searches per turn. Activates automatically for natural language code questions.

**When to use**: "How does X work?", data flow analysis, cross-cutting concerns
**When NOT to use**: Known function names (use `rg`), known files (just open them)

### Exa (AI Web & Code Search)

MCP tools for real-time web search and code context:

```
web_search_exa        # Real-time web search
get_code_context_exa  # Search GitHub, docs, StackOverflow
deep_search_exa       # Deep research with query expansion
crawling              # Extract content from specific URLs
```

**When to use**: Current documentation, latest API changes, code examples
**When NOT to use**: Information in codebase (use CASS), historical context (use cm)

---

## Operator Guide

### Installation

```bash
# Option 1: Interactive installer
git clone https://github.com/Mburdo/knowledge_and_vibes.git
cd knowledge_and_vibes
./install-kv.sh
kv install

# Option 2: Direct install scripts
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/mcp_agent_mail/main/scripts/install.sh | bash -s -- --yes
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/coding_agent_session_search/main/install.sh | bash -s -- --easy-mode
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ultimate_bug_scanner/master/install.sh | bash -s -- --easy-mode
```

### Initialize a Project

```bash
cd your-project
bd init
cp /path/to/knowledge_and_vibes/AGENTS_TEMPLATE.md ./AGENTS.md
# Edit AGENTS.md for your project
```

### Health Checks

```bash
bd doctor                            # Beads health
ubs doctor                           # UBS health
cm doctor                            # cass-memory health
curl http://127.0.0.1:8765/health    # Agent Mail health
cass health                          # CASS health
```

### MCP Server Setup

**Warp-Grep** (requires Morph API key):
```bash
claude mcp add morph-fast-tools -s user \
  -e MORPH_API_KEY=your-key \
  -e ALL_TOOLS=true \
  -- npx -y @morphllm/morphmcp
```

**Exa** (requires Exa API key from dashboard.exa.ai):
```bash
claude mcp add exa -s user \
  -e EXA_API_KEY=your-key \
  -- npx -y @anthropic-labs/exa-mcp-server
```

### Multi-Agent Setup

1. Start Agent Mail: `am`
2. Each agent registers via MCP tools
3. Agents reserve files before editing
4. Agents communicate via messages/threads

### Configuration

cass-memory config: `~/.cass-memory/config.json`
```json
{
  "provider": "anthropic",
  "cassPath": "cass",
  "validationEnabled": true
}
```

Set API key: `export ANTHROPIC_API_KEY="sk-ant-..."`

---

## Repository Structure

```
knowledge_and_vibes/
├── kv                       # Interactive CLI installer
├── install-kv.sh            # Curl-able installer
├── AGENTS_TEMPLATE.md       # Template for your projects
├── TUTORIAL.md              # Detailed workflow guide
├── patches/                 # Upstream bug fixes
│   └── fix-cass-memory.sh   # Until PRs merged
├── cass_memory_system/      # Patched cass-memory (included)
│   ├── AGENTS.md            # Excellent AGENTS.md example
│   └── src/                 # Source with patches applied
└── */README.md              # Per-tool documentation
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `bd: command not found` | Add `~/.local/bin` to PATH |
| CASS finds nothing | Run `cass index --full` |
| `cm context` returns empty | Check CASS is indexed, run `cm doctor` |
| Agent Mail won't start | Check port 8765 is free |
| UBS module errors | Run `ubs doctor --fix` |
| Warp-Grep not working | Check `/mcp` shows morph-fast-tools |
| Exa not working | Check `/mcp` shows exa, verify API key |

---

## Agent Onboarding

For agents setting up this toolkit in a new project:

### Quick Setup

Tell your agent:
```
"Set up Knowledge & Vibes for my project using SETUP_GUIDE.md"
```

The agent will:
1. Install all tools
2. Configure MCP servers (asks for API keys)
3. Initialize your project with Beads
4. Create AGENTS.md
5. Index past sessions
6. Help create your initial task backlog

### Creating Implementation Plans

For large features, use the `/plan` slash command or tell your agent:
```
"Create an implementation plan for [feature] and convert it to beads"
```

The agent will:
1. Gather context from codebase and past sessions
2. Create a detailed phased plan
3. Decompose into atomic beads (~500 lines each)
4. Set up dependencies
5. Commit the backlog

See [DECOMPOSITION.md](./DECOMPOSITION.md) for guidelines on breaking work into beads.

### Slash Commands (if configured)

```
/onboard   # Full setup wizard
/plan      # Create plan and convert to beads
```

---

## Links

### Core Guides
- [PHILOSOPHY.md](./PHILOSOPHY.md) - The 4-phase framework (Requirements → Plan → Implement → Reflect)
- [DECOMPOSITION.md](./DECOMPOSITION.md) - Breaking work into beads + planning patterns
- [CODEMAPS_TEMPLATE.md](./CODEMAPS_TEMPLATE.md) - Architecture documentation for AI agents

### Setup & Reference
- [SETUP_GUIDE.md](./SETUP_GUIDE.md) - Agent-driven setup instructions
- [AGENTS_TEMPLATE.md](./AGENTS_TEMPLATE.md) - Copy to your projects
- [TUTORIAL.md](./TUTORIAL.md) - Full workflow walkthrough

### Examples
- [cass_memory_system/AGENTS.md](./cass_memory_system/AGENTS.md) - Comprehensive AGENTS.md example
- [patches/README.md](./patches/README.md) - Upstream bug tracking
