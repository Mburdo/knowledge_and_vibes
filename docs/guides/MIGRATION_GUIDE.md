---
title: Migration Guide
description: Bring an existing project into the Knowledge & Vibes stack - restructuring Claude Code configuration, installing tools, and setting up documentation.
category: guides
---

# Migration Guide

Bring an existing project into the Knowledge & Vibes stack. This guide covers restructuring Claude Code configuration, installing tools, and setting up proper documentation.

**New project instead?** Use [SETUP_GUIDE.md](./SETUP_GUIDE.md).

---

## Table of Contents

1. [Pre-Migration Audit](#1-pre-migration-audit)
2. [Tool Installation](#2-tool-installation)
3. [Directory Structure Setup](#3-directory-structure-setup)
4. [Rules Migration](#4-rules-migration)
5. [Skills Migration](#5-skills-migration)
6. [CLAUDE.md Cleanup](#6-claudemd-cleanup)
7. [AGENTS.md Restructuring](#7-agentsmd-restructuring)
8. [Settings Configuration](#8-settings-configuration)
9. [Beads Initialization](#9-beads-initialization)
10. [Verification](#10-verification)

---

## 1. Pre-Migration Audit

Before making changes, audit your current state:

```bash
cd /path/to/your/project

# Check existing structure
ls -la AGENTS.md CLAUDE.md 2>/dev/null
ls -la .claude/ 2>/dev/null
ls -la skills/ 2>/dev/null
ls -la .beads/ 2>/dev/null

# Count lines in existing files
wc -l AGENTS.md CLAUDE.md 2>/dev/null
```

### Audit Checklist

| Item | Check | Notes |
|------|-------|-------|
| `AGENTS.md` exists? | ☐ | Will need restructuring |
| `CLAUDE.md` exists? | ☐ | Will need cleanup |
| `.claude/` directory? | ☐ | May need creation |
| `.claude/rules/` exists? | ☐ | Usually missing - needs creation |
| `.claude/skills/` exists? | ☐ | Often in wrong location |
| `skills/` in root? | ☐ | Wrong location - needs moving |
| `.claude/commands/` exists? | ☐ | Usually correct if exists |
| `.beads/` exists? | ☐ | Init if missing |
| `.claude/settings.local.json`? | ☐ | Needs tool permissions |

### Common Issues Found

1. **Skills in wrong location**: `skills/` instead of `.claude/skills/`
2. **Wrong skill filename**: `skill.md` instead of `SKILL.md`
3. **Missing frontmatter**: Skills need `name:` and `description:` in YAML frontmatter
4. **No rules directory**: Rules not extracted from AGENTS.md/CLAUDE.md
5. **Duplicated content**: Same rules in both AGENTS.md and CLAUDE.md
6. **Missing tool permissions**: settings.local.json doesn't allow K&V tools
7. **Missing tool documentation**: AGENTS.md doesn't cover the full stack

---

## 2. Tool Installation

Ensure all K&V tools are installed and up to date:

```bash
# Check current versions
bd --version && bv --version && cass --version && cm --version && ubs --version

# Install/update Agent Mail (includes bd, bv)
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/mcp_agent_mail/main/scripts/install.sh | bash -s -- --dir "$HOME/mcp_agent_mail" --yes

# Update bv to latest (recommended: unlocks --robot-triage, --robot-next, label analytics, history, etc.)
# If your installed bv supports it:
bv --check-update
bv --update --yes
#
# If `bv --update` is not available (older bv), use the official installer:
# curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/beads_viewer/main/install.sh?$(date +%s)" | bash

# Install/update CASS
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/coding_agent_session_search/main/install.sh | bash -s -- --easy-mode

# Install/update cass-memory
curl -L https://github.com/Dicklesworthstone/cass_memory_system/releases/latest/download/cass-memory-macos-arm64 -o ~/.local/bin/cm && chmod +x ~/.local/bin/cm

# Install/update UBS
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/ultimate_bug_scanner/master/install.sh | bash -s -- --easy-mode

# Verify all installed
bd --version && bv --version && cass --version && cm --version && ubs --version
```

### Expected Versions (December 2025)

| Tool | Minimum Version | Install Method |
|------|-----------------|----------------|
| `bd` | 0.29.0 | Homebrew tap `steveyegge/beads` |
| `bv` | 0.10.6 | `bv --update --yes` (preferred) or beads_viewer install script |
| `cass` | 0.1.35 | Install script |
| `cm` | 0.1.0 | Install from cass-memory releases |
| `ubs` | 5.0.0 | Install script |

---

## 3. Directory Structure Setup

Create the proper `.claude/` directory structure:

```bash
cd /path/to/your/project

# Create directories
mkdir -p .claude/rules
mkdir -p .claude/skills
mkdir -p .claude/commands

# If skills/ exists in root, move it
if [ -d "skills" ]; then
  mv skills/* .claude/skills/ 2>/dev/null
  rmdir skills 2>/dev/null
  echo "Moved skills/ to .claude/skills/"
fi
```

### Target Structure

```
your-project/
├── .claude/
│   ├── commands/           # Slash commands
│   │   └── *.md
│   ├── rules/              # Auto-loaded rules
│   │   ├── safety.md
│   │   └── [language].md
│   ├── skills/             # On-demand skills
│   │   └── style-guide/
│   │       ├── SKILL.md    # Note: uppercase
│   │       └── *.md
│   └── settings.local.json
├── .beads/                 # Beads database (auto-created)
├── AGENTS.md               # Agent workflow guide
└── CLAUDE.md               # Project context
```

---

## 4. Rules Migration

### Step 4.1: Identify Rules in Existing Files

Search for rules scattered across your config:

```bash
# Common rule patterns to look for
grep -n "NEVER\|MUST\|forbidden\|Rule #1\|CRITICAL" AGENTS.md CLAUDE.md 2>/dev/null
```

### Step 4.2: Create safety.md (Required)

Every project should have this rule:

```bash
cat > .claude/rules/safety.md << 'EOF'
# Safety Rules

## Rule #1: No File Deletion Without Permission

You are NEVER allowed to delete a file without express permission from the user. Even a new file that you yourself created. You must ALWAYS ask and RECEIVE clear, written permission before deleting any file or folder.

## Irreversible Git & Filesystem Actions

1. **Forbidden commands**: `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that deletes/overwrites code must never be run unless the user explicitly provides the exact command and states they understand the consequences.

2. **No guessing**: If there is any uncertainty about what a command might delete, stop immediately and ask for approval.

3. **Safer alternatives first**: Use non-destructive options (`git status`, `git diff`, `git stash`, backups) before considering destructive commands.

4. **Mandatory explicit plan**: Before any approved destructive command, restate the command verbatim, list what will be affected, and wait for confirmation.

## No Codemods

NEVER run scripts that process/change code files in this repo. Always make code changes manually. If changes are many but simple, use parallel subagents. If changes are subtle/complex, do them methodically yourself.
EOF
```

### Step 4.3: Create Language-Specific Rules

Create rules with path filters for your language:

**For TypeScript/JavaScript projects:**

```bash
cat > .claude/rules/typescript.md << 'EOF'
---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---

# TypeScript/JavaScript Rules

## Package Manager

Use **pnpm** (or npm/yarn - pick one) exclusively. Never mix package managers.

## No V2 Files

NEVER create `componentV2.tsx`, `component_improved.tsx`, `component_enhanced.tsx`, or similar. Revise existing files in place. New files are reserved for genuinely new functionality.

## No Backwards Compatibility

We do NOT care about backwards compatibility during early development. Do things the RIGHT way with NO TECH DEBT. Never create "compatibility shims."

## Type Checking After Changes

After any substantive code changes, run:
```bash
npx tsc --noEmit
npm run lint
```

If you see errors, understand and resolve each one carefully.
EOF
```

**For Python projects:**

```bash
cat > .claude/rules/python.md << 'EOF'
---
paths:
  - "**/*.py"
---

# Python Rules

## Package Manager

Use **uv** exclusively. Never use pip directly or poetry.

```bash
uv sync                    # Install dependencies
uv run pytest              # Run tests
uv add package-name        # Add dependency
```

## Environment Variables

Use `python-decouple` for all environment variable access:
```python
from decouple import config
API_KEY = config('API_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
```

## No V2 Files

NEVER create `module_v2.py`, `module_improved.py`, etc. Revise existing files in place.

## Code Quality

After changes, run:
```bash
ruff check . --fix
ruff format .
```
EOF
```

### Step 4.4: Create Domain-Specific Rules (Optional)

If your project has specific API requirements:

```bash
cat > .claude/rules/api-docs.md << 'EOF'
---
paths:
  - "src/api/**"
  - "app/services/**"
---

# API Documentation Requirement

Before making ANY edits to API integration code, you MUST consult the official documentation.

**Documentation location**: `/path/to/docs/` or `https://api.example.com/docs`

**Mandatory workflow**:
1. Read the relevant documentation sections first
2. Match implementation to documented contracts
3. Verify endpoint URLs and request/response formats
4. Do NOT guess how APIs work
EOF
```

---

## 5. Skills Migration

### Step 5.1: Fix Skill Location and Naming

```bash
# If skill is in wrong location
mv skills/style-guide .claude/skills/ 2>/dev/null

# Rename skill.md to SKILL.md (required)
mv .claude/skills/style-guide/skill.md .claude/skills/style-guide/SKILL.md 2>/dev/null
```

### Step 5.2: Add Required Frontmatter

Skills MUST have YAML frontmatter with `name` and `description`:

```yaml
---
name: style-guide
description: Code style guidelines for [Your Project]. Auto-activates when editing source files, creating components, or working with [your patterns].
---

# Style Guide Skill

[Rest of skill content...]
```

**Frontmatter requirements:**
- `name`: lowercase, numbers, hyphens only, max 64 characters
- `description`: max 1024 characters, CRITICAL for discovery (Claude matches this to decide when to invoke)

### Step 5.3: Skill Template

If you don't have a style guide skill, create one:

```bash
mkdir -p .claude/skills/style-guide

cat > .claude/skills/style-guide/SKILL.md << 'EOF'
---
name: style-guide
description: Code style guidelines for this project. Auto-activates when editing source files, creating new modules, or reviewing code quality.
---

# Style Guide Skill

## When This Activates

This skill auto-activates when you're:
- Editing source files
- Creating new components or modules
- Working on API integrations
- Reviewing code quality

## Quick Reference

| Technology | Version | Purpose |
|------------|---------|---------|
| [Language] | X.x | Main language |
| [Framework] | X.x | Core framework |

## Core Principles

1. **Human Readability First** - Code should read like well-written prose
2. **Type Safety** - Use strict typing, no `any` types
3. **Single Responsibility** - Each function/class does one thing
4. **No Tech Debt** - Do it right the first time

## File Reference

| File | Content |
|------|---------|
| `[name]-style.md` | Language conventions |
| `[framework]-patterns.md` | Framework patterns |
EOF
```

---

## 6. CLAUDE.md Cleanup

CLAUDE.md should be a concise project context file, NOT a rules dump.

### Step 6.1: Remove Duplicated Rules

If CLAUDE.md contains rules that are now in `.claude/rules/`, remove them:

```bash
# Check for rule duplication
grep -n "NEVER\|MUST\|Rule #1" CLAUDE.md
```

### Step 6.2: CLAUDE.md Template

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

**Note**: This project uses [bd (beads)](https://github.com/steveyegge/beads) for issue tracking and [mcp_agent_mail](https://github.com/Dicklesworthstone/mcp_agent_mail) for agent coordination.

**CRITICAL**: Read `AGENTS.md` for the mandatory workflow loop.

## Configuration

| Location | Purpose |
|----------|---------|
| `.claude/rules/` | Auto-loaded rules (safety, [language]) |
| `.claude/skills/` | Skills (auto-activate based on task) |
| `.claude/commands/` | Slash commands |
| `AGENTS.md` | Agent workflow guide |

## Project Overview

- **Name**: [Project Name]
- **Language**: [TypeScript/Python/Go/etc.]
- **Framework**: [Next.js/FastAPI/etc.]
- **Package Manager**: [pnpm/uv/etc.]

## Key Commands

```bash
# Development
[your dev command]

# Testing
[your test command]

# Linting
[your lint command]
```

## Architecture

[Brief architecture overview - keep concise]
```

---

## 7. AGENTS.md Restructuring

AGENTS.md is the agent workflow guide. It should contain:
1. Configuration reference
2. Project-specific workflow
3. Tool documentation for the K&V stack

### Step 7.1: Add Configuration Reference Section

Add this near the top of AGENTS.md (after any startup checklist):

```markdown
## 📁 Configuration Reference

| Location | Purpose |
|----------|---------|
| `.claude/rules/` | Auto-loaded rules (path-filtered) |
| `.claude/skills/` | Skills (auto-activate based on description match) |
| `.claude/commands/` | Slash commands for workflows |
| `AGENTS.md` | This file - agent workflow guide |
| `CLAUDE.md` | Project context and quick reference |

**Rules** (`.claude/rules/`):
- `safety.md` - File deletion, destructive git, no codemods
- `[language].md` - Language-specific rules
- [List your rules]

**Skills** (`.claude/skills/`):
- `style-guide/` - Code style patterns
- [List your skills]

**Commands** (`.claude/commands/`):
- [List your commands]
```

### Step 7.2: Replace Duplicated Rules with Pointers

If AGENTS.md has sections like "CRITICAL SAFETY RULES" that are now in `.claude/rules/`, replace with a compact table:

```markdown
## ⚠️ CRITICAL RULES (Auto-Loaded)

Rules in `.claude/rules/` are automatically loaded. Key rules:

| Rule File | Applies To | Key Points |
|-----------|------------|------------|
| `safety.md` | All files | No file deletion without permission, no destructive git |
| `[language].md` | `**/*.[ext]` | [Key points] |
```

### Step 7.3: Add Missing Tool Sections

Ensure AGENTS.md has documentation for all K&V tools. Copy these sections from the template:

**Required sections:**
1. Task Tracking (Beads) - `bd` commands
2. Using bv as AI sidecar - `bv --robot-*` commands
3. CASS - Cross-Agent Search
4. cass-memory - Cross-Agent Learning
5. UBS - Bug Scanner
6. MCP Agent Mail - Multi-agent coordination
7. Warp-Grep - Parallel code search (MCP)
8. Exa MCP - AI web & code search
9. Session Workflow - Session naming convention

See `templates/AGENTS_TEMPLATE.md` for complete sections to copy.

### Step 7.4: Add Session Naming Section

```markdown
## Session Workflow

### Session Naming (Claude Code 2.0.64+)

Name sessions immediately after Agent Mail registration for traceability:

```
{project}-{AgentMailName}-{YYYYMMDD-HHMMSS}
```

Examples:
- `myproject-GreenCastle-20251210-143022`
- `myproject-BlueLake-20251210-091500`

After `register_agent` or `macro_start_session` returns your agent name:

```bash
/rename myproject-GreenCastle-20251210-143022
```

To resume later:

```bash
claude --resume myproject-GreenCastle-20251210-143022
```
```

---

## 8. Settings Configuration

### Step 8.1: Create/Update settings.local.json

```bash
cat > .claude/settings.local.json << 'EOF'
{
  "permissions": {
    "allow": [
      "Read",
      "Write",
      "Edit",
      "Bash(bd:*)",
      "Bash(bd update:*)",
      "Bash(bd ready:*)",
      "Bash(bd list:*)",
      "Bash(bd close:*)",
      "Bash(bd show:*)",
      "Bash(bd create:*)",
      "Bash(bv:*)",
      "Bash(cass:*)",
      "Bash(cm:*)",
      "Bash(ubs:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(ls:*)",
      "Bash(cat:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Bash(wc:*)",
      "Bash(curl:*)",
      "mcp__mcp-agent-mail__ensure_project",
      "mcp__mcp-agent-mail__register_agent",
      "mcp__mcp-agent-mail__fetch_inbox",
      "mcp__mcp-agent-mail__send_message",
      "mcp__mcp-agent-mail__reply_message",
      "mcp__mcp-agent-mail__acknowledge_message",
      "mcp__mcp-agent-mail__mark_message_read",
      "mcp__mcp-agent-mail__search_messages",
      "mcp__mcp-agent-mail__summarize_thread",
      "mcp__mcp-agent-mail__file_reservation_paths",
      "mcp__mcp-agent-mail__release_file_reservations",
      "mcp__mcp-agent-mail__renew_file_reservations",
      "mcp__mcp-agent-mail__list_contacts",
      "mcp__mcp-agent-mail__whois",
      "mcp__mcp-agent-mail__set_contact_policy",
      "mcp__mcp-agent-mail__macro_start_session",
      "mcp__mcp-agent-mail__acquire_build_slot",
      "mcp__mcp-agent-mail__release_build_slot",
      "mcp__exa__web_search_exa",
      "mcp__exa__get_code_context_exa",
      "WebSearch"
    ]
  },
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": [
    "mcp-agent-mail"
  ]
}
EOF
```

### Step 8.2: Add Language-Specific Permissions

**For TypeScript/Node.js:**
```json
"Bash(npm:*)",
"Bash(npx:*)",
"Bash(pnpm:*)",
"Bash(npx tsc:*)",
"Bash(npm run lint:*)",
"Bash(npm run build:*)",
"Bash(npm run dev:*)"
```

**For Python:**
```json
"Bash(python:*)",
"Bash(python3:*)",
"Bash(uv:*)",
"Bash(uv run:*)",
"Bash(uv sync:*)",
"Bash(pytest:*)",
"Bash(ruff:*)"
```

---

## 9. Beads Initialization

If `.beads/` doesn't exist, initialize it:

```bash
cd /path/to/your/project

# Initialize beads
bd init

# Verify
ls -la .beads/
bd info
```

### Beads Best Practices

1. **Always commit `.beads/`** with code changes
2. **Never edit `.beads/*.jsonl`** directly - only via `bd` commands
3. **Use `bd doctor --fix`** if issues arise

---

## 10. Verification

### Final Checklist

Run this verification script:

```bash
cd /path/to/your/project

echo "=== Directory Structure ==="
find .claude -type f | sort

echo ""
echo "=== File Line Counts ==="
wc -l AGENTS.md CLAUDE.md 2>/dev/null

echo ""
echo "=== Tools Installed ==="
bd --version && bv --version && cass --version && cm --version && ubs --version

echo ""
echo "=== Beads Database ==="
ls -la .beads/beads.db 2>/dev/null || echo "No beads database - run 'bd init'"

echo ""
echo "=== Settings Permissions ==="
grep -c "Bash(bd" .claude/settings.local.json 2>/dev/null && echo "bd permissions found"
grep -c "Bash(bv" .claude/settings.local.json 2>/dev/null && echo "bv permissions found"
grep -c "Bash(cass" .claude/settings.local.json 2>/dev/null && echo "cass permissions found"
grep -c "Bash(cm" .claude/settings.local.json 2>/dev/null && echo "cm permissions found"
grep -c "Bash(ubs" .claude/settings.local.json 2>/dev/null && echo "ubs permissions found"
```

### Expected Output Structure

```
.claude/commands/*.md           # Your slash commands
.claude/rules/safety.md         # Required
.claude/rules/[language].md     # Language-specific
.claude/skills/style-guide/SKILL.md  # Main skill file
.claude/settings.local.json     # Tool permissions
```

### Test the Stack

```bash
# Test beads
bd ready --json

# Test bv (use robot flags!)
bv --robot-help

# Test cass
cass search "test" --robot --limit 1

# Test cass-memory
cm doctor

# Test ubs
ubs --help
```

---

## Troubleshooting

### "bd: command not found"

```bash
# Check if installed via homebrew
brew list bd 2>/dev/null || brew install steveyegge/beads/bd
```

### "bv: command not found"

```bash
# Reinstall via agent mail script
curl -fsSL https://raw.githubusercontent.com/Dicklesworthstone/beads_viewer/main/install.sh | bash
```

### "cass returns no results"

```bash
# Rebuild index
cass index --full
```

### "cm: command not found"

```bash
# Download latest release
curl -L https://github.com/Dicklesworthstone/cass_memory_system/releases/latest/download/cass-memory-macos-arm64 -o ~/.local/bin/cm && chmod +x ~/.local/bin/cm
```

### Skills not activating

1. Check location: must be in `.claude/skills/*/SKILL.md`
2. Check filename: must be `SKILL.md` (uppercase)
3. Check frontmatter: must have `name:` and `description:`
4. Check description: must match the task context

### Rules not loading

1. Check location: must be in `.claude/rules/*.md`
2. Check path filters: YAML frontmatter `paths:` must match your files
3. Restart Claude Code session after adding rules

---

## Quick Reference

### Migration Commands

```bash
# Create structure
mkdir -p .claude/{rules,skills,commands}

# Move skills
mv skills/* .claude/skills/ && rmdir skills

# Rename skill file
mv .claude/skills/*/skill.md .claude/skills/*/SKILL.md

# Initialize beads
bd init

# Verify tools
bd --version && bv --version && cass --version && cm --version && ubs --version
```

### File Purposes

| File | Purpose | Edit Frequency |
|------|---------|----------------|
| `CLAUDE.md` | Project context | Rarely |
| `AGENTS.md` | Workflow guide | Occasionally |
| `.claude/rules/*.md` | Constraints | Rarely |
| `.claude/skills/*/SKILL.md` | Detailed guides | Occasionally |
| `.claude/commands/*.md` | Slash commands | As needed |
| `.claude/settings.local.json` | Permissions | Once |

---

## See Also

- [SETUP_GUIDE.md](SETUP_GUIDE.md) - Fresh project setup
- [CLAUDE_CONFIG_GUIDE.md](CLAUDE_CONFIG_GUIDE.md) - Configuration reference
- [templates/AGENTS_TEMPLATE.md](../../templates/AGENTS_TEMPLATE.md) - AGENTS.md template
- [README.md](../../README.md) - Stack overview
