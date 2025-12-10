# Setup Guide

## Quick Setup (30 seconds)

```bash
bd init
```

Done. You now have task tracking.

---

## Optional: Add AGENTS.md

If you want agent instructions in your project:

```bash
curl -o AGENTS.md https://raw.githubusercontent.com/Mburdo/knowledge_and_vibes/master/AGENTS_TEMPLATE.md
```

Edit the top to match your project name/language.

---

## Optional: Index Past Sessions

If you want to search your past AI coding sessions:

```bash
cass index --full
```

---

## Start Working

```bash
bd ready --json          # See available tasks
bd create "My task" -t task -p 1   # Create a task
bd update <id> --status in_progress  # Claim it
# Do work
bd close <id> --reason "Done"  # Complete it
```

---

## Need the Tools?

If `bd` isn't installed:

```bash
# Beads
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/install.sh | bash

# CASS (optional - session search)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/coding_agent_session_search/main/install.sh | bash -s -- --easy-mode

# UBS (optional - bug scanner)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ultimate_bug_scanner/master/install.sh | bash -s -- --easy-mode
```

---

## Learn More

- [PHILOSOPHY.md](./PHILOSOPHY.md) - Development framework
- [DECOMPOSITION.md](./DECOMPOSITION.md) - Breaking work into tasks
- [TUTORIAL.md](./TUTORIAL.md) - Detailed workflow
