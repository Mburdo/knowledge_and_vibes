# Setup Guide

**For AI agents setting up Knowledge & Vibes.**

## Step 1: Install Tools

Run all of these (they skip if already installed):

```bash
# Beads (task tracking)
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/install.sh | bash

# CASS (session search) + cass-memory
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/coding_agent_session_search/main/install.sh | bash -s -- --easy-mode

# UBS (bug scanner)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ultimate_bug_scanner/master/install.sh | bash -s -- --easy-mode

# Agent Mail (multi-agent coordination)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/mcp_agent_mail/main/scripts/install.sh | bash -s -- --yes
```

Verify:
```bash
bd --version && cass --version && ubs --version && echo "✓ All installed"
```

## Step 2: API Keys (Optional)

**Ask the user:**
- "Do you have a Morph API key for Warp-Grep (faster code search)?"
- "Do you have an Exa API key for web search?"

If yes to Morph:
```bash
claude mcp add morph-fast-tools -s user -e MORPH_API_KEY=<key> -e ALL_TOOLS=true -- npx -y @morphllm/morphmcp
```

If yes to Exa:
```bash
claude mcp add exa -s user -e EXA_API_KEY=<key> -- npx -y @anthropic-labs/exa-mcp-server
```

## Step 3: Initialize Project

```bash
bd init
curl -o AGENTS.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/AGENTS_TEMPLATE.md
cass index --full
git add .beads/ AGENTS.md && git commit -m "Initialize Knowledge & Vibes"
```

## Step 4: Done

Tell the user:

```
Setup complete!

Installed: bd, cass, cm, ubs, agent-mail
Project: .beads/ and AGENTS.md added

Commands:
  bd ready --json     # See tasks
  bd create "..." -t task -p 1  # Create task
  ubs --staged        # Scan for bugs
  cass search "..."   # Search past sessions

Next: Create a plan for what you want to build.
Read PHILOSOPHY.md and DECOMPOSITION.md for guidance.
```
