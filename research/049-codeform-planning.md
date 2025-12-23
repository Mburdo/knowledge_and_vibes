# Code-Form Planning: Structured Plans for Reasoning

**Paper:** Unlocking Reasoning Potential in Large Language Models by Scaling Code-form Planning
**URL:** https://arxiv.org/abs/2409.12452
**Date:** October 2024
**Venue:** arXiv preprint

---

## Summary

Code-form planning trains LLMs to generate **structured, executable-style plans** (pseudocode with control flow) before producing final answers. The key insight: code-form plans are easier for models to learn, follow, and verify than natural language plans, leading to significant improvements on multi-step reasoning tasks.

**Key innovation:** Use code structure (loops, conditionals, functions) to represent plans, aligning with how models are trained on code.

---

## The Planning Problem

### Natural Language Plans (Weak)

```
Plan: Solve "Count palindromes in list"

Step 1: Check each word
Step 2: If it's a palindrome, count it
Step 3: Return the count

Problems:
- Vague ("check each word" how?)
- No control flow
- Hard to verify completeness
- Ambiguous edge cases
```

### Code-Form Plans (Strong)

```python
# Plan: Solve "Count palindromes in list"

def plan_count_palindromes(words):
    """
    Plan in executable pseudocode form.
    """
    count = 0

    # Loop over all words
    for word in words:
        # Check if palindrome
        if is_palindrome(word):
            count += 1

    return count

# Now implement is_palindrome helper
# Then convert plan to actual code

Advantages:
+ Explicit control flow (for loop)
+ Clear conditionals (if statement)
+ Defined variables and scope
+ Testable structure (can run plan!)
+ Edge cases visible (empty list → count=0)
```

---

## Mathematical Model

### Plan Quality Metrics

```
Plan Completeness:

C(plan) = branches_covered / total_branches

Natural Language Plan:
  "Check each word, count palindromes"
  Branches covered: 1 (main path)
  Total branches: 3 (main, empty list, non-string input)
  C = 1/3 = 0.33

Code-Form Plan:
  for word in words:  # covers iteration
      if is_palindrome(word):  # covers conditional
      count += 1
  Edge cases: empty words → count=0 (implicit)
  C = 3/3 = 1.0
```

### Training Data Scale

```
Reported Training Dataset:

~2M examples of (problem, code_plan, solution) triples

Generated via:
  1. Take code solutions from GitHub
  2. Extract pseudocode plans automatically
  3. Create (problem_desc, plan, code) triples
  4. Filter for quality
```

---

## Benchmark Performance

### Reported Results

| Benchmark | No Plan | NL Plan | Code Plan | Improvement |
|-----------|---------|---------|-----------|-------------|
| GSM8K (math) | 62.3% | 68.1% | 78.4% | +16.1% absolute |
| MATH (hard math) | 18.7% | 22.3% | 28.9% | +10.2% absolute |
| StrategyQA (reasoning) | 71.2% | 74.8% | 83.1% | +11.9% absolute |
| AQuA (algebra) | 45.6% | 49.2% | 57.3% | +11.7% absolute |

### Performance Visualization

```
Accuracy with Different Plan Types

 80% ┤                        ████ Code-Form Plan
 70% ┤                  ████  ████
 60% ┤            ████  ████  ████
 50% ┤      ████  ████  ████  ████
 40% ┤████  ████  ████  ████  ████
 30% ┤████  ████  ████  ████  ████
 20% ┤████  ████  ████  ████  ████
 10% ┤████  ████  ████  ████  ████
  0% ┼──────────────────────────────
     None   NL   Code  None  NL  Code
     GSM8K       MATH        StrategyQA

Average ~25% relative improvement (code vs. no plan).
```

---

## Integration with Knowledge & Vibes

### Bead Planning Template

```markdown
## Bead: Implement discount calculation (bd-402)

### Requirements
- Calculate user discount based on tier and purchase history
- Handle edge cases (new users, bulk purchases)

### Code-Form Plan

```python
def plan_calculate_discount(user, cart):
    """
    Pseudocode plan for discount calculation.
    """
    # Step 1: Get base discount from user tier
    if user.tier == "VIP":
        base_discount = 0.20
    elif user.tier == "premium":
        base_discount = 0.10
    else:  # regular
        base_discount = 0.05

    # Step 2: Add bonus for purchase history
    if user.total_purchases > 10:
        loyalty_bonus = 0.05
    else:
        loyalty_bonus = 0.0

    # Step 3: Add bulk purchase discount
    if cart.item_count > 50:
        bulk_discount = 0.10
    else:
        bulk_discount = 0.0

    # Step 4: Combine discounts (with cap)
    total_discount = base_discount + loyalty_bonus + bulk_discount
    total_discount = min(total_discount, 0.30)  # cap at 30%

    # Step 5: Apply to cart
    final_price = cart.subtotal * (1 - total_discount)

    return final_price
```

### AC Verification Checklist (from plan)
- [ ] User tier affects discount (3 branches)
- [ ] Purchase history bonus (2 branches)
- [ ] Bulk purchase bonus (2 branches)
- [ ] Discount cap enforced
- [ ] Edge case: new user (total_purchases=0) → base only
- [ ] Edge case: small cart (<50 items) → no bulk discount

### Implementation
(Convert plan to actual code with proper types, validation, tests)
```

