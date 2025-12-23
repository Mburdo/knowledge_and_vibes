---
title: FLAMES - Semantic-Guided Search for Program Repair
tags: [search, test-guided, memory-efficient, reward-function, best-first, repair]
protocols: [P6, P8]
---

# FLAMES: Semantic-Guided Search for Program Repair

**Paper:** Memory-Efficient Large Language Models for Program Repair with Semantic-Guided Patch Generation (FLAMES)
**URL:** https://arxiv.org/abs/2410.16655
**Date:** October 2024
**Venue:** arXiv preprint

---

## Summary

Memory-efficient program repair system that replaces beam search with **semantic-guided best-first search**. Uses test feedback as reward signal to guide token-level exploration. Achieves **133 fixes on Defects4J** with **83% VRAM reduction** vs beam-search baselines.

**Key innovation:** Execution feedback should steer search, not just validate. Greedy decoding + test-guided branching beats brute-force beam search.

---

## The Memory Problem in APR

### Why Beam Search Explodes

```
┌─────────────────────────────────────────────────────────────────┐
│              BEAM SEARCH FOR PROGRAM REPAIR                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Token 1: "def" → k=32 beams → 32 states in memory              │
│  Token 2: "fix" → 32×32=1024 candidates → OOM!                  │
│  Token 3: ...  → 32k+ states → GPU crashes                      │
│                                                                  │
│  VRAM Usage:                                                     │
│  Beam width 1:   2GB                                             │
│  Beam width 8:   16GB                                            │
│  Beam width 32:  64GB+ (requires multi-GPU)                      │
│                                                                  │
│  Problem: Memory grows exponentially with beam width            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    FLAMES (Greedy + Guided)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Beam width: 1 (greedy decoding)                                │
│  Memory: 2GB (constant)                                          │
│                                                                  │
│  Search guidance: Test feedback                                  │
│  Token 1: "def" → Run tests → Reward signal                     │
│  Token 2: "fix" → Run tests → Branch if promising               │
│  Token 3: ...  → Guided by execution results                    │
│                                                                  │
│  Result: 83% less VRAM, better results                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### VRAM Comparison

| Approach | Beam Width | VRAM | Fixes (Defects4J) |
|----------|------------|------|-------------------|
| Beam search | 32 | 64GB | ~120 |
| Beam search | 8 | 16GB | ~105 |
| **FLAMES** | **1 (greedy)** | **11GB** | **133** |

---

## FLAMES Architecture

### Best-First Search with Test Rewards

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLAMES SEARCH ALGORITHM                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Initialize:                                                     │
│  ├── Start with greedy decoding (beam=1)                        │
│  ├── Maintain priority queue of partial patches                 │
│  └── Score each partial patch by test feedback                  │
│                                                                  │
│  Search Loop:                                                    │
│  ┌────────────────────────────────────────────────┐             │
│  │ 1. Pop highest-scoring partial patch          │             │
│  │                                                │             │
│  │ 2. Generate next token(s) greedily             │             │
│  │                                                │             │
│  │ 3. If checkpoint reached:                      │             │
│  │    ├── Run tests on partial patch              │             │
│  │    ├── Compute reward from test results        │             │
│  │    └── Update priority                         │             │
│  │                                                │             │
│  │ 4. If tests show promise:                      │             │
│  │    └── Branch: explore alternative tokens      │             │
│  │                                                │             │
│  │ 5. If tests fail badly:                        │             │
│  │    └── Prune: abandon this search path         │             │
│  │                                                │             │
│  │ 6. If patch complete:                          │             │
│  │    └── Validate with full test suite           │             │
│  │                                                │             │
│  │ 7. Continue until success or budget exhausted  │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Reward Function Design

### Test-Based Reward Signal

```python
def compute_reward(partial_patch, test_results):
    """
    Compute reward from test execution feedback.
    """
    reward = 0.0

    # Primary signal: tests passing
    passing_rate = test_results.passed / test_results.total
    reward += passing_rate * 10.0

    # Secondary signal: test progress
    if test_results.passed > previous_passed:
        reward += 5.0  # Bonus for improvement

    # Penalty for new failures
    if test_results.failed > previous_failed:
        reward -= 3.0

    # Syntax/compilation bonus
    if test_results.compiles:
        reward += 2.0
    else:
        reward -= 10.0  # Heavy penalty for syntax errors

    # Targeted test signal (bug-exposing test)
    if test_results.target_test_passed:
        reward += 20.0  # Large bonus for fixing the bug

    return reward
