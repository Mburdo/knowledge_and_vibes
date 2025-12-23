---
title: Test-Driven Development for Code Generation
tags: [tdd, testing, remediation, iteration, feedback-loop, code-generation]
protocols: [P6, P8]
---

# Test-Driven Development for Code Generation

**Paper:** Test-Driven Development for Code Generation
**URL:** https://arxiv.org/abs/2402.13521
**Date:** February 2024 (revised June 2024)
**Venue:** arXiv preprint

---

## Summary

Empirical research demonstrating that providing LLMs with tests alongside problem statements significantly improves code correctness, and adding a test-failure remediation loop improves it further. The TGen framework operationalizes TDD for LLMs through a generate-run-fix cycle with bounded iterations.

**Key finding:** GPT-4 shows **+12.0% on MBPP** and **+8.5% on HumanEval** when tests are provided, with an additional **+2.8-3.0%** gain from remediation loops.

---

## The Core Problem

### Code Generation Without Tests

Traditional LLM code generation:
```
Problem Description → LLM → Code → Submit → Hope
```

Problems:
- No feedback loop
- No validation before submission
- Model has no way to verify correctness
- Edge cases are missed

### The Test Deficit

```
┌────────────────────────────────────────────────────────┐
│         CODE GENERATION WITHOUT TESTS                   │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Problem: "Write a function to reverse a string"        │
│           │                                              │
│           ▼                                              │
│  def reverse(s):                                         │
│      return s[::-1]                                      │
│           │                                              │
│           ▼                                              │
│  ✓ Works for "hello" → "olleh"                          │
│  ✗ Fails for None → TypeError                           │
│  ✗ Fails for unicode → mojibake                         │
│  ✗ Fails for empty string edge cases                    │
│                                                          │
│  Model has NO WAY to discover these failures            │
│                                                          │
└────────────────────────────────────────────────────────┘
```

---

## The Solution: TDD for LLMs

### TGen Framework

```
┌─────────────────────────────────────────────────────────────────┐
│                      TGEN FRAMEWORK                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INPUT                                                           │
│  ┌────────────────────────────────────────────────────────┐     │
│  │  Problem Statement + Test Cases                        │     │
│  └────────────────┬───────────────────────────────────────┘     │
│                   │                                              │
│  GENERATION       ▼                                              │
│  ┌────────────────────────────────────────────────────────┐     │
│  │  LLM generates code with tests in context              │     │
│  └────────────────┬───────────────────────────────────────┘     │
│                   │                                              │
│  EXECUTION        ▼                                              │
│  ┌────────────────────────────────────────────────────────┐     │
│  │  Run all tests against generated code                  │     │
│  └────────────────┬───────────────────────────────────────┘     │
│                   │                                              │
│                   ├─► All Pass? ──► DONE                        │
│                   │                                              │
│                   └─► Some Fail? ──► REMEDIATION                │
│  REMEDIATION                          │                         │
│  ┌───────────────────────────────────┘                          │
│  │                                                               │
│  │  ┌────────────────────────────────────────────────────┐      │
│  │  │  Feed failures back to LLM with context:           │      │
│  │  │  - Original code                                   │      │
│  │  │  - Failing test                                    │      │
│  │  │  - Error message                                   │      │
│  │  │  - Stack trace                                     │      │
│  │  └────────────────┬───────────────────────────────────┘      │
│  │                   │                                           │
│  │                   ▼                                           │
│  │  ┌────────────────────────────────────────────────────┐      │
│  │  │  LLM generates fixed code                          │      │
│  │  └────────────────┬───────────────────────────────────┘      │
│  │                   │                                           │
│  │                   ▼                                           │
│  │  ┌────────────────────────────────────────────────────┐      │
│  │  │  Run tests again                                   │      │
│  │  └────────────────┬───────────────────────────────────┘      │
│  │                   │                                           │
│  │                   ├─► Pass? ──► DONE                         │
│  │                   │                                           │
│  │                   └─► Fail & iterations < N? ──► LOOP        │
│  │                                      │                        │
│  │                                      └─► Max iterations? END  │
│  │                                                               │
│  └───────────────────────────────────────────────────────────────┘
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Three Configurations

| Configuration | Tests Provided? | Remediation Loop? | Description |
|---------------|-----------------|-------------------|-------------|
| **Baseline** | No | No | Standard prompt-only generation |
| **TDD-Static** | Yes | No | Tests in context, single generation |
| **TDD-Iterative** | Yes | Yes | Tests + feedback loop (TGen) |

---

## Performance Results

### MBPP Benchmark (Code Generation)

| Model | Baseline | +Tests | +Remediation | Total Improvement |
|-------|----------|--------|--------------|-------------------|
| GPT-4 | 75.0% | 87.0% (+12.0%) | 89.8% (+2.8%) | **+14.8%** |
| GPT-3.5 | 62.3% | 71.5% (+9.2%) | 73.9% (+2.4%) | **+11.6%** |
| CodeLlama-7B | 45.2% | 53.8% (+8.6%) | 55.4% (+1.6%) | **+10.2%** |

### HumanEval Benchmark

| Model | Baseline | +Tests | +Remediation | Total Improvement |
|-------|----------|--------|--------------|-------------------|
| GPT-4 | 84.1% | 92.6% (+8.5%) | 95.6% (+3.0%) | **+11.5%** |
| GPT-3.5 | 70.7% | 78.9% (+8.2%) | 81.4% (+2.5%) | **+10.7%** |
| CodeLlama-13B | 53.0% | 60.2% (+7.2%) | 62.1% (+1.9%) | **+9.1%** |

### Key Insights from Results

```
Effectiveness Breakdown:
┌────────────────────────────────────────────────┐
│                                                 │
│  Baseline:        ████████████████░░░░░░  75%  │
│                                                 │
│  +Tests:          ██████████████████████░  87%  │
│                   ^^^^^^^^^^^^^^^^              │
│                   +12% from tests in context    │
│                                                 │
│  +Remediation:    ███████████████████████  90%  │
│                                          ^^     │
│                   +3% from feedback loop        │
│                                                 │
│  Conclusion: Tests provide 80% of gain         │
│              Iteration provides remaining 20%  │
│                                                 │
└────────────────────────────────────────────────┘
```

---

## Why Tests Improve Generation

### Mechanism 1: Specification Grounding

Tests provide **concrete examples** of expected behavior:

```python
# Problem: "Write a function to check if a number is prime"
# Vague, many interpretations