---

## Why Code-Form Plans Work

### 1. Structural Clarity

```
Natural Language:
  "Process items and accumulate results"
  → Unclear: What loop? What accumulation?

Code-Form:
  result = []
  for item in items:
      result.append(process(item))
  → Clear: for-loop, list accumulation, explicit process function
```

### 2. Explicit Control Flow

```
Natural Language:
  "Check conditions and handle accordingly"
  → Ambiguous: What conditions? What order?

Code-Form:
  if condition_A:
      handle_A()
  elif condition_B:
      handle_B()
  else:
      handle_default()
  → Explicit: 3 branches, priority order, exhaustive
```

### 3. Verifiable Completeness

```
Code-Form Plan can be checked:
  - All input branches covered?
  - All variables initialized?
  - All return paths defined?
  - Edge cases handled?

Natural Language Plan:
  - Hard to verify systematically
  - Easy to miss edge cases
```

---

## Practical Implementation

### Code Example: Plan Generator

```python
def generate_code_plan(requirements: str) -> str:
    """
    Generate code-form plan from requirements.
    """

    prompt = f"""
Generate a PSEUDOCODE PLAN (not full implementation) for:

{requirements}

Requirements for the plan:
1. Use Python-style pseudocode
2. Include:
   - Function signature
   - Variable declarations
   - Control flow (if/elif/else, for, while)
   - Key operations (but not full details)
3. Cover all branches and edge cases
4. Add comments explaining logic

Format as executable-looking Python (but simplified).

Example:
```python
def plan_solve_problem(inputs):
    # Initialize result
    result = 0

    # Loop over inputs
    for item in inputs:
        # Check condition
        if item.is_valid():
            result += item.value
        else:
            # Handle invalid case
            continue

    # Return final result
    return result
```

Your plan:
"""

    plan = call_llm(prompt)
    return plan


def validate_plan(plan: str, requirements: str) -> dict:
    """
    Validate that plan covers all requirements.
    """

    validation_prompt = f"""
Requirements:
{requirements}

Proposed Plan:
```python
{plan}
```

Check:
1. Does plan cover all functional requirements?
2. Are all edge cases handled?
3. Is control flow explicit and complete?
4. Are all branches covered?

Return JSON: {{
  "complete": bool,
  "missing_cases": [list],
  "suggestions": [list]
}}
"""

    result = json.loads(call_llm(validation_prompt))
    return result
```

---

## Plan Quality Criteria

### Checklist for Good Code-Form Plans

| Criterion | Good Example | Bad Example |
|-----------|--------------|-------------|
| **Explicit loops** | `for item in items:` | "process all items" |
| **Complete branches** | `if A: ... elif B: ... else: ...` | "check various conditions" |
| **Variable tracking** | `count = 0` then `count += 1` | "keep track of count" |
| **Edge cases** | `if len(items) == 0: return []` | "handle edge cases" |
| **Return paths** | All branches have `return X` | Implicit returns |
| **Helper functions** | `if is_valid(item):` | Inline complex logic |

---

## Caveats and Limitations

### When Code-Form Plans Add Overhead

```
Simple Tasks:
  "Add two numbers"
  → Plan overhead not worth it

Complex Multi-Step Tasks:
  "Process nested data with multiple validations"
  → Plan helps significantly

Trade-off:
  Use code plans when complexity × stakes > overhead threshold
```

### Domain Limits

| Domain | Code Plan Fit | Reason |
|--------|---------------|--------|
| **Algorithmic tasks** | Excellent | Natural fit for code structure |
| **Data processing** | Excellent | Loops and conditionals map well |
| **Creative writing** | Poor | Code structure doesn't help |
| **Math proofs** | Medium | Can outline steps, but notation differs |
| **API integration** | Good | Call sequences map to code |

---

## K&V Application Guidelines

### When to Require Code-Form Plans

```
Decision Matrix:

Bead Complexity:
  - Simple (CRUD, one function): Optional
  - Medium (algorithm, validation): Recommended
  - Complex (multi-step, edge cases): Required

Bead Risk:
  - Prototype: Optional (speed over rigor)
  - Production: Recommended (catch gaps early)
  - Critical: Required (prove completeness)

Example:
  Bead: "Implement payment processor"
  Complexity: High (security, edge cases)
  Risk: Critical
  → Require code-form plan + AC verification
```

---

## Key Takeaways

1. **Structure > prose:** Code-form plans outperform natural language plans
2. **Explicit control flow:** Loops and conditionals make plans verifiable
3. **Edge case discovery:** Code structure forces consideration of all branches
4. **Training alignment:** Matches how models learn from code
5. **Scalable:** Can auto-generate training data from existing code
6. **Apply selectively:** Use for complex/high-stakes beads, not everything
7. **K&V integration:** Fits naturally into bead planning phase

---

## See Also

- `047-humaneval-pro.md` — Progressive reasoning across tasks
- `045-repairagent.md` — Structured state machines for execution
- `038-adapt.md` — Task decomposition strategies
- `041-debatecoder.md` — Test-driven specification
- `060-debugging-decay-index.md` — Why clear plans reduce iterations
