# Setup Guide

**For AI agents setting up Knowledge & Vibes.**

## Step 1: Install Tools

Run all of these (they skip if already installed):

```bash
# Agent Mail (installs bd, bv, and am) - installs globally to ~/mcp_agent_mail
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/mcp_agent_mail/main/scripts/install.sh | bash -s -- --dir "$HOME/mcp_agent_mail" --yes

# CASS (session search)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/coding_agent_session_search/main/install.sh | bash -s -- --easy-mode

# cass-memory (cross-agent learning) - CHOOSE ONE OPTION:

# Option A: Build from patched source (RECOMMENDED - fixes upstream bugs)
# Requires: bun (install via: curl -fsSL https://bun.sh/install | bash)
git clone --depth 1 https://github.com/Mburdo/knowledge_and_vibes.git /tmp/kv-install && \
  cd /tmp/kv-install/cass_memory_system && bun install && bun run build && \
  cp dist/cass-memory ~/.local/bin/cm && chmod +x ~/.local/bin/cm && \
  cd ~ && rm -rf /tmp/kv-install

# Option B: Use upstream binary (has known bugs - see patches/README.md)
# curl -L https://github.com/Dicklesworthstone/cass_memory_system/releases/latest/download/cass-memory-darwin-arm64 -o ~/.local/bin/cm && chmod +x ~/.local/bin/cm

# UBS (bug scanner)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ultimate_bug_scanner/master/install.sh | bash -s -- --easy-mode

# NTM (multi-agent tmux manager)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ntm/main/install.sh | bash
```

Verify:
```bash
bd --version && bv --version && cass --version && cm --version && ubs --version && ntm --version && echo "✓ All installed"
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
  bv --robot-priority       # Recommended next task
  cm context "task" --json  # Get relevant context
  ubs --staged              # Scan for bugs
  cass search "..." --robot # Search past sessions

Next: Create a plan for what you want to build.
Read PHILOSOPHY.md and DECOMPOSITION.md for guidance.
```
