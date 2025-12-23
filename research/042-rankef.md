# RankEF: Execution-Feedback Reranking

**Paper:** Sifting through the Chaff: On Utilizing Execution Feedback for Ranking the Generated Code Candidates
**URL:** https://arxiv.org/abs/2408.13976
**Date:** August 2024
**Venue:** ASE 2024

---

## Summary

RankEF addresses a critical gap in code generation: **how to select the best solution from multiple candidates** when you can't afford to execute them all. Traditional "generate N candidates, pick the best" approaches fail because ranking by surface features (e.g., code length, confidence scores) is unreliable.

**Key innovation:** Train a specialized ranker that predicts both correctness AND failure modes by learning from execution feedback patterns, enabling better selection without runtime execution.

---

## The Selection Problem

### Why Selection Matters

```
Generation Pipeline Quality

Without Selection (Pass@1):
  ┌──────────────────────────────┐
  │ Generate 1 solution          │  Success Rate: ~40%
  │ Hope it works                │
  └──────────────────────────────┘

With Sampling (Pass@10, Oracle):
  ┌──────────────────────────────┐
  │ Generate 10 solutions        │  Success Rate: ~85%
  │ Execute all                  │  (if you pick the right one)
  │ Select the one that works    │
  └──────────────────────────────┘

With Sampling (Pass@10, Bad Ranker):
  ┌──────────────────────────────┐
  │ Generate 10 solutions        │  Success Rate: ~45%
  │ Rank by confidence/length    │  (barely better than pass@1)
  │ Pick top-ranked              │
  └──────────────────────────────┘

With Sampling (Pass@10, RankEF):
  ┌──────────────────────────────┐
  │ Generate 10 solutions        │  Success Rate: ~70%
  │ Rank by predicted execution  │  (major improvement without
  │ Pick top-ranked              │   executing all candidates)
  └──────────────────────────────┘
```

**Problem:** The gap between oracle (85%) and bad ranker (45%) wastes generation effort.

---

## RankEF Architecture

### Dual-Objective Learning

```
┌─────────────────────────────────────────────────────────────────┐
│                    RANKEF RANKER MODEL                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Input: Code Candidate                                           │
│      │                                                           │
│      ├──→ Encoder (CodeBERT/GraphCodeBERT)                       │
│      │                                                           │
│      ├──→ Classification Head                                    │
│      │    └──→ Binary: Correct / Incorrect                       │
│      │                                                           │
│      └──→ Feedback Prediction Head                               │
│           ├──→ Error Type (syntax/runtime/logic)                 │
│           ├──→ Error Location (line/function)                    │
│           └──→ Failure Pattern (template match)                  │
│                                                                  │
│  Output: Composite Score                                         │
│      Score = α × P(correct) + β × P(execution_pattern)           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Training Data Generation

```
For each problem P:
    1. Generate N candidates {c₁, c₂, ..., cₙ}
    2. Execute each candidate against test suite
    3. Collect execution feedback:
       - Pass/fail status
       - Error messages
       - Execution traces
       - Failure patterns
    4. Create training pairs:
       (code, correctness_label, failure_pattern)
```

---

## Mathematical Model

### Ranking Score Function

```
For candidate code c:

RankEF_Score(c) = α × P(correct | c) +
                  β × Σ P(failure_pattern_i | c)
                      i∈{syntax, runtime, logic}

Where:
  P(correct | c)           = Binary correctness probability
  P(failure_pattern_i | c) = Predicted failure mode distribution
  α, β                     = Learned weighting coefficients

Selection:
  best_candidate = argmax(RankEF_Score(c))
                   c ∈ candidates
```

### Execution Feedback Templates

Common failure patterns learned by RankEF:

| Pattern | Example Feedback | Predictive Signal |
|---------|------------------|-------------------|
| IndexError | "list index out of range" | Loop bounds issue |
| NameError | "name 'x' is not defined" | Variable scope problem |
| TypeError | "unsupported operand type(s)" | Type mismatch |
| AssertionError | "assertion failed at line 42" | Logic error location |
| TimeoutError | "execution exceeded 5s" | Infinite loop likely |

---

## Benchmark Performance

### Reported Results

| Benchmark | Metric | Baseline (Pass@1) | CodeRanker | RankEF | Improvement |
|-----------|--------|-------------------|------------|--------|-------------|
| APPS (Introductory) | Pass@1 | 25.3% | 28.7% | 33.1% | +30.97% relative |
| APPS (Interview) | Pass@1 | 11.2% | 13.1% | 15.8% | +41.07% relative |
| MBPP | Pass@1 | 52.6% | 56.3% | 61.2% | +16.35% relative |
| HumanEval | Pass@1 | 68.9% | 72.4% | 76.8% | +11.47% relative |

### Ranking Quality vs. Execution Cost

```
Success Rate by Candidates Executed

 80% ┤                                            ████ Oracle (execute all)
 70% ┤                                      ██████
 60% ┤                                ██████            ╔══ RankEF
 50% ┤                          ██████                  ║
 40% ┤                    ██████                        ║
 30% ┤              ██████                              ║
 20% ┤        ██████ Random/Confidence                  ║
 10% ┤  ██████                                          ║
  0% ┼──────────────────────────────────────────────────╨──────
     0    1    2    3    4    5    6    7    8    9   10
                  Number of Candidates Generated

