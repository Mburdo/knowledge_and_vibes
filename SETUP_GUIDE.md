# Setup Guide

**For AI agents setting up Knowledge & Vibes.**

## Step 1: Install Tools

Run all of these (they skip if already installed):

```bash
# Agent Mail (installs bd, bv, and am) - installs globally to ~/mcp_agent_mail
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/mcp_agent_mail/main/scripts/install.sh | bash -s -- --dir "$HOME/mcp_agent_mail" --yes

# CASS (session search)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/coding_agent_session_search/main/install.sh | bash -s -- --easy-mode

# cass-memory (cross-agent learning)
curl -L https://github.com/Dicklesworthstone/cass_memory_system/releases/latest/download/cass-memory-macos-arm64 -o ~/.local/bin/cm && chmod +x ~/.local/bin/cm

# UBS (bug scanner)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ultimate_bug_scanner/master/install.sh | bash -s -- --easy-mode

# NTM (multi-agent tmux manager)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ntm/main/install.sh | bash
```

Verify:
```bash
bd --version && bv --version && cass --version && cm --version && ubs --version && ntm --version && echo "✓ All installed"
```

### Update Beads Viewer (Recommended)

`bv` has a built-in updater (v0.10.6+):

```bash
bv --check-update
bv --update --yes
```

If your `bv` is older and lacks `--update`, run the installer:

```bash
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/beads_viewer/main/install.sh?$(date +%s)" | bash
```

## Step 2: MCP Servers

**Get API keys:**
- Morph (Warp-Grep): https://morphllm.com
- Exa (web/code search): https://dashboard.exa.ai

**Install:**
```bash
# Warp-Grep - parallel codebase search
claude mcp add morph-fast-tools -s user -e MORPH_API_KEY=<key> -e ALL_TOOLS=true -- npx -y @morphllm/morphmcp

# Exa - real-time web and code search
claude mcp add exa -s user -e EXA_API_KEY=<key> -- npx -y @anthropic-labs/exa-mcp-server
```

## Step 3: Initialize Project

```bash
bd init
curl -o AGENTS.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/AGENTS_TEMPLATE.md
cass index --full
git add .beads/ AGENTS.md && git commit -m "Initialize Knowledge & Vibes"
```

## Step 4: Configure Agent Instructions (Optional)

Set up rules, skills, and commands for better agent behavior:

```bash
# Create directory structure
mkdir -p .claude/rules .claude/skills .claude/commands

# Add safety rules (recommended)
cat > .claude/rules/safety.md << 'EOF'
# Safety Rules

Never delete files without explicit user permission.
Never run destructive git commands without approval.
EOF

git add .claude/ && git commit -m "Add Claude Code configuration"
```

### Add Recommended Safety Rule (Optional)

This repo also ships a more explicit safety rule you can copy into your project:
- `safety.md` → `.claude/rules/safety.md`

Copy from a local clone:
```bash
cp /path/to/knowledge_and_vibes/.claude/rules/safety.md .claude/rules/safety.md
git add .claude/rules/safety.md && git commit -m "Add safety rule"
```

Or fetch from GitHub:
```bash
curl -fsSL -o .claude/rules/safety.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/.claude/rules/safety.md
git add .claude/rules/safety.md && git commit -m "Add safety rule"
```

### Add Recommended Slash Commands (Optional)

This repo includes two starter slash commands you can copy into your project’s `.claude/commands/`:
- `/prime` → `.claude/commands/prime.md`
- `/next-bead` → `.claude/commands/next-bead.md`
- `/ground` → `.claude/commands/ground.md`
- `/decompose-task` → `.claude/commands/decompose-task.md`

Copy them from a local clone of `knowledge_and_vibes`:
```bash
cp /path/to/knowledge_and_vibes/.claude/commands/prime.md .claude/commands/prime.md
cp /path/to/knowledge_and_vibes/.claude/commands/next-bead.md .claude/commands/next-bead.md
cp /path/to/knowledge_and_vibes/.claude/commands/ground.md .claude/commands/ground.md
cp /path/to/knowledge_and_vibes/.claude/commands/decompose-task.md .claude/commands/decompose-task.md
```

Or fetch from GitHub:
```bash
curl -fsSL -o .claude/commands/prime.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/.claude/commands/prime.md
curl -fsSL -o .claude/commands/next-bead.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/.claude/commands/next-bead.md
curl -fsSL -o .claude/commands/ground.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/.claude/commands/ground.md
curl -fsSL -o .claude/commands/decompose-task.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/.claude/commands/decompose-task.md
```

Then commit:
```bash
git add .claude/commands && git commit -m "Add Claude slash commands"
```

For detailed guidance on rules, skills, and commands, see:
https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/CLAUDE_CONFIG_GUIDE.md

## Step 5: Done

Tell the user:

```
Setup complete!

Installed: bd, bv, cass, cm, ubs, ntm, am
Project: .beads/ and AGENTS.md added

Commands:
  bd ready --json           # See tasks
  bv --robot-next           # Recommended next task (minimal)
  bv --robot-triage         # Full triage bundle (recommended)
  cm context "task" --json  # Get relevant context
  ubs --staged              # Scan for bugs
  cass search "..." --robot # Search past sessions

Next: Create a plan for what you want to build.
Read PHILOSOPHY.md and PLANNING_AND_DECOMPOSITION.md for guidance.
```