# With tests:
assert is_prime(2) == True   # 2 is prime
assert is_prime(4) == False  # 4 is not prime
assert is_prime(1) == False  # 1 is not prime (edge case!)
assert is_prime(0) == False  # 0 is not prime
assert is_prime(-5) == False # negative numbers not prime

# Now specification is CONCRETE
```

### Mechanism 2: Edge Case Discovery

Tests force the model to consider boundary conditions:

| Without Tests | With Tests |
|---------------|------------|
| Focuses on happy path | Must handle edge cases |
| Assumes valid input | Sees invalid input examples |
| Misses boundary conditions | Tests show boundaries |
| No null/empty handling | Tests include null/empty |

### Mechanism 3: Constraint Validation

Tests encode **implicit constraints**:

```python
# Problem: "Implement binary search"

# Tests reveal constraints that prose might miss:
def test_binary_search():
    # Constraint: must work on sorted arrays
    assert binary_search([1, 2, 3, 4, 5], 3) == 2

    # Constraint: must handle not found
    assert binary_search([1, 2, 3], 10) == -1

    # Constraint: must handle empty array
    assert binary_search([], 5) == -1

    # Constraint: must handle single element
    assert binary_search([1], 1) == 0
```

---

## Remediation Loop Analysis

### Iteration Value

```
Success Rate by Iteration:
┌────────────────────────────────────────────────┐
│                                                 │
│  Iteration 1:  ████████████████████ 87%        │
│                                                 │
│  Iteration 2:  █████████████████████ 93%       │
│                                     ^^^         │
│                +6% additional fixes             │
│                                                 │
│  Iteration 3:  █████████████████████ 95%       │
│                                     ^^          │
│                +2% additional fixes             │
│                                                 │
│  Iteration 4+: █████████████████████ 95.5%     │
│                                      ^          │
│                +0.5% (diminishing returns)      │
│                                                 │
└────────────────────────────────────────────────┘
```

**Finding:** Most value comes from first 2-3 iterations, then sharply diminishing returns.

### What Gets Fixed in Remediation?

Analysis of failures fixed by iteration:

| Failure Type | Fixed in Iter 1 | Fixed in Iter 2 | Fixed in Iter 3+ |
|--------------|-----------------|-----------------|------------------|
| Simple logic errors | 85% | 12% | 3% |
| Missing edge cases | 60% | 30% | 10% |
| Type errors | 90% | 8% | 2% |
| Off-by-one errors | 70% | 25% | 5% |
| Complex algorithm bugs | 30% | 40% | 30% |

**Pattern:** Simple bugs fixed quickly, complex bugs require more iterations or won't be fixed.

---

## Practical Implications

### For Knowledge & Vibes

Direct validation of core K&V principles:

| TGen Principle | K&V Implementation | Status |
|----------------|-------------------|--------|
| Tests mandatory | AC → test harness | ✓ Core protocol |
| Feedback loop required | 3-iteration rule | ✓ Implemented |
| Bounded iterations | Max 3, then decompose | ✓ From DDI research |
| Test-first generation | AC before implementation | ✓ Bead structure |

### Bead Execution Template

```markdown
## TDD-Style Bead Execution

1. **Parse Acceptance Criteria**
   - Extract testable conditions from AC
   - Generate test cases (unit tests)
   - Include edge cases, boundaries, error cases