RankEF achieves 70% success with just top-1 selection from 10 candidates
vs. 45% for confidence-based ranking (reported).
```

---

## Integration with Knowledge & Vibes

### Multi-Candidate Generation Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│              K&V MULTI-CANDIDATE WORKFLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Identify High-Risk Beads                                │
│      ├── Complex algorithms                                      │
│      ├── Ambiguous requirements                                  │
│      └── Novel problem domains                                   │
│                                                                  │
│  Step 2: Generate Multiple Approaches                            │
│      ├── Agent 1 → Candidate A (approach 1)                      │
│      ├── Agent 2 → Candidate B (approach 2)                      │
│      └── Agent 3 → Candidate C (approach 3)                      │
│                                                                  │
│  Step 3: Rank Without Full Execution (RankEF-style)              │
│      ├── Static analysis (UBS, typecheck)                        │
│      ├── Lint/complexity metrics                                 │
│      ├── Pattern matching (known failure modes)                  │
│      └── Quick smoke tests (fast subset)                         │
│                                                                  │
│  Step 4: Validate Top Candidate                                  │
│      ├── Full test suite execution                               │
│      ├── If pass → Select this candidate                         │
│      └── If fail → Try next ranked candidate                     │
│                                                                  │
│  Step 5: Fallback Strategy                                       │
│      └── If all fail → ADaPT decompose                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Proxy Execution Signals (Cheaper than Full Execution)

| Signal | Cost | Predictive Power | Implementation |
|--------|------|------------------|----------------|
| Static type check | Very Low | Medium | mypy, TypeScript compiler |
| Linting | Very Low | Low-Medium | ruff, eslint |
| Complexity metrics | Very Low | Low | radon, complexity |
| UBS (unit-based signals) | Low | Medium-High | pytest --collect-only |
| Import checks | Very Low | Medium | ast.parse + import resolver |
| Fast smoke tests | Medium | High | Run 10% of test suite |
| Full test suite | High | Very High | Run all tests |

---

## Practical Implementation

### Code Example: Ranking Function

```python
from dataclasses import dataclass
from typing import List
import subprocess

@dataclass
class CandidateScore:
    """Score components for ranking."""
    static_score: float      # From static analysis
    pattern_score: float     # From failure pattern matching
    smoke_score: float       # From quick tests
    composite: float         # Final ranking score

def rank_candidates(candidates: List[str],
                   test_file: str,
                   quick_tests: List[str]) -> List[tuple[str, CandidateScore]]:
    """
    RankEF-inspired ranking without full execution.

    Args:
        candidates: List of code implementations
        test_file: Path to test suite
        quick_tests: Subset of tests for smoke testing

    Returns:
        List of (candidate, score) sorted by score descending
    """
    scores = []

    for candidate in candidates:
        # Component 1: Static analysis
        static_score = check_static_analysis(candidate)

        # Component 2: Failure pattern prediction
        pattern_score = predict_failure_patterns(candidate)

        # Component 3: Quick smoke tests (10% of suite)
        smoke_score = run_smoke_tests(candidate, quick_tests)

        # Composite score (weighted)
        composite = (
            0.3 * static_score +
            0.3 * pattern_score +
            0.4 * smoke_score
        )

        scores.append((
            candidate,
            CandidateScore(static_score, pattern_score,
                         smoke_score, composite)
        ))

    # Sort by composite score descending
    return sorted(scores, key=lambda x: x[1].composite, reverse=True)

