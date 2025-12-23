---
title: Planning-Driven Programming (LPW)
tags: [planning, debugging, verification, code-generation, workflow]
protocols: [P3, P8]
---

# Planning-Driven Programming (LPW)

**Paper:** Planning-Driven Programming: A Large Language Model Programming Workflow
**URL:** https://arxiv.org/abs/2411.14503
**Date:** November 2024

---

## Summary

A two-phase workflow where the LLM first generates a step-by-step plan, verifies it against test cases, then generates code. The verified plan is also used to guide debugging when tests fail.

**Key finding:** The debugging benefit (9.1%) exceeds the verification benefit (3.0%).

---

## The LPW Workflow

### Four-Phase Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    LPW WORKFLOW                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: PLAN GENERATION                                        │
│  ├── LLM creates numbered steps                                  │
│  └── Steps describe algorithm, not code                          │
│                                                                  │
│  Phase 2: PLAN VERIFICATION                                      │
│  ├── LLM traces through plan with test inputs                   │
│  ├── Compares expected vs actual outputs                         │
│  └── If mismatch: revise plan                                   │
│                                                                  │
│  Phase 3: CODE GENERATION                                        │
│  ├── Generate code following verified plan                       │
│  └── Plan serves as implementation guide                         │
│                                                                  │
│  Phase 4: PLAN-GUIDED DEBUGGING                                  │
│  ├── If tests fail: compare code to plan                         │
│  ├── Find where code diverges from plan                          │
│  └── Fix the divergence                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Performance Results

### GPT-3.5 Results

| Benchmark | LPW | Baseline | Improvement |
|-----------|-----|----------|-------------|
| HumanEval | 89.0% | 82.9% | +6.1% |
| MBPP | 76.0% | 72.4% | +3.6% |

### GPT-4o on Harder Benchmarks

| Benchmark | LPW | Baseline | Improvement |
|-----------|-----|----------|-------------|
| CodeContests | 34.7% | 29.3% | +5.4% |
| APPS | 62.6% | 53.2% | +9.4% |

```
Improvement Range:
────────────────────────────────────────────
Easy (HumanEval)      ████████  +6.1%
Medium (MBPP)         ██████    +3.6%
Hard (CodeContests)   ████████  +5.4%
Very Hard (APPS)      ████████████████  +9.4%
────────────────────────────────────────────
```

**Pattern:** Improvement increases with task difficulty.

---

## Ablation: What Matters Most

| Component Removed | Performance Drop |
|-------------------|------------------|
| Plan verification | -3.0% |
| Code refinement loop | -9.1% |

```
Contribution Analysis:
──────────────────────────────────────────────────────
Plan Verification    ██████        3.0%  (initial boost)
Plan-Guided Debug    ██████████████████  9.1%  (big win)
──────────────────────────────────────────────────────
```

### Why Debugging Matters More

The plan becomes most valuable when things go wrong:

```
Without Plan:
─────────────
Test fails → What's wrong?
Guess → Try fix → Often wrong

With Verified Plan:
─────────────
Test fails → Compare code to plan
Find divergence → Fix specific issue
Much higher success rate
```

---

## Verification Accuracy

### The 92.7% Problem

Plan verification accuracy: **92.7%**

This means 7.3% of verifications are wrong. The plan passes verification but contains errors.

| Verification Result | Actual | Consequence |
|---------------------|--------|-------------|
| Pass (correct) | 92.7% | Good |
| Pass (incorrect) | 7.3% | Silent failure |

**Implication:** Don't treat verified plans as ground truth. Still run tests.

---

## Root Cause Analysis

### Half of Failures are Planning Failures

```
Failure Distribution:
────────────────────────────────────────
Planning failures     ████████████████████  ~50%
Implementation bugs   ████████████████████  ~50%
────────────────────────────────────────
```

If the LLM can't create a valid plan, no amount of verification or debugging helps.

---

## Practical Implications

### For Knowledge & Vibes

| Finding | Application |
|---------|-------------|
| Plans help most for debugging | Use phase docs to debug drift |
| 9.1% improvement from debug loop | Compare implementation to plan when stuck |
| Verification only 3% benefit | Plans are guides, not guarantees |
| 92.7% verification accuracy | Always run tests, don't trust verification alone |

### How to Apply This

```markdown
## When Bead Implementation Fails

1. Don't just retry
2. Compare current code to bead description
3. Find where implementation diverged from intent
4. Fix the specific divergence
5. This is more effective than blind iteration
```

### Plan Format That Works

```
Step 1: Parse input to extract X
Step 2: Validate X against constraint Y
Step 3: Transform X using algorithm Z
Step 4: Handle edge case: empty input
Step 5: Return formatted result
```

Plans should describe **what to do**, not **how to code it**.

---

## Key Takeaways

1. **Plan-then-implement works** - Consistent 2-10% improvement
2. **Debugging is the big win** - 9.1% from plan-guided debugging
3. **Verification helps less than expected** - Only 3% initial boost
4. **Don't trust verification alone** - 7.3% false positive rate
5. **Half of failures are planning failures** - Bad plan = bad code

---

## See Also

- `019-plansearch.md` - Alternative planning approaches
- `020-codetree.md` - Tree-structured code generation
- `014-codeplan.md` - Repository-level planning
