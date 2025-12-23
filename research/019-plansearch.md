---
title: PlanSearch - Natural Language Plan Search
tags: [planning, search, diversity, code-generation, sampling, observations]
protocols: [P3, P10]
---

# PlanSearch: Search Over Natural-Language Plans

**Paper:** Planning In Natural Language Improves LLM Search For Code Generation
**URL:** https://arxiv.org/html/2409.03733v1
**Date:** September 2024
**Venue:** arXiv preprint

---

## Summary

Critical research demonstrating that **searching over plans** (natural language strategies) is more effective than searching over code implementations. PlanSearch achieves substantial pass@k improvements by generating diverse problem-solving approaches first, then implementing each.

**Key insight:** Idea diversity > implementation diversity. Searching in "plan space" explores fundamentally different solutions; searching in "code space" produces near-duplicates.

---

## The Search Problem

### Why Repeated Sampling Fails

```
Traditional Approach: Sample code N times

┌─────────────────────────────────────────────────────────────────┐
│           NAIVE REPEATED SAMPLING (N=10)                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Sample 1: Binary search (correct)                ✓             │
│  Sample 2: Binary search (off-by-one error)       ✗             │
│  Sample 3: Binary search (same off-by-one error)  ✗             │
│  Sample 4: Binary search (different variable names) ✗           │
│  Sample 5: Binary search (slightly different logic) ✓           │
│  Sample 6: Binary search (formatting variation)   ✗             │
│  Sample 7: Binary search (another variation)      ✗             │
│  Sample 8: Linear scan (correct but slow)         ✗ (timeout)   │
│  Sample 9: Binary search (yet another variation)  ✗             │
│  Sample 10: Binary search (similar to sample 1)   ✓             │
│                                                                  │
│  Result: 3/10 correct (30%)                                     │
│  Problem: 9/10 are binary search variations                     │
│           Low diversity in approaches                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The Diversity Problem

```
Code-Space Search:
├─ Generate 100 implementations
├─ Most are variations of the same approach
├─ Different syntax, same algorithm
└─ Wasted sampling budget on near-duplicates

Plan-Space Search:
├─ Generate 10 different strategies
├─ Each strategy is fundamentally different
├─ Implement top K strategies
└─ Better coverage of solution space
```

---

## The PlanSearch Architecture

### Two-Stage Search

```
┌─────────────────────────────────────────────────────────────────┐
│                    PLANSEARCH PIPELINE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STAGE 1: PLAN GENERATION (Search in Idea Space)                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Step 1: Generate Observations                             │ │
│  │  ├─ What patterns does the problem exhibit?                │ │
│  │  ├─ What constraints matter?                               │ │
│  │  ├─ What are edge cases?                                   │ │
│  │  └─ Output: N diverse observations (N=20-30)               │ │
│  │                                                             │ │
│  │  Step 2: Synthesize Plans                                  │ │
│  │  ├─ Combine observations into strategies                   │ │
│  │  ├─ Each plan describes an approach                        │ │
│  │  ├─ Natural language, not code                             │ │
│  │  └─ Output: M candidate plans (M=10-15)                    │ │
│  │                                                             │ │
│  │  Step 3: Rank Plans                                        │ │
│  │  ├─ Score each plan for correctness likelihood            │ │
│  │  ├─ Consider: Completeness, clarity, edge case coverage   │ │
│  │  └─ Output: Top K plans (K=3-5)                            │ │
│  │                                                             │ │
│  └────────────────────────┬───────────────────────────────────┘ │
│                           │                                     │
│                           ▼                                     │
│  STAGE 2: IMPLEMENTATION (Search in Code Space)                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  For each plan (top K):                                    │ │
│  │  ├─ Generate L implementations (L=3-5)                     │ │
│  │  ├─ Each follows the plan's strategy                       │ │
│  │  ├─ Variations in implementation details                   │ │
│  │  └─ Total: K × L candidate solutions                       │ │
│  │                                                             │ │
│  └────────────────────────┬───────────────────────────────────┘ │
│                           │                                     │
│                           ▼                                     │
│  STAGE 3: VALIDATION (Test and Select)                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ├─ Run all K × L solutions                                │ │
│  │  ├─ Filter: Keep only passing solutions                    │ │
│  │  ├─ Rerank: By confidence/efficiency                       │ │
│  │  └─ Return: Best solution                                  │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Plan Generation Examples

