---
description: Pre-ship checklist for release readiness (Protocol P12)
argument-hint: [rigor_tier]
---

# /release

Run the `release` skill for final verification before shipping.

**Rigor Tier:** $ARGUMENTS (defaults to project's configured tier)

Execute the full protocol from `.claude/skills/release/SKILL.md`:
1. Gather status: Verify all beads closed, check for orphaned work
2. Run verification: Test suite, security scan (`ubs --staged`)
3. Multi-agent cleanup: Check inbox, release file reservations
4. Generate report: Release readiness summary with blockers

**Gate:** Zero blockers before shipping. Security scan must be clean.
