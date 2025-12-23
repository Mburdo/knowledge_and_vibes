---
title: AI Code Review 2025 - State of Practice
tags: [code-review, review, security, comments, adoption, production]
protocols: [P13, P14]
---

# AI Code Review 2025: State of Practice

**Sources:**
- BitsAI-CR: Automated Code Review via LLM (FSE 2025, ByteDance)
- "State of AI Code Review 2025" (Pullflow analysis)
- "Does AI Code Review Lead to Code Changes?" (arXiv August 2025)

---

## Summary

Comprehensive 2025 analysis of AI-powered code review in production. Key finding: **1 in 7 PRs (14.9%) now involve AI agents**, up 14x from early 2024.

**Critical insight:** AI review is mainstream, but effectiveness varies significantly based on comment format and triggering.

---

## Industry Adoption (2025)

### Growth Trajectory

| Period | AI Agent PR Participation |
|--------|--------------------------|
| Feb 2024 | 1.1% |
| Nov 2025 | **14.9%** |
| Growth | **14x** |

### Top AI Review Agents (by PR volume)

1. **CodeRabbit** — Standalone review tool
2. **GitHub Copilot** — IDE-integrated
3. **Google Gemini** — Multi-modal review
4. **Cursor AI** — IDE-native
5. **Greptile** — Codebase-aware

---

## BitsAI-CR: Production System

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        BitsAI-CR                                 │
│                    (ByteDance Production)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Stage 1: RuleChecker                                            │
│  ─────────────────────                                           │
│  ├── 500+ review rules taxonomy                                  │
│  ├── LLM matches code to rules                                   │
│  └── High recall, moderate precision                             │
│                                                                  │
│  Stage 2: ReviewFilter                                           │
│  ─────────────────────                                           │
│  ├── Precision verification                                      │
│  ├── False positive filtering                                    │
│  └── Comment quality scoring                                     │
│                                                                  │
│  Stage 3: Data Flywheel                                          │
│  ─────────────────────                                           │
│  ├── Track developer acceptance                                  │
│  ├── Outdated Rate metric                                        │
│  └── Continuous improvement                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Production Scale

| Metric | Value |
|--------|-------|
| Daily reviews | 24,000+ |
| Users served | 12,000+ |
| Code changes requested | 400K+ |
| Adoption rate | Growing |

### Outdated Rate Metric

Novel metric to track adoption:

```
Outdated Rate = Comments marked outdated by code changes
             / Total comments generated

Lower = developers accept more suggestions
Higher = suggestions ignored or wrong
```

---

## What Makes AI Review Effective

### From GitHub Actions Study

Analysis of 22,000+ review comments across 178 repositories:

| Factor | Impact on Code Changes |
|--------|----------------------|
| **Concise comments** | Higher adoption |
| **Contains code snippets** | Higher adoption |
| **Manually triggered** | Higher adoption |
| **Auto-triggered on all PRs** | Lower adoption |
| **Vague suggestions** | Often ignored |

### Comment Format Best Practices

```markdown
## Effective AI Review Comment

### Issue
Potential SQL injection on line 42.

### Suggested Fix
```python
# Before (vulnerable)
query = f"SELECT * FROM users WHERE id = {user_id}"

# After (safe)
query = "SELECT * FROM users WHERE id = ?"
cursor.execute(query, (user_id,))
```

### Why This Matters
User input is directly interpolated into SQL query,
enabling attackers to execute arbitrary SQL commands.
```

### Ineffective Comment
```markdown
Consider improving the code quality in the database access section.
```

---

## Speed vs Quality Tradeoffs

### Review Time Comparison

| Review Type | Time to First Feedback | Quality |
|-------------|----------------------|---------|
| Human only | 4-24 hours | High |
| AI only | 30 seconds | Variable |
| AI + Human | 30 seconds initial + human validation | High |

### Recommended Workflow

```
PR Opened
    │
    ▼
AI Review (30 seconds)
├── Security issues
├── Bug detection
├── Style violations
└── Performance concerns
    │
    ▼
Human Review (focused)
├── Architecture decisions
├── Business logic
├── Edge cases AI missed
└── Final approval
```

---

## What AI Review Catches

### Strong Detection Areas

| Category | AI Detection Rate |
|----------|------------------|
| Security vulnerabilities | 95% improvement |
| Null pointer issues | High |
| Resource leaks | High |
| Style violations | Very High |
| API misuse | High |

### Weak Detection Areas

| Category | Why AI Struggles |
|----------|-----------------|
| Business logic errors | Lacks domain context |
| Architecture violations | Needs system-level view |
| Performance at scale | Can't profile |
| Subtle race conditions | Requires deep reasoning |

---

## Practical Implications

### For Knowledge & Vibes

AI code review complements our security gates:

| Stage | Human | AI |
|-------|-------|-----|
| Pre-commit | TDD | `ubs --staged` |
| PR review | Architecture, logic | Security, bugs, style |
| Calibration | Requirements alignment | Automated checks |

### Integration Pattern

```markdown
## Bead Close Checklist

- [x] Tests pass
- [x] `ubs --staged` clean
- [x] AI review issues addressed
- [x] Human review (if required for complexity)
- [x] Commit
```

---

## Key Takeaways

1. **AI review is mainstream** — 14.9% of PRs in late 2025
2. **Two-stage works best** — Detection + verification
3. **Comment format matters** — Code snippets > vague suggestions
4. **Manual trigger > auto** — Targeted reviews more effective
5. **AI + Human hybrid** — Best of both worlds
6. **Track adoption** — Outdated Rate shows real impact

---

## See Also

- `061-llm-security-2025.md` — Security scanning
- `052-llm-security-vulnerabilities.md` — What AI misses
- `.claude/skills/ubs-scanner/SKILL.md` — UBS tool