def check_static_analysis(code: str) -> float:
    """
    Run static analysis and return normalized score.

    Returns: 0.0 (many issues) to 1.0 (clean)
    """
    issues = []

    # Type checking
    type_result = subprocess.run(
        ["mypy", "--strict", "-"],
        input=code,
        capture_output=True,
        text=True
    )
    issues.extend(parse_mypy_output(type_result.stderr))

    # Linting
    lint_result = subprocess.run(
        ["ruff", "check", "-"],
        input=code,
        capture_output=True,
        text=True
    )
    issues.extend(parse_ruff_output(lint_result.stdout))

    # Score: 1.0 - (issues / max_acceptable_issues)
    return max(0.0, 1.0 - len(issues) / 10.0)

def predict_failure_patterns(code: str) -> float:
    """
    Check for common failure patterns.

    Returns: 0.0 (many red flags) to 1.0 (clean)
    """
    red_flags = []

    # Pattern 1: Unbounded loops
    if "while True" in code and "break" not in code:
        red_flags.append("infinite_loop_risk")

    # Pattern 2: Unchecked array access
    if "[" in code and "if len(" not in code:
        red_flags.append("index_error_risk")

    # Pattern 3: Unhandled exceptions
    if "raise" in code and "try:" not in code:
        red_flags.append("unhandled_exception")

    # Pattern 4: Magic numbers
    import re
    numbers = re.findall(r'\b\d+\b', code)
    if len([n for n in numbers if int(n) > 1]) > 5:
        red_flags.append("magic_numbers")

    # Score: 1.0 - (red_flags / max_acceptable_flags)
    return max(0.0, 1.0 - len(red_flags) / 5.0)

def run_smoke_tests(code: str, quick_tests: List[str]) -> float:
    """
    Run subset of tests for quick validation.

    Returns: 0.0 (all fail) to 1.0 (all pass)
    """
    passed = 0
    total = len(quick_tests)

    for test in quick_tests:
        try:
            # Write code to temp file and run test
            result = subprocess.run(
                ["pytest", test, "-v"],
                timeout=5,  # Quick timeout
                capture_output=True
            )
            if result.returncode == 0:
                passed += 1
        except subprocess.TimeoutExpired:
            # Timeout counts as failure
            pass

    return passed / total if total > 0 else 0.0
```

---

## When to Use Multi-Candidate Generation

### Decision Matrix

| Bead Characteristics | Use Multi-Candidate? | Number of Candidates |
|----------------------|----------------------|----------------------|
| Clear spec, simple logic | No | 1 |
| Clear spec, complex algorithm | Yes | 2-3 |
| Ambiguous spec, simple logic | Yes | 2-3 |
| Ambiguous spec, complex algorithm | Yes | 3-5 |
| Novel problem domain | Yes | 3-5 |
| Performance-critical | Yes | 3-5 (test different approaches) |
| Security-sensitive | Yes | 3-5 (adversarial validation) |

---

## Caveats and Limitations

### Benchmark vs. Real-World Differences

| Benchmark Context | Real-World Context | Adaptation Needed |
|-------------------|-------------------|-------------------|
| Self-contained functions | Multi-file modules | Rank at module level |
| Public test suites | Private/unknown tests | Use proxy signals |
| Clear correctness | Subjective quality | Multi-criteria ranking |
| Single language | Polyglot codebases | Language-specific rankers |

### Cost Considerations

```
Cost Analysis (per bead):

Single-Candidate:
  Generation: 1 × cost
  Validation: 1 × test_cost
  Total: 1 × (cost + test_cost)

Multi-Candidate (N=5) + RankEF:
  Generation: 5 × cost
  Ranking: 5 × proxy_cost (proxy_cost << test_cost)
  Validation: 1-2 × test_cost (only top candidates)
  Total: 5 × cost + 5 × proxy_cost + 1.5 × test_cost

Worthwhile when:
  Increased success rate × value > extra generation cost
```

---

## Key Takeaways

1. **Selection is first-class:** Generating multiple candidates is wasted without good selection
2. **Proxy signals work:** Static analysis + patterns approximate execution at much lower cost
3. **Composite scoring wins:** Combine multiple signals for better ranking than any single metric
4. **Execution feedback teaches:** Failure patterns from past executions predict future failures
5. **Tiered validation:** Quick checks first, expensive validation only for top candidates
6. **Cost-quality tradeoff:** More candidates = higher cost but better chance of success
7. **Apply strategically:** Use multi-candidate generation for high-risk/high-value beads

---

## See Also

- `041-debatecoder.md` — Multi-agent debate for code selection
- `043-rlef.md` — Learning from execution feedback
- `045-repairagent.md` — Tool-based validation loops
- `048-eg-cfg.md` — Execution-guided generation
- `060-debugging-decay-index.md` — Why iteration limits matter
- `038-adapt.md` — When to decompose instead of retry
