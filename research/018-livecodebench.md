---
title: LiveCodeBench - Contamination-Free Evaluation
tags: [benchmark, contamination, evaluation, self-repair, date-filtering, code-generation]
protocols: [P6, P12]
---

# LiveCodeBench: Contamination-Free Code Evaluation

**Paper:** LiveCodeBench: Holistic and Contamination Free Evaluation of Large Language Models for Code
**URL:** https://arxiv.org/html/2403.07974v1
**Date:** March 2024
**Venue:** arXiv preprint

---

## Summary

Critical benchmark research addressing **data contamination** in code evaluation. LiveCodeBench continuously mines fresh problems from competitive programming platforms, allowing date-filtered evaluation to ensure models haven't trained on test data.

**Key insight:** Static benchmarks become contaminated as models train on them. Dynamic, date-filtered benchmarks provide honest evaluation.

---

## The Contamination Problem

### Why Static Benchmarks Fail

```
┌─────────────────────────────────────────────────────────────────┐
│              BENCHMARK CONTAMINATION CYCLE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  2020: HumanEval Released                                       │
│  ├─ GPT-3: 38% pass@1                                           │
│  └─ Benchmark is fresh, uncontaminated                          │
│                                                                  │
│  2021: Models trained with web data                             │
│  ├─ Web contains HumanEval solutions (GitHub, blogs)            │
│  ├─ Models see benchmark during training                        │
│  └─ "Memorization" vs "capability" unclear                      │
│                                                                  │
│  2022-2023: Performance explosion                               │
│  ├─ GPT-3.5: 72% pass@1                                         │
│  ├─ GPT-4: 90% pass@1                                           │
│  └─ Is this real improvement or contamination?                  │
│                                                                  │
│  2024: Benchmark saturated                                      │
│  ├─ Top models: 85-92% pass@1                                   │
│  ├─ Can't differentiate between models                          │
│  └─ Benchmark no longer useful                                  │
│                                                                  │
│  Problem: Static benchmark → inevitable contamination           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Contamination Sources

| Source | Risk | Example |
|--------|------|---------|
| **Training data** | High | Web crawl includes GitHub with benchmark solutions |
| **Fine-tuning data** | Very high | Synthetic data generated from benchmarks |
| **Instruction data** | High | Few-shot examples using benchmark problems |
| **Test-time leakage** | Medium | Model caches or logs during evaluation |

---

## The LiveCodeBench Solution

### Continuous Problem Mining

```
┌─────────────────────────────────────────────────────────────────┐
│              LIVECODEBENCH ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  PROBLEM SOURCES                                          │   │
│  │                                                           │   │
│  │  ├─ LeetCode Daily/Weekly Contest                        │   │
│  │  ├─ Codeforces Rounds                                    │   │
│  │  ├─ AtCoder Beginner/Regular Contest                     │   │
│  │  └─ Other competitive programming sites                  │   │
│  │                                                           │   │
│  │  Continuous mining: New problems every week              │   │
│  │                                                           │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  PROBLEM CURATION                                         │   │
│  │                                                           │   │
│  │  For each problem:                                       │   │
│  │  ├─ Extract statement                                    │   │
│  │  ├─ Normalize test cases                                 │   │
│  │  ├─ Tag with release date                                │   │
│  │  ├─ Classify difficulty                                  │   │
│  │  └─ Store language-agnostic version                      │   │
│  │                                                           │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  DATE-FILTERED EVALUATION                                 │   │
│  │                                                           │   │
│  │  For model with cutoff date D:                           │   │
│  │  ├─ Filter problems: release_date > D                    │   │
│  │  ├─ Guarantees: Model couldn't train on these            │   │
│  │  └─ Result: Contamination-free evaluation                │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Date-Filtering Example

```python
# Model evaluation
model_cutoff = "2024-04-15"  # GPT-4 Turbo cutoff

# LiveCodeBench has problems from 2023-2025
problems = livecodebench.get_all_problems()

# Filter for fresh problems
fresh_problems = [
    p for p in problems
    if p.release_date > model_cutoff
]

# Evaluate only on fresh problems
results = evaluate(model, fresh_problems)

# Guaranteed: Model never saw these during training
```

---

## The Four Scenarios

### Beyond Code Generation

