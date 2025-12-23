---
title: SWE-Bench Pro 2025 - Realistic Agent Evaluation
tags: [benchmark, enterprise, complex-tasks, multi-file, realistic, verification]
protocols: [P12, P14]
---

# SWE-Bench Pro 2025: Realistic Agent Evaluation

**Sources:**
- SWE-Bench Pro OpenReview paper (2025)
- Scale AI Leaderboard (December 2025)
- Vals.ai SWE-Bench analysis (December 2025)

---

> **⚠️ Historical Note (December 2025):** Model capabilities improve rapidly. The specific success rates cited here reflect a snapshot in time and are likely outdated. The methodological insight—that harder benchmarks reveal more about real capability—remains valuable. Adjust verification based on task complexity, not static percentages.

---

## Summary

**SWE-Bench Pro** is a more challenging benchmark designed to evaluate AI coding agents on **long-horizon, complex, enterprise-level** software engineering tasks.

**Key insight:** Harder, more realistic benchmarks reveal capability gaps that simpler benchmarks miss. Task complexity should drive verification requirements.

---

## The Reality Check

### Performance Comparison (December 2025)

| Model | SWE-Bench Verified | SWE-Bench Pro | Drop |
|-------|-------------------|---------------|------|
| Claude Opus 4.5 | 74.6% | ~23% | **-51%** |
| GPT-5 | 74.9% | ~23% | **-52%** |
| Gemini 3 Pro | 76.2% | ~22% | **-54%** |

**Implication:** Don't trust benchmark scores as real-world predictors. Verify everything.

---

## What Makes Pro Harder

### SWE-Bench vs SWE-Bench Pro

| Dimension | SWE-Bench | SWE-Bench Pro |
|-----------|-----------|---------------|
| Task complexity | Single issue | Multi-issue |
| Code changes | Few files | Many files |
| Dependencies | Minimal | Cross-module |
| Context needed | Local | Repository-wide |
| Time horizon | Short | Long |
| Domain | Open source | Enterprise-style |

### Task Characteristics

```
SWE-Bench (Typical):
├── 1 issue to fix
├── 1-3 files to modify
├── Clear error message
└── Tests provided

SWE-Bench Pro (Typical):
├── Complex feature request
├── 5-15 files to modify
├── Ambiguous requirements
├── Cross-cutting concerns
├── Integration testing needed
└── Performance constraints
```

---

## Benchmark Statistics

| Metric | Value |
|--------|-------|
| Total problems | 1,865 |
| Repositories | 41 |
| Avg files per patch | 8.3 |
| Avg lines changed | 247 |
| Domain coverage | Business, B2B, DevTools |

### Repository Types

```
Enterprise Categories:
├── Business applications (CRM, ERP)
├── B2B services (APIs, integrations)
├── Developer tools (IDEs, CLIs)
├── Infrastructure (deployment, monitoring)
└── Data pipelines (ETL, analytics)
```

---

## Why Models Fail

### Analysis of Failure Modes

| Failure Mode | Frequency | Root Cause |
|--------------|-----------|------------|
| Incomplete changes | 34% | Miss cross-file dependencies |
| Wrong localization | 28% | Misunderstand codebase structure |
| Semantic errors | 19% | Misinterpret requirements |
| Integration breaks | 12% | Changes break other components |
| Performance issues | 7% | Meet function but not NFRs |

### The Localization Problem

```
Simple Task (SWE-Bench):
├── Error in auth/login.py:42
└── Fix: Modify line 42

Complex Task (SWE-Bench Pro):
├── Feature: "Add SSO support"
├── Touches:
│   ├── auth/sso.py (new)
│   ├── auth/login.py (modify)
│   ├── auth/middleware.py (modify)
│   ├── config/settings.py (modify)
│   ├── api/routes.py (modify)
│   ├── tests/test_sso.py (new)
│   └── docs/auth.md (modify)
└── Dependencies: 3 external services
```

---

## Implications for Practice

### Don't Trust High Benchmarks

```markdown
## Reality Check Protocol

Before trusting AI on a task:

1. Assess task complexity
   - Single file? → AI likely capable
   - Multi-file? → Verify carefully
   - Cross-module? → Human oversight essential

2. Check for Pro-like characteristics
   - Ambiguous requirements? → Clarify first
   - Integration dependencies? → Test thoroughly
   - Performance constraints? → Measure explicitly

3. Verification gates
   - All tests pass
   - Integration tests pass
   - Performance benchmarks met
   - Security scan clean
```

### Calibration for Real Tasks

| Task Complexity | Expected AI Success | Verification Level |
|-----------------|--------------------|--------------------|
| Single-file bug fix | 70-80% | Standard |
| Multi-file feature | 30-50% | Thorough |
| Cross-module refactor | 15-25% | Extensive |
| Architecture change | <10% | Human-led |

---

## For Knowledge & Vibes

### Alignment with Our Approach

SWE-Bench Pro validates our design:

| Pro Failure Mode | K&V Mitigation |
|------------------|----------------|
| Incomplete changes | Bead traceability (REQ → code) |
| Wrong localization | Edit locus in bead description |
| Semantic errors | TDD (tests define correctness) |
| Integration breaks | Calibration at phase boundaries |
| Performance issues | NFRs in acceptance criteria |

### Decomposition Matters

Pro tasks require proper decomposition:

```
Pro Task: "Add SSO support"
    │
    └── K&V Decomposition:
        ├── phase-1.1: SSO provider integration
        ├── phase-1.2: Login flow modification
        ├── phase-1.3: Middleware updates
        ├── phase-1.4: Configuration changes
        ├── phase-1.5: Route updates
        ├── phase-1.6: Unit tests
        ├── phase-1.7: Integration tests
        └── phase-1.8: Documentation

Each bead:
- Clear scope
- Testable completion
- Tracked dependencies
```

---

## Key Takeaways

1. **Real-world is harder** — 50%+ drop from Verified to Pro
2. **Don't trust benchmarks blindly** — They don't predict real performance
3. **Multi-file is the challenge** — Single-file tasks mask true capability
4. **Decomposition essential** — Complex tasks need structure
5. **Verification critical** — Can't assume AI got it right
6. **Human oversight required** — For complex, cross-cutting work

---

## See Also

- `050-swe-bench-pro.md` — Earlier Pro analysis
- `051-metr-rct.md` — AI productivity reality check
- `021-swe-bench-plus.md` — Benchmark contamination issues