### Problem: Find Median of Two Sorted Arrays

```
OBSERVATION GENERATION (Sample):

Observation 1:
"Both arrays are sorted - we can exploit this property"

Observation 2:
"Median is the middle element(s) - we need to find the Kth element"

Observation 3:
"We can use binary search on the smaller array to partition efficiently"

Observation 4:
"For arrays of size m and n, median is at position (m+n+1)/2"

Observation 5:
"Edge cases: empty arrays, arrays of size 1, unequal sizes"

... (20+ observations generated)

PLAN SYNTHESIS (Sample Plans):

Plan A: Binary Search Partition
"Use binary search on the smaller array to find the correct partition point.
Ensure left side has (m+n+1)/2 elements total. Check partition validity by
comparing edge elements. Handle odd/even length cases separately."

Plan B: Merge and Extract
"Merge both sorted arrays into a single sorted array using two pointers.
Extract the median from the merged array. Simple but O(m+n) space."

Plan C: Two-Pointer Kth Element
"Use two pointers to find the Kth element without merging. Advance pointers
based on which array has the smaller current element. Stop when K elements seen."

Plan D: Divide and Conquer
"Recursively eliminate half of one array based on median comparisons.
Reduce problem size until base case (small arrays). Combine results."

PLAN RANKING:

Plan A: 0.92 (most efficient, handles edge cases)
Plan C: 0.85 (simple, correct, but slower)
Plan B: 0.71 (correct but inefficient for large inputs)
Plan D: 0.68 (complex, harder to implement correctly)

Selected for implementation: Plans A, C, B (top 3)
```

---

## Performance Results

### Pass@k Improvements

```
LiveCodeBench (Claude 3.5 Sonnet):

┌──────────────────────┬─────────────┬─────────────┬───────────┐
│      Approach        │   Pass@1    │   Pass@10   │  Pass@200 │
├──────────────────────┼─────────────┼─────────────┼───────────┤
│ Naive sampling       │    45.8%    │    62.3%    │   71.4%   │
│ PlanSearch           │    47.2%    │    68.7%    │   83.1%   │
│                      │   (+1.4%)   │   (+6.4%)   │  (+11.7%) │
└──────────────────────┴─────────────┴─────────────┴───────────┘

Observation:
├─ Modest pass@1 improvement (plan generation overhead)
├─ Strong pass@10 improvement (diversity pays off)
└─ Massive pass@200 improvement (search space better covered)
```

### Diversity Analysis

```
Solution Diversity (measured by edit distance):

Naive Sampling (100 solutions):
├─ Unique approaches: 2.3 (most are variations)
├─ Average edit distance: 15% (very similar code)
└─ Algorithmic diversity: Low

PlanSearch (100 solutions):
├─ Unique approaches: 8.7 (fundamentally different)
├─ Average edit distance: 47% (substantially different)
└─ Algorithmic diversity: High

Impact:
├─ PlanSearch covers 3.8x more algorithmic approaches
├─ Higher chance of finding correct approach
└─ Better use of sampling budget
```

---

## Token Cost Analysis

### The Cost-Benefit Tradeoff