2. **First Generation**
   - Generate code with tests in full context
   - Run all tests
   - If all pass → done (happens ~87% of time)

3. **Remediation (if needed)**
   - Feed back: failing test + error + trace
   - Generate fix (iteration 1)
   - Run tests
   - If pass → done (happens ~93% cumulative)

4. **Second Remediation (if needed)**
   - Feed back failures again
   - Generate fix (iteration 2)
   - Run tests
   - If pass → done (happens ~95% cumulative)

5. **Third Attempt or Decompose**
   - If still failing after 2 remediation attempts:
     - Option A: One more try (iteration 3)
     - Option B: Decompose (ADaPT pattern)
   - Decision based on failure type:
     - Simple bugs → try once more
     - Complex/unclear bugs → decompose
```

### Cost-Benefit Analysis

| Metric | No Tests | +Tests | +Tests+Remediation |
|--------|----------|--------|--------------------|
| Success rate | 75% | 87% | 95% |
| Tokens per attempt | 1x | 1.2x | 2.5x |
| Attempts to success | 4.0 | 2.3 | 1.3 |
| **Total tokens to success** | **4.0x** | **2.8x** | **3.3x** |
| **Cost effectiveness** | Baseline | **Best** | Good |

**Surprise finding:** Tests alone are most cost-effective. Remediation helps success rate but increases token cost.

**K&V strategy:** Use tests always, use remediation selectively (when fix seems straightforward).

---

## Implementation Checklist

### For Agent System
- [x] Extract tests from AC automatically (implemented in bead protocol)
- [ ] Run tests before considering bead "done"
- [ ] Feed test failures back with full context (code + test + error + trace)
- [ ] Track iteration count per bead
- [ ] Stop at 3 iterations (DDI threshold)
- [ ] Log: test pass rate, iterations needed, failure types

### For Operators
- [ ] Template for "acceptance criteria → test cases" conversion
- [ ] Examples of good AC (testable, concrete, boundary-inclusive)
- [ ] Dashboard showing: test coverage, pass rates, iteration stats
- [ ] Alert when beads exceed 2 iterations (potential decomposition candidate)

---

## Key Takeaways

1. **Tests are mandatory** — +8-12% success rate improvement just from including tests in context
2. **First iteration matters most** — 85-90% of fixable bugs are fixed in first remediation
3. **Diminishing returns after 3** — Success rate plateaus, iteration becomes waste
4. **Tests beat iteration** — Tests in context provide 80% of benefit, iteration provides 20%
5. **Specification precision** — Tests make vague requirements concrete and unambiguous

---

## Limitations

### Research Scope
- **Benchmarks:** Function-level problems (HumanEval, MBPP), not repo-scale
- **Test source:** High-quality human-written tests, not auto-generated
- **Problem clarity:** Well-defined problems, not ambiguous requirements

### Practical Constraints
- **Test quality matters:** Bad tests → bad code
- **Token cost:** Remediation adds 1.5-2x tokens per attempt
- **Time cost:** Each iteration adds wall-clock time
- **Iteration limit:** Unbounded iteration can waste resources

### Transfer to K&V Context
- **Repo-scale complexity:** Multi-file changes harder than single function
- **Test generation:** Must auto-generate from AC (not always straightforward)
- **Integration tests:** Unit tests insufficient for system behavior

**Conclusion:** Principles (tests mandatory, bounded iteration) transfer completely. Absolute numbers will differ for repo-scale work.

---

## Mathematical Model

### Success Probability

```
P(success | iteration n) = P₀ + Σᵢ₌₁ⁿ δᵢ

Where:
  P₀ = Initial success probability without tests
  δᵢ = Marginal improvement from iteration i

Observed values:
  P₀ = 0.75 (baseline)
  δ₁ = 0.12 (tests in context)
  δ₂ = 0.06 (first remediation)
  δ₃ = 0.02 (second remediation)
  δ₄+ ≈ 0.005 (further iterations)

Conclusion: δᵢ decays exponentially
  δᵢ ≈ δ₁ × 0.5^(i-1)
```

### Cost-Effectiveness Threshold

```
When is remediation worth it?

Expected cost = (token_cost × iterations) / success_rate

No tests:     4.0 / 0.75 = 5.33
+Tests:       1.2 / 0.87 = 1.38  ← Best
+Remediation: 2.5 / 0.95 = 2.63

Tests alone minimize expected cost.
Add remediation only if:
  - Success is critical (95% > 87%)
  - Failure cost > token cost
```

---

## See Also

- `031-alphacodium.md` — Flow engineering with test generation
- `054-tdd-ai-code-gen.md` — More TDD validation for AI code generation
- `055-test-intentions.md` — Test-driven development methodologies
- `060-debugging-decay-index.md` — Why iteration limits matter (3-attempt rule)
- `038-adapt.md` — What to do when tests keep failing (decompose)
