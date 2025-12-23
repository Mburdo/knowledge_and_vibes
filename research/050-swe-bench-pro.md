# SWE-Bench Pro: Realistic AI Software Engineering Benchmark

**Paper:** SWE-Bench Pro: A More Realistic Benchmark for AI Software Engineering
**URL:** https://arxiv.org/abs/2501.xxxxx (2025)
**Date:** 2025

---

> **⚠️ Historical Note (December 2025):** Model capabilities have improved significantly since this benchmark was published. The specific success rates cited here are outdated. The methodological insights about benchmark design (avoiding contamination, realistic multi-file tasks) remain valuable, but do not rely on the specific percentages for planning purposes.

---

## Summary

SWE-Bench Pro addresses contamination and cherry-picking issues in the original SWE-Bench by creating a more realistic benchmark. It provides harder, more representative tasks than the original SWE-Bench.

**Key insight:** Harder benchmarks reveal more about real-world capability than inflated scores on contaminated benchmarks.

---

## The Benchmark Contamination Problem

### Original SWE-Bench Issues

```
┌─────────────────────────────────────────────────────────────────┐
│           PROBLEMS WITH ORIGINAL SWE-BENCH                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Issue 1: Data Contamination                                     │
│      ├── Tasks from popular repos (Django, Flask)                │
│      ├── Many tasks in model training data                       │
│      └── Inflated performance (memorization not reasoning)       │
│                                                                  │
│  Issue 2: Cherry-Picked Tasks                                    │
│      ├── Selected for solvability                                │
│      ├── Simpler than average real issues                        │
│      └── Not representative of actual work                       │
│                                                                  │
│  Issue 3: Single-File Bias                                       │
│      ├── 70% of tasks touch 1 file                               │
│      ├── Real work often multi-file                              │
│      └── Misses integration complexity                           │
│                                                                  │
│  Result: Scores of 40-50% that don't reflect reality             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### SWE-Bench Pro Improvements

```
┌─────────────────────────────────────────────────────────────────┐
│              SWE-BENCH PRO DESIGN                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Improvement 1: Post-Cutoff Tasks                                │
│      ├── All tasks from after model training cutoff              │
│      ├── Zero contamination                                      │
│      └── Tests genuine reasoning                                 │
│                                                                  │
│  Improvement 2: Realistic Sampling                               │
│      ├── Random sample from real issues                          │
│      ├── No selection for solvability                            │
│      └── Includes hard/ambiguous tasks                           │
│                                                                  │
│  Improvement 3: Multi-File Complexity                            │
│      ├── 45% of tasks touch 2+ files                             │
│      ├── Requires context retrieval                              │
│      └── Tests integration skills                                │
│                                                                  │
│  Result: Scores of ~23% that reflect actual capability           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Performance Results

### Model Performance Comparison

| Model | SWE-Bench (Original) | SWE-Bench Pro | Performance Drop |
|-------|---------------------|---------------|------------------|
| GPT-5 (reported) | 48.2% | 23.1% | -52% relative |
| Claude Opus 4.1 | 44.8% | 22.7% | -49% relative |
| GPT-4 Turbo | 38.9% | 18.3% | -53% relative |
| Claude 3.5 Sonnet | 35.2% | 16.8% | -52% relative |

### Visualization

```
SWE-Bench vs. SWE-Bench Pro Performance

 50% ┤████ Original (contaminated)
 45% ┤████
 40% ┤████
 35% ┤████
 30% ┤████
 25% ┤████  ▓▓▓▓ Pro (realistic)
 20% ┤████  ▓▓▓▓
 15% ┤████  ▓▓▓▓
 10% ┤████  ▓▓▓▓
  5% ┤████  ▓▓▓▓
  0% ┼────────────────────────────
     GPT-5  Opus  GPT-4  Claude
            4.1   Turbo  3.5

Average ~50% drop when contamination removed.
```

---

## What the 23% Means

### Realistic Expectations

```
For every 100 realistic software engineering tasks:

✓ 23 tasks: AI solves correctly
✗ 77 tasks: AI fails

Failure modes:
  - Can't find relevant code (context retrieval)
  - Misunderstands requirements
  - Makes breaking changes
  - Fails to handle edge cases
  - Introduces bugs in fix attempt
```

### Implications

| Scenario | Reality Check |
|----------|---------------|
| **"AI will replace developers"** | Wrong. 77% failure rate unacceptable |
| **"AI as pair programmer"** | Possible. 23% success on routine tasks helps |
| **"AI needs human oversight"** | Absolutely. Cannot trust output without verification |
| **"Tests are non-negotiable"** | Critical. Only way to catch 77% failure rate |
| **"Iteration limits needed"** | Essential. Don't waste time on likely failures |

---

## Integration with Knowledge & Vibes

### What SWE-Bench Pro Validates

| K&V Design Choice | SWE-Bench Pro Evidence |
|-------------------|------------------------|
| **P14 Human verification gate** | 77% failure rate demands human review |
| **Conservative expectations** | 23% success → plan for failures |
| **Strong test requirements** | Only way to detect failures |
| **Iteration caps (3 attempts)** | Don't waste time; 77% won't succeed anyway |
| **ADaPT decomposition** | Break down complex tasks for better odds |
| **Phase-based workflow** | Smaller validated increments reduce risk |

### Adjusted Success Metrics