```
┌─────────────────────────────────────────────────────────────────┐
│              PLANSEARCH COST ANALYSIS                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Naive Sampling (100 attempts):                                 │
│  ├─ Prompt: 500 tokens × 100 = 50K tokens                       │
│  ├─ Generation: 300 tokens × 100 = 30K tokens                   │
│  └─ Total: 80K tokens                                           │
│                                                                  │
│  PlanSearch (10 plans, 10 implementations each):                │
│  ├─ Observation gen: 500 prompt + 2K output = 2.5K             │
│  ├─ Plan synthesis: 2.5K prompt + 1.5K output = 4K             │
│  ├─ Plan ranking: 4K prompt + 0.5K output = 4.5K               │
│  ├─ Implementation: (1K prompt + 300 output) × 100 = 130K       │
│  └─ Total: 141K tokens (+76% cost)                              │
│                                                                  │
│  ROI Analysis:                                                   │
│  ├─ Cost increase: +76%                                         │
│  ├─ Pass@100 increase: +16%                                     │
│  └─ Effective cost per success: -9% (better!)                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

Conclusion: Higher upfront cost, but better success rate
makes it more cost-effective overall.
```

---

## Implementation Guide

### Plan Generation

```python
class PlanSearchGenerator:
    """
    Implements PlanSearch: search over plans, then implement.
    """

    def __init__(self, model):
        self.model = model

    def solve(self, problem, num_plans=10, implementations_per_plan=5):
        """Generate and implement multiple plans."""

        # Stage 1: Generate diverse observations
        observations = self._generate_observations(problem)

        # Stage 2: Synthesize plans from observations
        plans = self._synthesize_plans(problem, observations, num_plans)

        # Stage 3: Rank plans
        ranked_plans = self._rank_plans(plans, problem)

        # Stage 4: Implement top plans
        solutions = []
        for plan in ranked_plans[:5]:  # Top 5 plans
            implementations = self._implement_plan(
                problem,
                plan,
                implementations_per_plan
            )
            solutions.extend(implementations)

        # Stage 5: Validate and select
        best_solution = self._validate_and_select(problem, solutions)

        return best_solution

    def _generate_observations(self, problem, num_observations=20):
        """Generate diverse observations about the problem."""

        observations = []

        prompt = f"""
        Problem:
        {problem.statement}

        Generate {num_observations} diverse observations about this problem.

        Consider:
        - What patterns or properties does the input have?
        - What mathematical/algorithmic insights apply?
        - What are the edge cases?
        - What constraints matter?
        - What common pitfalls exist?

        Generate ONE observation at a time.
        """

        for i in range(num_observations):
            observation = self.model.generate(
                prompt + f"\n\nObservation {i+1}:"
            )
            observations.append(observation)

        return observations

    def _synthesize_plans(self, problem, observations, num_plans):
        """Combine observations into coherent plans."""

        prompt = f"""
        Problem:
        {problem.statement}

        Observations:
        {self._format_observations(observations)}

        Synthesize {num_plans} different solution strategies.

        Each strategy should:
        - Describe a distinct algorithmic approach
        - Be written in natural language (not code)
        - Reference relevant observations
        - Explain how it handles edge cases
        - Include time/space complexity

        Generate diverse strategies, not variations.
        """

        plans = []
        for i in range(num_plans):
            plan = self.model.generate(
                prompt + f"\n\nStrategy {i+1}:"
            )
            plans.append({
                'id': i,
                'description': plan,
                'observations_used': self._extract_observations(plan, observations)
            })

        return plans

    def _rank_plans(self, plans, problem):
        """Rank plans by likelihood of correctness."""

        scored_plans = []

        for plan in plans:
            prompt = f"""
            Problem:
            {problem.statement}

            Proposed Strategy:
            {plan['description']}

            Evaluate this strategy on:
            1. Correctness: Will it solve all cases? (0-1)
            2. Completeness: Handles all edge cases? (0-1)
            3. Clarity: Is the approach clear? (0-1)
            4. Efficiency: Good time/space complexity? (0-1)

            Return JSON with scores and overall confidence.
            """

            evaluation = self.model.generate(prompt)
            plan['score'] = evaluation['overall_confidence']
            scored_plans.append(plan)

        # Sort by score
        scored_plans.sort(key=lambda p: p['score'], reverse=True)

        return scored_plans

    def _implement_plan(self, problem, plan, num_implementations):
        """Generate multiple implementations of a plan."""

        implementations = []

        for i in range(num_implementations):
            prompt = f"""
            Problem:
            {problem.statement}

            Strategy to implement:
            {plan['description']}

            Implement this strategy in Python.

            Implementation {i+1}:
            """

            code = self.model.generate(prompt)
            implementations.append({
                'code': code,
                'plan': plan,
                'variation': i
            })

        return implementations

    def _validate_and_select(self, problem, solutions):
        """Test all solutions and select the best."""

        passing = []

        for solution in solutions:
            # Run tests
            results = run_tests(problem, solution['code'])

            if results['all_passed']:
                passing.append({
                    **solution,
                    'test_results': results
                })

        if not passing:
            return None  # No solution passed

        # Rank passing solutions
        # Prefer: higher plan score, better efficiency, simpler code
        passing.sort(
            key=lambda s: (
                s['plan']['score'],
                -s['test_results']['runtime'],
                -len(s['code'])
            ),
            reverse=True
        )

        return passing[0]
```