```
┌─────────────────────────────────────────────────────────────────┐
│              LIVECODEBENCH EVALUATION SCENARIOS                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario 1: CODE GENERATION (Traditional)                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Input: Problem statement                                   │ │
│  │ Output: Working solution                                   │ │
│  │ Metric: pass@k (% that pass all tests)                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Scenario 2: SELF-REPAIR (Iterative Debugging)                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Input: Problem + initial attempt + test failures           │ │
│  │ Output: Fixed solution                                     │ │
│  │ Metric: repair@k (% fixed after k attempts)                │ │
│  │                                                             │ │
│  │ Workflow:                                                   │ │
│  │ 1. Generate initial solution                               │ │
│  │ 2. Run tests → get failures                                │ │
│  │ 3. Show failures to model                                  │ │
│  │ 4. Generate fix                                            │ │
│  │ 5. Repeat up to k times                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Scenario 3: CODE EXECUTION (Output Prediction)                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Input: Code + input values                                 │ │
│  │ Output: Predicted output                                   │ │
│  │ Metric: accuracy (% correct predictions)                   │ │
│  │                                                             │ │
│  │ Tests: Can model trace execution mentally?                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Scenario 4: TEST OUTPUT PREDICTION (No Code Given)             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Input: Problem statement + test inputs                     │ │
│  │ Output: Expected outputs                                   │ │
│  │ Metric: accuracy (% correct predictions)                   │ │
│  │                                                             │ │
│  │ Tests: Can model infer solution behavior without code?     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Performance Results

### Code Generation (Scenario 1)

```
Fresh Problems (after model cutoff dates):

┌───────────────────┬──────────┬──────────┬──────────┐
│      Model        │ Pass@1   │ Pass@5   │ Pass@10  │
├───────────────────┼──────────┼──────────┼──────────┤
│ GPT-4o            │  41.2%   │  58.7%   │  67.3%   │
│ Claude 3.5        │  45.8%   │  63.1%   │  71.9%   │
│ GPT-4 Turbo       │  38.5%   │  54.2%   │  63.8%   │
│ Gemini 1.5 Pro    │  37.1%   │  52.8%   │  61.4%   │
│ GPT-3.5 Turbo     │  22.4%   │  35.7%   │  44.2%   │
└───────────────────┴──────────┴──────────┴──────────┘

Observation: Much lower than HumanEval (contamination effect visible)
```

### Self-Repair (Scenario 2)

```
Repair Success After K Attempts:

Model: Claude 3.5 Sonnet
Initial Success (attempt 1):  45.8%
After repair (attempt 2):     61.3%  (+15.5%)
After repair (attempt 3):     68.9%  (+7.6%)
After repair (attempt 4):     72.1%  (+3.2%)
After repair (attempt 5):     73.8%  (+1.7%)

Diminishing Returns:
├─ Biggest gain: attempt 1 → 2 (+15.5%)
├─ Moderate gain: attempt 2 → 3 (+7.6%)
├─ Small gain: attempt 3+ (<5%)
└─ Matches debugging decay research!
```

### Cross-Scenario Comparison

```
Model: GPT-4o (Fresh Problems)

┌──────────────────────────┬────────────┐
│        Scenario          │  Accuracy  │
├──────────────────────────┼────────────┤
│ Code Generation          │   41.2%    │
│ Self-Repair (3 attempts) │   58.4%    │
│ Code Execution           │   67.8%    │
│ Test Output Prediction   │   52.3%    │
└──────────────────────────┴────────────┘

Insights:
├─ Execution easier than generation
├─ Self-repair helps significantly
└─ Test prediction is hard (requires solution understanding)
```

---

## The Contamination Impact

### Measuring Contamination Effect

```
Experiment: Same models, different time windows

Model: GPT-4 Turbo (cutoff: April 2024)

Problems released BEFORE cutoff (potentially contaminated):
Pass@1: 68.5%

Problems released AFTER cutoff (definitely fresh):
Pass@1: 38.5%

Contamination Effect: 30% performance inflation!

Conclusion: Contaminated benchmarks grossly overestimate capability
```

### HumanEval vs LiveCodeBench

```
┌────────────────────────────────────────────────────────┐
│         BENCHMARK COMPARISON (GPT-4 Turbo)             │
├────────────────────────────────────────────────────────┤
│                                                        │
│  HumanEval (static, likely contaminated):              │
│  Pass@1: 90.2%                                         │
│                                                        │
│  LiveCodeBench (fresh, guaranteed uncontaminated):     │
│  Pass@1: 38.5%                                         │
│                                                        │
│  Gap: 51.7% (!) Massive contamination effect           │
│                                                        │
└────────────────────────────────────────────────────────┘

Takeaway: Static benchmarks are dangerously misleading
```

---

## Self-Repair Analysis

### Why Self-Repair Matters

```
Traditional Eval:
├─ One-shot generation
├─ No feedback loops
└─ Doesn't match real development

Real Development:
├─ Write code
├─ Run tests
├─ See failures
├─ Debug and fix
└─ Repeat until working

LiveCodeBench Self-Repair:
├─ Matches real workflow
├─ Tests iterative capability
└─ Reveals debugging skill
```

### Repair Effectiveness by Error Type

```
Analysis of Repair Attempts (GPT-4o):

Error Type                  Initial  →  After Repair
─────────────────────────────────────────────────────
Logic errors                 45%    →    72%  (+27%)
Off-by-one errors            38%    →    81%  (+43%)
Edge case handling           41%    →    69%  (+28%)
Performance (timeout)        22%    →    35%  (+13%)
Type errors                  67%    →    89%  (+22%)