```

### Checkpoints for Test Execution

```
Code Generation Progress:
├── Token 0-10: Function signature
│   └── Checkpoint 1: Can it compile? (Quick check)
├── Token 11-25: Bug fix region
│   └── Checkpoint 2: Does target test pass?
├── Token 26-40: Rest of function
│   └── Checkpoint 3: Do all tests pass?
└── Complete
    └── Final validation

Checkpoints reduce test overhead while providing guidance
```

---

## Branching Strategy

### When to Branch

```python
class SearchNode:
    def __init__(self, partial_patch, reward):
        self.patch = partial_patch
        self.reward = reward
        self.branches = []

    def should_branch(self):
        """
        Decide whether to explore alternative tokens.
        """
        # Branch if showing promise but not perfect
        if 0.3 < self.reward < 0.8:
            return True

        # Branch at critical decision points
        if self.at_bug_location():
            return True

        # Don't branch if clearly failing
        if self.reward < 0.1:
            return False

        # Don't branch if already succeeding
        if self.reward > 0.9:
            return False

        return False

    def generate_branches(self, k=3):
        """
        Generate k alternative next tokens.
        """
        # Sample top-k tokens from model
        alternatives = self.model.sample_top_k(
            self.patch,
            k=k,
            temperature=0.8
        )

        for alt in alternatives:
            branch = SearchNode(
                self.patch + alt,
                reward=self.compute_reward()
            )
            self.branches.append(branch)
```

---

## Reported Results

### Performance Metrics

```
Defects4J Benchmark:

FLAMES Results:
┌──────────────────────────────────────┐
│ Bugs fixed:     133                  │
│ Success rate:   ~34%                 │
│ VRAM usage:     11GB                 │
│ Improvement:    +10 fixes over SOTA  │
│ Efficiency:     83% VRAM reduction   │
└──────────────────────────────────────┘

Additional Benchmarks:
- HumanEval-Java: 72% pass@1
- QuixBugs: 89% pass@1
- Transformed APR datasets: +12% avg improvement
```

### Cost-Effectiveness

| Metric | Beam Search (32) | FLAMES (greedy+guided) |
|--------|------------------|----------------------|
| **Fixes** | 120 | 133 |
| **VRAM** | 64GB | 11GB |
| **Time** | 45min/bug | 38min/bug |
| **GPU cost** | 4x A100 | 1x A100 |
| **$/fix** | $3.20 | $0.85 |

---

## Mathematical Model

### Search Efficiency

```
Beam Search:
  States explored ∝ beam_width^depth
  Memory ∝ beam_width × model_size
  No execution feedback until completion

FLAMES (Best-First):
  States explored ∝ log(beam_width) × feedback_quality
  Memory ∝ 1 × model_size (constant)
  Execution feedback guides search

If feedback quality Q > 0.5:
  FLAMES explores fewer states than beam search
  While achieving better or equal results
```

### Reward-Guided Efficiency

```
Let R(p) = reward for partial patch p
Let B(p) = should_branch(p)

Expected states explored:
  Beam search: O(b^d) where b=beam width, d=depth
  FLAMES: O(Σ B(p)) where B(p) ∝ R(p) uncertainty

Empirical observation:
  FLAMES branches ~3-5 times per bug
  vs beam search exploring 32^d states
