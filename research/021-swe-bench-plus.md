---
title: SWE-Bench+ - Enhanced Benchmark
tags: [benchmark, validation, test-quality, decontamination, leakage, evaluation]
protocols: [P6, P12]
---

# SWE-Bench+: Enhanced Coding Benchmark for LLMs

**Paper:** SWE-Bench+: Enhanced Coding Benchmark for LLMs
**URL:** https://arxiv.org/html/2410.06992v1
**Date:** October 2024
**Venue:** arXiv preprint

---

## Summary

Critical analysis revealing that SWE-bench scores are **inflated by up to 32.7%** due to solution leakage and weak tests. SWE-Bench+ provides a decontaminated, quality-controlled benchmark that eliminates these artifacts.

**Key finding:** When problematic instances are filtered, previously-reported solve rates drop sharply, exposing the gap between apparent capability and actual performance.

---

## The Benchmark Quality Problem

### Three Sources of Inflation

```
┌─────────────────────────────────────────────────────────────────┐
│                  SWE-BENCH QUALITY ISSUES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Issue 1: Solution Leakage (32.7% of "successes")               │
│  ┌────────────────────────────────────────────────┐             │
│  │ GitHub Issue: "User auth fails"                │             │
│  │ Comment: "The fix is to add salt to hash..."   │ ← Solution  │
│  │ Agent: *reads comment, applies fix*            │   in issue! │
│  │ Result: "Success" (but not autonomous solving) │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  Issue 2: Weak Tests (31.1% of "successes")                     │
│  ┌────────────────────────────────────────────────┐             │
│  │ Patch: Changes behavior in unexpected way      │             │
│  │ Test: Only checks happy path                   │             │
│  │ Result: Tests pass but bug not truly fixed     │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  Issue 3: Contamination Risk                                    │
│  ┌────────────────────────────────────────────────┐             │
│  │ Issue created: Jan 2023                        │             │
│  │ Model trained: Apr 2024                        │             │
│  │ Problem: Issue likely in training data         │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Inflation Breakdown

| Issue Type | Percentage | Impact |
|------------|------------|--------|
| **Solution leakage** | 32.7% | Issue comments contain the fix |
| **Weak tests** | 31.1% | Patch passes but doesn't truly fix |
| **Contamination risk** | Unknown | Issues predate model cutoffs |
| **Combined effect** | ~60%+ | Majority of "successes" suspect |

---

## SWE-Bench+ Improvements

### Decontamination Strategy

```
Original SWE-bench:
├── 2,294 instances
├── Issues from 2018-2023
├── 12 repositories
└── No leakage filtering

         ↓ Apply SWE-Bench+ filters

SWE-Bench+ (decontaminated):
├── Filtered instances
├── Post-cutoff issues only
├── Solution-free issue text
└── Stronger test validation

         ↓ Result

Score drops of 20-40%
(revealing true capability)
```

### Quality Criteria

SWE-Bench+ ensures:

1. **Temporal isolation** — Issues created after model knowledge cutoffs
2. **Solution-free** — No fix hints in issue text or comments
3. **Test strength validation** — Tests must discriminate correct vs plausible fixes
4. **Behavioral coverage** — Tests cover edge cases, not just happy paths

---

## Performance Reality Check

### Before vs After Decontamination

```
Reported Score (SWE-bench)
100% ┤
 90% ┤
 80% ┤
 70% ┤       ████████████
 60% ┤       ████████████
 50% ┤       ████████████
 40% ┤       ████████████
 30% ┤       ████████████    ████████
 20% ┤       ████████████    ████████
 10% ┤       ████████████    ████████
  0% ┼───────────────────────────────────
         Apparent      True Capability
         Capability    (SWE-Bench+)
```

### Score Degradation by Model Class

| Model Type | SWE-bench Score | SWE-Bench+ Score | Drop |
|------------|-----------------|------------------|------|
| GPT-4 class | 45-50% | 25-30% | ~40% relative |
| GPT-3.5 class | 25-30% | 12-18% | ~45% relative |
| Open source | 20-25% | 10-15% | ~50% relative |

---

## The Weak Test Problem

### Why Tests Fail to Discriminate

```python
# Example: Weak Test Suite

# ORIGINAL BUG: Division by zero in edge case
def calculate_average(values):
    return sum(values) / len(values)  # Bug: len(values) can be 0

# TEST SUITE (weak)
def test_average():
    assert calculate_average([1, 2, 3]) == 2.0  # Only tests happy path

# AGENT PATCH (plausible but wrong)
def calculate_average(values):
    if not values:
        return 0  # Wrong: should return None or raise
    return sum(values) / len(values)