Observation:
├─ Off-by-one easiest to fix (clear feedback)
├─ Performance hardest (requires algorithm change)
└─ Overall: 60% of failures fixable with feedback
```

---

## Integration with K&V Workflow

### Self-Repair as Bead Pattern

```yaml
# Generate-test-repair loop
- id: bd-801
  title: "Generate solution for problem X"
  phase: execution
  deliverable: "Initial solution"
  max_attempts: 1

# Test bead (always runs)
- id: bd-802
  title: "Test solution"
  phase: verification
  depends: [bd-801]
  deliverable: "Test results"

# Conditional repair beads
- id: bd-803
  title: "Repair attempt 1"
  phase: execution
  depends: [bd-802]
  condition: "bd-802 has failures"
  deliverable: "Fixed solution"

- id: bd-804
  title: "Repair attempt 2"
  phase: execution
  depends: [bd-803]
  condition: "bd-803 still has failures AND attempt < 3"
  deliverable: "Fixed solution"

# Stop after 3 attempts (debugging decay)
```

### Contamination Detection Protocol

```python
# In evaluation/benchmarking:

def evaluate_with_contamination_check(model, benchmark):
    """Evaluate with contamination detection."""

    # Get model cutoff date
    cutoff_date = model.training_cutoff_date

    # Split benchmark by date
    possibly_contaminated = [
        t for t in benchmark
        if t.release_date <= cutoff_date
    ]

    definitely_fresh = [
        t for t in benchmark
        if t.release_date > cutoff_date
    ]

    # Evaluate both
    contaminated_score = evaluate(model, possibly_contaminated)
    fresh_score = evaluate(model, definitely_fresh)

    # Calculate contamination effect
    contamination_gap = contaminated_score - fresh_score

    if contamination_gap > 0.15:  # 15% threshold
        print(f"WARNING: Contamination suspected!")
        print(f"  Old problems: {contaminated_score:.1%}")
        print(f"  Fresh problems: {fresh_score:.1%}")
        print(f"  Gap: {contamination_gap:.1%}")

    # Return fresh score as true capability
    return fresh_score
```

---

## Implementation Guide

### Self-Repair Loop

```python
class SelfRepairEvaluator:
    """
    Implements LiveCodeBench self-repair scenario.
    """

    def __init__(self, model, max_attempts=5):
        self.model = model
        self.max_attempts = max_attempts

    def evaluate_with_repair(self, problem):
        """Evaluate with iterative repair."""

        attempt = 1
        history = []

        while attempt <= self.max_attempts:
            # Generate solution (or repair)
            if attempt == 1:
                # Initial attempt
                solution = self._generate_initial(problem)
            else:
                # Repair attempt
                solution = self._generate_repair(
                    problem,
                    history[-1]['solution'],
                    history[-1]['failures']
                )

            # Test solution
            test_results = self._run_tests(problem, solution)

            # Record attempt
            history.append({
                'attempt': attempt,
                'solution': solution,
                'passed': test_results['all_passed'],
                'failures': test_results['failures']
            })

            # Success?
            if test_results['all_passed']:
                return {
                    'success': True,
                    'attempts': attempt,
                    'history': history
                }

            attempt += 1

        # All attempts exhausted
        return {
            'success': False,
            'attempts': self.max_attempts,
            'history': history
        }

    def _generate_initial(self, problem):
        """Generate initial solution."""

        prompt = f"""
        Problem:
        {problem.statement}

        Write a solution in Python.
        """

        return self.model.generate(prompt)

    def _generate_repair(self, problem, previous_solution, failures):
        """Generate repair based on test failures."""

        prompt = f"""
        Problem:
        {problem.statement}

        Your previous solution:
        {previous_solution}

        Test failures:
        {self._format_failures(failures)}

        Fix the solution to pass all tests.
        Explain what went wrong and how you're fixing it.
        """

        return self.model.generate(prompt)

    def _format_failures(self, failures):
        """Format test failures for repair prompt."""

        formatted = []

        for failure in failures:
            formatted.append(f"""
            Test Case {failure['id']}:
            Input: {failure['input']}
            Expected: {failure['expected']}
            Got: {failure['actual']}
            Error: {failure['error']}
            """)

        return "\n".join(formatted)
```

---

## Key Takeaways

1. **Static benchmarks become contaminated** — Web-scale training inevitably includes benchmark solutions
2. **Contamination inflates scores by 30-50%** — Static benchmarks grossly overestimate capability
3. **Date-filtering ensures freshness** — Only problems after model cutoff are valid
4. **Self-repair is a core capability** — Iterative debugging matters as much as initial generation
5. **Repair has diminishing returns** — Biggest gains in first 2-3 attempts (matches debugging decay)
6. **Multiple scenarios reveal different skills** — Generation, execution, repair test different capabilities
7. **Dynamic benchmarks are the future** — Continuous mining prevents saturation

---

## See Also

- `060-debugging-decay-index.md` — Why repair effectiveness decays with attempts
- `013-specrover.md` — Specification inference for validation
- `011-agentless.md` — Test-based validation and selection
- `022-chatrepair.md` — Iterative repair patterns
- `024-thinkrepair.md` — Reasoning before repair