---

## Integration with K&V Workflow

### Plan-Based Bead Decomposition

```yaml
# Phase 0: Planning (Generate alternative strategies)
- id: bd-plan-generation
  title: "Generate solution strategies"
  phase: planning
  deliverable: "5 diverse solution strategies with rankings"

# Phase 1: Implementation (Parallel execution of top plans)
- id: bd-impl-plan-a
  title: "Implement Strategy A (Binary Search)"
  phase: execution
  depends: [bd-plan-generation]
  deliverable: "3 implementations of Strategy A"

- id: bd-impl-plan-b
  title: "Implement Strategy B (Two Pointers)"
  phase: execution
  depends: [bd-plan-generation]
  deliverable: "3 implementations of Strategy B"

- id: bd-impl-plan-c
  title: "Implement Strategy C (Merge)"
  phase: execution
  depends: [bd-plan-generation]
  deliverable: "3 implementations of Strategy C"

# Phase 2: Validation (Test all implementations)
- id: bd-validation
  title: "Validate all implementations"
  phase: verification
  depends: [bd-impl-plan-a, bd-impl-plan-b, bd-impl-plan-c]
  deliverable: "Best solution selected from passing implementations"
```

### Calibration with Plan Diversity

```python
# In calibration:

def assess_plan_diversity(generated_plans):
    """Check if plans are sufficiently diverse."""

    # Compute pairwise similarity
    similarities = []
    for i, plan_a in enumerate(generated_plans):
        for j, plan_b in enumerate(generated_plans[i+1:]):
            sim = compute_similarity(plan_a, plan_b)
            similarities.append(sim)

    avg_similarity = sum(similarities) / len(similarities)

    if avg_similarity > 0.7:  # Too similar
        report = {
            'status': 'INSUFFICIENT_DIVERSITY',
            'avg_similarity': avg_similarity,
            'recommendation': 'Generate more diverse plans'
        }
    else:
        report = {
            'status': 'SUFFICIENT_DIVERSITY',
            'avg_similarity': avg_similarity,
            'unique_approaches': count_unique_approaches(generated_plans)
        }

    return report
```

---

## Key Takeaways

1. **Search over ideas, not implementations** — Plan-space search finds fundamentally different approaches
2. **Diversity is the goal** — Near-duplicate code wastes sampling budget
3. **Two-stage search is powerful** — Generate plans, then implement top plans
4. **Natural language plans work** — Don't need formal specifications
5. **Pass@k benefits more than pass@1** — Diversity pays off with more samples
6. **Higher upfront cost, better ROI** — Plan generation costs tokens but improves success rate
7. **Observations drive plan diversity** — Rich observation set → diverse plans

---

## See Also

- `020-codetree.md` — Tree search with strategy branching
- `019-plansearch.md` — Plan-based search
- `013-specrover.md` — Specification as plan validation
- `014-codeplan.md` — Multi-step planning for edits
- `038-adapt.md` — Adaptive planning and decomposition