```

---

## Integration with K&V Workflow

### Test-Guided Repair Pattern

FLAMES validates the K&V principle: execution signals should steer work, not just validate.

| K&V Pattern | FLAMES Evidence |
|-------------|-----------------|
| Tests guide iteration | Reward function based on test pass rate |
| Early feedback is valuable | Checkpoints during generation |
| Prune bad directions early | Low-reward paths abandoned |
| Resource constraints matter | 83% VRAM reduction enables deployment |

### Bead-Level Test-Guided Search

```markdown
## Repair Bead with Test Guidance

**Bead ID:** bd-repair-123
**Approach:** flames-style

**Bug:** Division by zero in calculate_average()

**Search Log:**
1. **Initial attempt** (greedy):
   - Generated: `if values:`
   - Tests: 2/5 pass
   - Reward: 0.4
   - Decision: BRANCH (showing promise)

2. **Branch 1** (`if values:`):
   - Generated: `    return sum(values) / len(values)`
   - Tests: 2/5 pass (still fails on empty)
   - Reward: 0.4
   - Decision: PRUNE (no progress)

3. **Branch 2** (`if len(values) > 0:`):
   - Generated: `    return sum(values) / len(values)`
   - Tests: 5/5 pass
   - Reward: 1.0
   - Decision: ACCEPT

**Result:** Fixed in 3 attempts (vs 32 beam states)
```

---

## Critical Insight: Feedback as Search Guide

### Why Execution Beats Probability

```
┌─────────────────────────────────────────────────────────────────┐
│           PROBABILITY vs EXECUTION FOR SEARCH                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PROBABILITY-GUIDED (Beam Search):                               │
│  ├── Token 1: "if" (p=0.8)                                       │
│  ├── Token 2: "x" (p=0.6)                                        │
│  ├── Token 3: ">" (p=0.7)                                        │
│  └── Result: High probability but wrong fix                     │
│                                                                  │
│  EXECUTION-GUIDED (FLAMES):                                      │
│  ├── Token 1: "if" → Tests: 1/5 pass (reward=0.2)               │
│  │   └── Try alternative: "while" → Tests: 0/5 (reward=0.0)     │
│  │       └── Backtrack to "if"                                  │
│  ├── Token 2: "len" → Tests: 3/5 pass (reward=0.6)              │
│  │   └── Good progress, continue                                │
│  ├── Token 3: "(" → Tests: 3/5 pass (reward=0.6)                │
│  └── Result: Guided to correct fix by test feedback             │
│                                                                  │
│  Key Difference: Execution provides ground truth                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practical Implications

### When to Use Test-Guided Search

```
Use FLAMES-style approach when:
✓ Tests available and informative
✓ Memory/compute constrained
✓ Partial execution is cheap
✓ Incremental feedback possible
✓ Search space is large

Beam search may be better when:
○ No tests available
○ Execution is expensive
○ Memory abundant
○ Small search space
```

### Designing Reward Functions

```python
# Good reward function characteristics:

1. CONTINUOUS
   - Not just binary pass/fail
   - Partial credit for progress
   - Example: 3/5 tests passing = 0.6 reward

2. DIFFERENTIATING
   - Small code changes = measurable reward change
   - Guides model to right direction
   - Example: Fixing one more test = +0.2 reward

3. FAST TO COMPUTE
   - Run subset of tests for speed
   - Cache test results
   - Example: Run 5 critical tests, not all 100

4. ALIGNED WITH GOAL
   - Reward correlates with correctness
   - Penalize regressions
   - Example: Target test passing = high reward
```

---

## Implementation Details

### Checkpoint Placement

```python
def determine_checkpoints(code_length):
    """
    Decide when to run tests during generation.
    """
    checkpoints = []

    # Always check after signature (can it compile?)
    checkpoints.append(code_length * 0.2)

    # Check at bug location (main fix point)
    checkpoints.append(code_length * 0.5)

    # Check near end (behavioral preservation)
    checkpoints.append(code_length * 0.9)

    return checkpoints
```

### Search Algorithm

