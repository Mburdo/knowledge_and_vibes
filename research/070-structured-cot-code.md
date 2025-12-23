---
title: Structured Chain-of-Thought for Code Generation
tags: [chain-of-thought, structured-reasoning, control-flow, planning, prompting, code-generation]
protocols: [P3, P8]
---

# Structured Chain-of-Thought for Code Generation

**Paper:** Structured Chain-of-Thought Prompting for Code Generation
**URL:** https://arxiv.org/abs/2305.06599 / https://dl.acm.org/doi/10.1145/3690635
**Date:** 2023 (published ACM 2025)

---

## Summary

Research proposing **Structured CoT (SCoT)** prompting, which adapts chain-of-thought reasoning to code generation by using program structures (sequence, branch, loop) as intermediate reasoning steps.

**Key insight:** Standard CoT uses natural language reasoning, but code has inherent structure. SCoT uses that structure.

---

## The CoT Problem for Code

### Standard CoT Limitations

```markdown
## Standard CoT for Code

Prompt: "Write a function to find the maximum in a list"

CoT Reasoning (natural language):
1. First, I need to iterate through the list
2. I'll keep track of the current maximum
3. Compare each element to the maximum
4. Return the result

Problem: Natural language reasoning doesn't map cleanly to code structure
```

### SCoT Solution

```markdown
## SCoT for Code

Prompt: "Write a function to find the maximum in a list"

SCoT Reasoning (code structure):
1. SEQUENCE: Initialize max_val = first element
2. LOOP: For each element in list[1:]
3. BRANCH: If element > max_val
4.   SEQUENCE: max_val = element
5. SEQUENCE: Return max_val

Benefit: Reasoning maps directly to code structure
```

---

## SCoT Architecture

### Three Core Structures

```
Program Structures in SCoT:

1. SEQUENCE
   ├── Statement A
   ├── Statement B
   └── Statement C
   (Executed in order)

2. BRANCH
   ├── IF condition
   │   └── Then block
   └── ELSE
       └── Else block
   (Conditional execution)

3. LOOP
   ├── FOR/WHILE condition
   └── Loop body
   (Repeated execution)
```

### SCoT Prompt Format

```python
"""
Problem: {problem_description}

Plan using program structures:
1. SEQUENCE: {what to do first}
2. LOOP over {what}: {condition}
   2.1 BRANCH: IF {condition}
       - SEQUENCE: {then action}
   2.2 ELSE:
       - SEQUENCE: {else action}
3. SEQUENCE: {final action}

Now implement:
```python
def solution(...):
    # Implementation following the structured plan
```
"""
```

---

## Example: Binary Search

### Standard CoT

```
Reasoning:
1. We need to find an element in a sorted array
2. We can use divide and conquer
3. Check the middle, go left or right
4. Repeat until found

(Vague, doesn't capture structure well)
```

### SCoT

```
Structured Plan:
1. SEQUENCE: Initialize left=0, right=len(arr)-1
2. LOOP: WHILE left <= right
   2.1 SEQUENCE: Calculate mid = (left + right) // 2
   2.2 BRANCH: IF arr[mid] == target
       - SEQUENCE: Return mid (found)
   2.3 BRANCH: ELSE IF arr[mid] < target
       - SEQUENCE: left = mid + 1 (search right)
   2.4 ELSE:
       - SEQUENCE: right = mid - 1 (search left)
3. SEQUENCE: Return -1 (not found)
```

**Result:** Direct mapping to implementation.

---

## Performance Results

| Benchmark | Standard CoT | SCoT | Improvement |
|-----------|--------------|------|-------------|
| HumanEval | 67.1% | 76.8% | +9.7% |
| MBPP | 52.3% | 61.4% | +9.1% |
| CodeContests | 8.2% | 12.1% | +3.9% |

### By Problem Type

| Problem Type | Improvement |
|--------------|-------------|
| Loop-heavy | +12% |
| Branch-heavy | +11% |
| Recursive | +8% |
| Simple sequence | +3% |

**Key insight:** Biggest gains on problems requiring complex control flow.

---

## When to Use SCoT

### Best For

| Scenario | Why SCoT Helps |
|----------|----------------|
| Algorithm implementation | Clear loop/branch structure |
| Data processing pipelines | Sequential + conditional |
| State machines | Explicit branching logic |
| Recursive algorithms | Natural tree structure |

### Less Beneficial For

| Scenario | Why |
|----------|-----|
| Simple getters/setters | Trivial structure |
| Configuration code | Mostly declarative |
| API calls | Sequential with no control flow |
| One-liners | No structure to reason about |

---

## Integration with TDD

### SCoT + TDD Workflow

```markdown
## Enhanced Bead Workflow

1. Read bead requirements
2. Write tests (TDD)
3. Create SCoT plan:
   - Identify key structures (loops, branches)
   - Map requirements to structures
   - Validate plan covers all test cases
4. Implement following SCoT plan
5. Run tests
6. If fail: Revise SCoT plan, then code
```

### Example

```markdown
## Bead: Implement user validation

Tests:
- test_valid_email_passes
- test_invalid_email_fails
- test_password_length_check
- test_password_complexity_check

SCoT Plan:
1. SEQUENCE: Define validate_user(email, password)
2. BRANCH: IF not valid_email_format(email)
   - SEQUENCE: Return ValidationError("invalid email")
3. BRANCH: IF len(password) < 8
   - SEQUENCE: Return ValidationError("password too short")
4. BRANCH: IF not has_complexity(password)
   - SEQUENCE: Return ValidationError("password too simple")
5. SEQUENCE: Return ValidationSuccess()

Now implement to pass all tests...
```

---

## Practical Implications

### For Knowledge & Vibes

SCoT applies to bead descriptions:

| Standard Bead | SCoT-Enhanced Bead |
|---------------|-------------------|
| "Implement X feature" | Structured plan with SEQUENCE/BRANCH/LOOP |
| Vague implementation | Clear control flow |
| Interpretation varies | Consistent structure |

### Bead Description Format

```markdown
## Bead: {title}

### Requirements
{requirements}

### Tests
{test code}

### Implementation Plan (SCoT)
1. SEQUENCE: {initialization}
2. LOOP: {iteration logic}
   2.1 BRANCH: {condition handling}
3. SEQUENCE: {finalization}

### Files
{edit locus}
```

---

## Key Takeaways

1. **Code has structure** — Use it for reasoning
2. **SCoT > Standard CoT for code** — 9-10% improvement
3. **Biggest gains on complex control flow** — Loops and branches
4. **Direct mapping to implementation** — Less interpretation needed
5. **Combine with TDD** — Plan should cover all tests

---

## Limitations

- Overhead for simple problems
- Requires understanding of control structures
- May over-constrain creative solutions
- Less applicable to functional/declarative styles

---

## See Also

- `049-codeform-planning.md` — Structured pseudocode plans
- `054-tdd-ai-code-gen.md` — TDD integration
- `002-planning-driven-programming.md` — Plan-then-implement research