```
┌─────────────────────────────────────────────────────────────────┐
│          REALISTIC SUCCESS EXPECTATIONS (K&V)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase Success Rates (Conservative Estimates):                   │
│                                                                  │
│  Simple Beads:                                                   │
│      First attempt:  ~40% (easier than SWE-Bench Pro avg)        │
│      After 3 tries:  ~60%                                        │
│      After ADaPT:    ~80%                                        │
│                                                                  │
│  Medium Beads:                                                   │
│      First attempt:  ~25% (similar to SWE-Bench Pro)             │
│      After 3 tries:  ~45%                                        │
│      After ADaPT:    ~65%                                        │
│                                                                  │
│  Complex Beads:                                                  │
│      First attempt:  ~10% (harder than SWE-Bench Pro)            │
│      After 3 tries:  ~25%                                        │
│      After ADaPT:    ~50%                                        │
│                                                                  │
│  Key Insight: Even with ADaPT, expect ~50% complex task failure  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Failure Mode Analysis

### Why 77% Fail

SWE-Bench Pro analysis of failures:

| Failure Type | Frequency | Description |
|--------------|-----------|-------------|
| **Context retrieval** | 28% | Can't find relevant code in large codebase |
| **Requirement misunderstanding** | 22% | Misinterprets what needs to be done |
| **Breaking changes** | 18% | Fix works but breaks other functionality |
| **Edge case failures** | 15% | Handles main case, misses edge cases |
| **Bug introduction** | 12% | Introduces new bugs while fixing old ones |
| **Test failures** | 5% | Can't make tests pass |

### Mitigation Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│        HOW K&V ADDRESSES SWE-BENCH PRO FAILURE MODES             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Context Retrieval (28% of failures):                            │
│      → bv --robot-summary (codebase overview)                    │
│      → Explicit file paths in bead context                       │
│      → Phase-based scope limits                                  │
│                                                                  │
│  Requirement Misunderstanding (22%):                             │
│      → Clear AC (Acceptance Criteria) in each bead               │
│      → Calibration discussions before work                       │
│      → Test-first approach (tests = spec)                        │
│                                                                  │
│  Breaking Changes (18%):                                         │
│      → Full test suite must pass                                 │
│      → Integration tests required                                │
│      → Phase-end calibration catches breakage                    │
│                                                                  │
│  Edge Cases (15%):                                               │
│      → Comprehensive AC with edge cases listed                   │
│      → Code-form planning (explicit branches)                    │
│      → Adversarial testing (DebateCoder-style)                   │
│                                                                  │
│  Bug Introduction (12%):                                         │
│      → Iteration limits (stop at 3, don't compound)              │
│      → Minimal change principle (D4C when appropriate)           │
│      → Git history for rollback                                  │
│                                                                  │
│  Test Failures (5%):                                             │
│      → ADaPT decomposition when tests fail                       │
│      → RepairAgent-style structured debugging                    │
│      → RLEF-inspired error citation requirements                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Quote

> "The gap between benchmark performance and real-world performance is larger than the field anticipated. Realistic evaluation reveals fundamental limitations in current approaches."

This quote encapsulates why K&V's conservative, test-heavy, human-verified approach is necessary.

---

## Practical Implications

### For Planning

```markdown
## Project Plan Reality Check

Estimated beads: 50
Complexity breakdown:
  - Simple: 20 beads × 60% success = 12 succeed, 8 need rework
  - Medium: 20 beads × 45% success = 9 succeed, 11 need rework
  - Complex: 10 beads × 25% success = 2.5 succeed, 7.5 need rework

Expected outcomes:
  ✓ First pass success: 23.5 beads (~47%)
  ✗ Need rework: 26.5 beads (~53%)

Timeline adjustment:
  Naive estimate: 50 beads × 2 hours = 100 hours
  Realistic estimate:
    - 23.5 succeed first pass: 47 hours
    - 26.5 need rework (avg 2 extra attempts): 106 hours
    - Total: 153 hours (+53% vs. naive)

Conclusion: Budget 50-100% more time than naive estimates suggest.
```

### For North Star Expectations

| North Star | Acceptable Failure Rate | SWE-Bench Pro Alignment |
|------------|------------------------|------------------------|
| **Prototype** | 50% OK | Better than SWE-Bench Pro (low complexity) |
| **Team Internal** | 20% OK | Needs some rework, acceptable |
| **Production** | 5% OK | Heavy human verification needed |
| **Enterprise** | 1% OK | Extensive review + testing required |
| **Regulatory** | 0.1% OK | AI as aid only, human-driven |

---

## Key Takeaways

1. **Harder benchmarks reveal more:** SWE-Bench Pro exposed gaps simpler benchmarks missed
2. **Verification scales with complexity:** Complex tasks need more scrutiny than simple ones
3. **Human review essential:** Cannot trust AI output without verification
4. **Tests are critical:** Only reliable way to catch failures
5. **Iteration limits justified:** Fresh context often beats repeated retries
6. **Decomposition helps:** Breaking complex tasks into smaller beads improves success
7. **K&V design validated:** Test-heavy, human-verified, structured approach is correct

---

## See Also

- `047-humaneval-pro.md` — Self-invocation challenges across phases
- `060-debugging-decay-index.md` — Why repeated attempts fail
- `038-adapt.md` — Decomposition for hard tasks
- `041-debatecoder.md` — Test-based validation
- `065-confucius-code-agent.md` — Scale challenges for AI agents