# Result: Tests pass, but fix is semantically incorrect
```

### Test Quality Metrics

| Metric | Weak Tests | Strong Tests |
|--------|------------|--------------|
| Edge case coverage | < 20% | > 80% |
| Discriminating power | Low | High |
| False positive rate | 30%+ | < 5% |
| Behavioral completeness | Partial | Comprehensive |

---

## Practical Implications

### The Optimization Paradox

```
┌─────────────────────────────────────────────────────────────────┐
│            YOU CAN'T OPTIMIZE A BROKEN METRIC                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Weak benchmark → False signal → Bad optimization               │
│                                                                  │
│  Example:                                                        │
│  1. Agent scores 45% on SWE-bench                                │
│  2. You optimize workflow → 50% on SWE-bench                     │
│  3. Test on SWE-Bench+ → 23% (no real improvement)               │
│                                                                  │
│  Root cause: Optimized for leakage, not capability               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### What This Means for Agent Development

1. **Benchmark hygiene matters** — Validate your validation
2. **Test strength is critical** — Weak tests create false confidence
3. **Temporal isolation required** — Post-cutoff data only
4. **Multiple validation layers** — Don't rely on tests alone

---

## Integration with K&V Workflow

### Calibration Implications

SWE-Bench+ validates core K&V principles:

| K&V Principle | SWE-Bench+ Evidence |
|---------------|---------------------|
| Evidence-based claims | 63% of "successes" were false positives |
| Test-driven validation | Weak tests mislead agents |
| Discriminating tests | Must falsify wrong approaches |
| Calibration as adjudication | Tests decide, not rhetoric |

### Test Quality Protocol

```markdown
## Before Accepting a Fix

1. **Test completeness check:**
   - Does it cover edge cases?
   - Does it test error conditions?
   - Does it verify behavioral preservation?

2. **Discriminating power check:**
   - Would this test catch plausible-but-wrong fixes?
   - Can we write a variant that breaks our fix?
   - Are there known failure modes not covered?

3. **Leakage check:**
   - Did we derive the fix from tests/comments?
   - Could we solve this without seeing the solution?
   - Is this autonomous solving or pattern matching?
```

### ADaPT Integration

```
Task Bead: "Fix authentication bug"
    │
    ├── Attempt 1: Generate patch
    ├── Attempt 2: Refine patch
    ├── Attempt 3: Alternative approach
    │
    └── CALIBRATION (SWE-Bench+ mindset):
            │
            ├── Are tests strong enough?
            ├── Did we leak solution from comments?
            ├── Does patch truly fix root cause?
            │
            └── Evidence Required:
                ├── Discriminating test suite
                ├── Edge case coverage
                └── Behavioral proof (not just "tests pass")
```

---

## Critical Caveats

### What SWE-Bench+ Doesn't Solve

1. **Harder ≠ Better**
   - Some "hard" instances may be underspecified
   - Difficulty can come from poor issue descriptions
   - Not all hard problems are meaningful

2. **Still Benchmark-Shaped**
   - Python-heavy ecosystem
   - Single-repository scope
   - Predetermined test suites
   - Real-world work is messier

3. **Decontamination Arms Race**
   - Models will eventually train on SWE-Bench+
   - Continuous benchmark updates required
   - See also: `028-swe-bench-live.md`

---

## Research Methodology

### How They Identified Leakage

```python
# Automated analysis pipeline

1. Parse GitHub issue + comments
2. Extract code snippets from text
3. Compare with ground-truth patch
4. Flag if >70% overlap

Results:
- 32.7% contained solution in comments
- Most leakage in "helpful" maintainer replies
- Some issues literally paste the fix
```

### Test Weakness Detection

```python
# Mutation testing approach

1. Generate plausible-but-wrong patches
2. Run against test suite
3. If tests pass → test suite is weak

Results:
- 31.1% of test suites pass wrong patches
- Happy-path bias dominates
- Error handling rarely tested
```

---

## Key Metrics to Track

When evaluating agents, track:

```json
{
  "benchmark": "swe-bench-plus",
  "score": 0.28,
  "decontamination_verified": true,
  "test_strength_validated": true,
  "solution_leakage_filtered": true,
  "temporal_isolation": "post-cutoff-only",
  "notes": "True capability metric, not inflated"
}
```

---

## Key Takeaways

1. **Benchmark quality determines optimization quality** — Broken metrics lead to broken agents
2. **63% of SWE-bench successes were suspect** — Solution leakage + weak tests
3. **Test strength is a first-class concern** — Must discriminate correct from plausible
4. **Decontamination is mandatory** — Pre-cutoff data inflates scores
5. **Validate your validation** — Don't trust tests blindly

---

## See Also

- `060-debugging-decay-index.md` — Why iteration limits matter for repair
- `022-chatrepair.md` — Conversation-driven repair with test feedback
- `028-swe-bench-live.md` — Continuously updated benchmark to prevent contamination
- `029-swe-rebench.md` — Standardized decontaminated evaluation
- `026-flames.md` — Test-guided semantic search for repair