```python
def flames_search(bug, model, max_budget=1000):
    """
    FLAMES best-first search with test guidance.
    """
    # Priority queue: (reward, partial_patch)
    queue = PriorityQueue()

    # Start with empty patch
    initial = PartialPatch(code="", reward=0.0)
    queue.push(initial)

    states_explored = 0
    while not queue.empty() and states_explored < max_budget:
        # Get most promising partial patch
        current = queue.pop()

        # Greedy next token
        next_token = model.generate_greedy(current.code)
        extended = current.extend(next_token)

        # Check if at checkpoint
        if extended.at_checkpoint():
            # Run tests and compute reward
            test_results = run_tests(extended)
            extended.reward = compute_reward(test_results)

            # Decide: branch, prune, or continue
            if extended.should_branch():
                # Generate alternative branches
                for alt in model.sample_alternatives(current.code):
                    branch = current.extend(alt)
                    branch.reward = compute_reward(run_tests(branch))
                    queue.push(branch)

            elif extended.reward < 0.1:
                # Prune: abandon this path
                continue

        # If complete, validate
        if extended.is_complete():
            if validate_fix(extended, bug):
                return extended

        queue.push(extended)
        states_explored += 1

    return None  # No fix found
```

---

## Critical Caveats

### What FLAMES Doesn't Solve

1. **Test Signal Quality Dependency**
   - Weak tests mislead search
   - Reward function is only as good as tests
   - May converge on plausible-but-wrong fix
   - See: `021-swe-bench-plus.md`

2. **Search Policy Engineering**
   - Reward function design is critical
   - Checkpoint placement affects efficiency
   - Branching heuristics need tuning
   - Not fully automatic

3. **Partial Execution Overhead**
   - Running tests frequently has cost
   - Not all bugs have fast tests
   - May slow down vs pure generation
   - Trade-off: better guidance vs more overhead

4. **APR Scope Limitations**
   - Still focused on single-function fixes
   - Repo-scale issues harder
   - Multi-file coordination not addressed
   - Environment setup assumed

---

## Research Methodology

### Experimental Setup

- **Datasets:** Defects4J, HumanEval-Java, QuixBugs
- **Baselines:** Beam search (width 8, 16, 32), CoCoNut, AlphaRepair
- **Hardware:** Single NVIDIA A100 (40GB) for FLAMES, 4x A100 for beam-32
- **Metrics:** Fix rate, VRAM usage, wall-clock time

### Ablation Studies

| Configuration | Fix Rate | VRAM |
|---------------|----------|------|
| Greedy only (no guidance) | 18% | 2GB |
| + Test checkpoints | 28% | 4GB |
| + Branching | 32% | 8GB |
| **+ Reward-guided pruning** | **34%** | **11GB** |

---

## Key Metrics to Track

When implementing test-guided search:

```json
{
  "bug_id": "Defects4J_Math_42",
  "approach": "flames",
  "search_stats": {
    "states_explored": 87,
    "branches_created": 4,
    "paths_pruned": 12,
    "checkpoints_evaluated": 23
  },
  "resource_usage": {
    "vram_peak_gb": 11.2,
    "wall_time_sec": 142,
    "test_executions": 23
  },
  "reward_progression": [
    {"step": 10, "reward": 0.2},
    {"step": 25, "reward": 0.6},
    {"step": 42, "reward": 1.0}
  ],
  "result": "success"
}
```

---

## Key Takeaways

1. **Execution feedback beats probability** — Tests guide search better than model confidence
2. **Memory efficiency enables deployment** — 83% VRAM reduction makes single-GPU repair viable
3. **Strategic branching outperforms brute force** — Selective exploration beats beam search
4. **Test quality gates effectiveness** — Weak tests mislead test-guided search
5. **Resource constraints drive innovation** — Efficiency requirements led to better algorithm

---

## See Also

- `022-chatrepair.md` — Iterative repair with test feedback
- `021-swe-bench-plus.md` — Test quality determines search effectiveness
- `060-debugging-decay-index.md` — Why bounded search matters
- `023-toggle.md` — Constrained generation reduces search space
- `024-thinkrepair.md` — Knowledge-guided repair
