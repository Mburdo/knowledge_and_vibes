---
title: Claude Code Configuration Guide
description: How to organize agent instructions using Claude Code's official configuration system - rules, skills, and commands.
category: guides
---

# Claude Code Configuration Guide

How to organize agent instructions using Claude Code's official configuration system: rules, skills, and commands.

---

## Table of Contents

1. [The Configuration Layers](#the-configuration-layers)
2. [Recommended Structure](#recommended-structure)
3. [CLAUDE.md ,  Project Context](#claudemd--project-context)
4. [Rules ,  Constraints & Conventions](#rules--constraints--conventions)
5. [Skills ,  Packaged Capabilities](#skills--packaged-capabilities)
6. [Commands ,  Slash Commands](#commands--slash-commands)
7. [What Goes Where](#what-goes-where)
8. [Setup Checklist](#setup-checklist)

---

## Quick Reference

| Layer | Location | When Loaded |
|-------|----------|-------------|
| Memory | `CLAUDE.md` | Auto at startup |
| Rules | `.claude/rules/*.md` | Auto (path-filtered) |
| Skills | `.claude/skills/*/SKILL.md` | When relevant to task |
| Commands | `.claude/commands/*.md` | On `/command` |

---

## The Configuration Layers

| Layer | Location | Purpose | When Loaded |
|-------|----------|---------|-------------|
| **Memory** | `CLAUDE.md` | Project context | Auto at startup |
| **Rules** | `.claude/rules/*.md` | Constraints/conventions | Auto (can be path-filtered) |
| **Skills** | `.claude/skills/*/SKILL.md` | Capabilities with docs | When Claude decides it's relevant |
| **Commands** | `.claude/commands/*.md` | User-triggered workflows | On `/command` |
| **Templates** | `.claude/templates/**/*.md` | Runtime templates for commands | Read by commands as needed |

**Note**: `AGENTS.md` is an **industry convention** for AI coding agents. Claude Code reads it automatically if present. Use it for agent workflow instructions (startup checklists, tool usage, coordination).

---

## Recommended Structure

```
your-project/
├── CLAUDE.md                    # Project context (official)
├── AGENTS.md                    # Workflow instructions (industry convention)
└── .claude/
    ├── settings.local.json      # Permissions
    ├── rules/                   # Constraints (official)
    │   └── safety.md
    ├── skills/                  # Capabilities (official)
    │   └── style-guide/
    │       └── SKILL.md
    ├── commands/                # Slash commands (official)
    │   └── audit.md
    └── templates/               # Runtime templates for commands
        ├── beads/               # Bead lifecycle (claimed, closed, etc.)
        ├── calibration/         # Calibration roundtable templates
        └── planning/            # Phase/bead breakdown templates
```

---

## CLAUDE.md ,  Project Context

**Official feature.** Automatically loaded at startup.

### Locations (in precedence order)

1. Enterprise: `/etc/claude-code/CLAUDE.md` (Linux/macOS)
2. Project: `./CLAUDE.md` or `./.claude/CLAUDE.md`
3. User: `~/.claude/CLAUDE.md`
4. Project local: `./CLAUDE.local.md` (git-ignored, personal)

### What to Include

- Project overview and architecture
- Tech stack and dependencies
- Common commands (build, test, run)
- Key files and directories

### File Imports

Reference other files with `@`:

```markdown
# CLAUDE.md

See the API documentation: @docs/api.md
```

---

## Rules ,  Constraints & Conventions

**Official feature.** Rules are short, direct constraints that always apply (or apply to specific file paths).

### Locations

- Project: `.claude/rules/*.md`
- Personal: `~/.claude/rules/*.md`

### Format

```markdown
---
paths: src/api/**/*.py
---

# Rule Title

Clear, direct instructions.
```

### Path Filtering (Official)

The `paths` frontmatter uses glob patterns:

```yaml
# Single pattern
paths: **/*.py

# Multiple patterns
paths:
  - src/**/*.ts
  - lib/**/*.ts

# No paths field = applies to everything
```

Supported patterns:
- `**/*.ts` ,  all TypeScript files
- `src/**/*` ,  all files under src/
- `{src,lib}/**/*.ts` ,  multiple directories

### Example Rules

**`.claude/rules/safety.md`** (no paths - always applies):
```markdown
# Safety Rules

## File Deletion
Never delete files without explicit user permission in this session.

## Destructive Commands
Forbidden without explicit approval:
- `git reset --hard`
- `git clean -fd`
- `rm -rf`
```

**`.claude/rules/typescript.md`** (path-specific):
```markdown
---
paths: **/*.ts
---

# TypeScript Rules

- Strict mode, no `any` without justification
- Async/await for all promises
- Zod for runtime validation at boundaries
```

---

## Skills ,  Packaged Capabilities

**Official feature.** Skills are capabilities that Claude discovers and invokes automatically based on the task.

### Locations

- Project: `.claude/skills/skill-name/SKILL.md`
- Personal: `~/.claude/skills/skill-name/SKILL.md`

### Directory Structure

```
.claude/skills/my-skill/
├── SKILL.md           # Required
├── reference.md       # Optional
├── examples.md        # Optional
└── scripts/           # Optional
    └── helper.sh
```

### SKILL.md Format (Required Fields)

```markdown
---
name: skill-name
description: What this skill does and when to use it. Be specific.
---

# Skill Title

Content here...
```

**Required frontmatter:**
- `name` ,  lowercase, numbers, hyphens only, max 64 chars
- `description` ,  max 1024 chars, **critical for discovery**

**Optional frontmatter:**
- `allowed-tools` ,  restrict which tools the skill can use

### Tool Restrictions

```markdown
---
name: read-only-reviewer
description: Review code without making changes.
allowed-tools: Read, Grep, Glob
---
```

When `allowed-tools` is specified, Claude can only use those tools without asking permission.

### How Skills Are Discovered

Claude decides to invoke a skill based on:
1. The user's request
2. The skill's `description` field

**The description is critical.** Write it to clearly indicate:
- What the skill does
- When Claude should use it

```yaml
# Good description
description: Python coding standards and patterns. Use when writing new Python code, reviewing Python files, or refactoring Python modules.

# Bad description
description: Style guide for code.
```

### Example Skill

**`.claude/skills/style-guide/SKILL.md`**:
```markdown
---
name: style-guide
description: Code style standards. Use when writing new code, reviewing code, or refactoring existing modules.
allowed-tools: Read, Grep, Glob, Edit
---

# Style Guide

## Core Principles

1. Readability over cleverness
2. Explicit over implicit
3. Single responsibility per function

## Files

| File | Content |
|------|---------|
| `python.md` | Python conventions |
| `naming.md` | Naming standards |

## Quick Reference

- `snake_case` for functions/variables
- `PascalCase` for classes
- Type hints on all signatures
```

---

## Commands ,  Slash Commands

**Official feature.** User-triggered workflows invoked with `/command-name`.

### Locations

- Project: `.claude/commands/command-name.md`
- Personal: `~/.claude/commands/command-name.md`
- Subdirectories supported for namespacing

### Basic Format

```markdown
---
description: Brief description of command
---

# Command instructions

Do something with the arguments: $ARGUMENTS
```

### Arguments

**All arguments:**
```markdown
Create a commit with message: $ARGUMENTS
```

**Positional arguments:**
```markdown
Review PR #$1 with priority $2 assigned to $3
```

### Frontmatter Options

```yaml
---
description: Brief description shown in /help
allowed-tools: Bash(git add:*), Bash(git status:*)
argument-hint: [message]
model: claude-3-5-haiku-20241022
---
```

| Field | Purpose |
|-------|---------|
| `description` | Shown in command list |
| `allowed-tools` | Required for bash execution |
| `argument-hint` | Placeholder shown to user |
| `model` | Override model for this command |

### Bash Execution

Use `!` prefix to execute bash and include output:

```markdown
---
allowed-tools: Bash(git status:*)
---

Current git status:
!`git status`

Now review the changes above.
```

**Note:** Must include `allowed-tools` with appropriate Bash permissions.

### File References

Use `@` prefix to include file contents:

```markdown
Review the helper functions:
@src/utils/helpers.js

Check for issues.
```

### Example Command

**`.claude/commands/audit-style.md`**:
````markdown
---
description: Audit code against style standards
allowed-tools: Read, Grep, Glob
argument-hint: <file-or-directory>
---

# Audit Code Style

## Target
Audit: $ARGUMENTS

## Instructions

1. Read the style guide from `.claude/skills/style-guide/`
2. Read the target file(s)
3. Check compliance with each standard
4. Report issues with line numbers
5. Offer to fix issues

## Output Format

```
# Style Audit: {path}

## Issues
- {file}:{line} - {issue}

## Passed
- {check name}
```
````

---

## What Goes Where

| Content Type | Location | Why |
|-------------|----------|-----|
| Project architecture | `CLAUDE.md` | Auto-loaded context |
| Build/test commands | `CLAUDE.md` | Quick reference |
| Session workflow | `AGENTS.md` | Industry convention |
| Tool quick reference | `AGENTS.md` | Industry convention |
| Safety constraints | `.claude/rules/` | Always enforced |
| Coding conventions | `.claude/rules/` | Path-specific |
| Detailed style guide | `.claude/skills/` | Auto-discovered |
| Tool documentation | `.claude/skills/` | Auto-discovered |
| Audit procedures | `.claude/commands/` | User-triggered |
| Code generators | `.claude/commands/` | User-triggered |
| Runtime templates | `.claude/templates/` | Loaded by commands on-demand |

---

## Setup Checklist

### Minimal Setup

```bash
# Create directory
mkdir -p .claude/rules

# Add safety rules
cat > .claude/rules/safety.md << 'EOF'
# Safety Rules

Never delete files without explicit user permission.
Never run destructive git commands without approval.
EOF

# Create CLAUDE.md
cat > CLAUDE.md << 'EOF'
# Project Name

## Overview
Brief description.

## Commands
- `npm run dev` - Start dev server
- `npm test` - Run tests
EOF
```

### Add a Skill

```bash
mkdir -p .claude/skills/style-guide

cat > .claude/skills/style-guide/SKILL.md << 'EOF'
---
name: style-guide
description: Code style standards. Use when writing or reviewing code.
---

# Style Guide

## Naming
- camelCase for functions/variables
- PascalCase for classes
EOF
```

### Add a Command

```bash
mkdir -p .claude/commands

cat > .claude/commands/new-feature.md << 'EOF'
---
description: Scaffold a new feature
argument-hint: <feature-name>
---

# Create feature: $ARGUMENTS

1. Create directory `src/features/$ARGUMENTS/`
2. Add index file with exports
3. Add test file
EOF
```

---

## Tips

### Rules: Keep Short
If you need detailed explanations, use a skill instead.

### Skills: Description Matters
Claude finds skills by matching the description to the task. Be specific about what it does AND when to use it.

### Commands: Use Frontmatter
`allowed-tools` is required for bash execution. `argument-hint` helps users know what to pass.

### Don't Duplicate
If something is in a rule, don't repeat it in CLAUDE.md.

---

## Reference

### Rules Frontmatter

```yaml
---
paths: src/**/*.ts       # Optional: glob pattern(s)
---
```

### Skills Frontmatter

```yaml
---
name: skill-name         # Required: lowercase, hyphens, max 64 chars
description: ...         # Required: max 1024 chars
allowed-tools: Read, Grep  # Optional: restrict tools
---
```

### Commands Frontmatter

```yaml
---
description: ...         # Optional: shown in /help
allowed-tools: ...       # Required for bash
argument-hint: ...       # Optional: placeholder
model: ...               # Optional: override model
---
```

### Special Syntax

| Syntax | Where | Purpose |
|--------|-------|---------|
| `$ARGUMENTS` | Commands | All arguments |
| `$1, $2, $3` | Commands | Positional arguments |
| `!`backticks | Commands | Execute bash |
| `@path` | Commands, CLAUDE.md | Include file contents |
