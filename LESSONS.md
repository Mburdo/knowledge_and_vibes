# Lessons Learned

Hard-won knowledge from multi-agent coordination. Search this file with CASS when troubleshooting.

---

## bd sync - When to Use (and When NOT to)

**Date:** 2025-12-17
**Tags:** beads, bd sync, multi-agent, git, coordination

### The Problem

Running `bd sync` in multi-agent environments causes race conditions and conflicts:

```
Agent A: bd sync (starts git pull)
Agent B: bd sync (starts git pull)
Agent A: bd sync (commits + pushes)
Agent B: bd sync (push fails - HEAD ref conflict)
```

### Root Cause

`bd sync` does 5 operations atomically:
1. Export to JSONL
2. Git commit
3. Git pull
4. Import from JSONL
5. Git push

In concurrent environments, steps 3-5 race between agents.

### The Solution

**DON'T use `bd sync` in multi-agent workflows.**

Instead:
```bash
# After bd update/create/close (auto-exports to JSONL)
git add -A && git commit -m "your message"
git pull --rebase && git push
```

### When bd sync IS Appropriate

1. **Single-agent workflows** - No race conditions
2. **After merge conflict resolution** - Use `bd sync --import-only` to reload the JSONL

### Why Auto-Export Works

After every `bd update`, `bd create`, or `bd close`:
- Changes auto-export to `.beads/issues.jsonl` (5-second debounce)
- SQLite DB is just a local cache
- JSONL is the source of truth (committed to git)

### Hash-Based IDs Prevent Collisions

Beads uses content-addressable IDs:
- Two agents creating beads simultaneously get unique IDs
- No central counter to conflict on
- Safe for parallel branch work

### Symptoms of bd sync Misuse

- "HEAD ref conflict" errors
- Bead state reverting after sync
- Lost bead updates
- Inconsistent `.beads/issues.jsonl` between agents

### References

- Official docs: https://github.com/steveyegge/beads
- FAQ: "Do I need to run bd sync manually?" → No, auto-sync handles it
- GIT_INTEGRATION.md explains the full workflow

---

## Template for New Lessons

```markdown
## Title - Brief Description

**Date:** YYYY-MM-DD
**Tags:** tag1, tag2, tag3

### The Problem
What went wrong or was confusing.

### Root Cause
Why it happened.

### The Solution
How to fix or avoid it.

### Symptoms
How to recognize this issue.

### References
Links to docs, PRs, or related lessons.
```
