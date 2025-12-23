# SWE-bench-Live: Continuously Updating SWE Benchmark

**Paper:** SWE-bench Goes Live!
**URL:** https://arxiv.org/abs/2505.23419
**Date:** May 2025
**Venue:** arXiv preprint

---

## Summary

Continuously updating benchmark for software engineering agents with **~1,319 tasks** from post-2024 GitHub issues across **93 repositories**. Addresses contamination and benchmark drift through monthly updates and automated instance creation.

**Key innovation:** Benchmarks must evolve as fast as models train. Static benchmarks rot—live benchmarks stay fresh and reveal true generalization.

---

## The Static Benchmark Problem

### Why Benchmarks Degrade

```
┌─────────────────────────────────────────────────────────────────┐
│               STATIC BENCHMARK LIFECYCLE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Month 0: Benchmark Released                                     │
│  ├── Novel tasks                                                │
│  ├── Clean evaluation                                           │
│  └── Scores reflect true capability                             │
│                                                                  │
│  Month 3: Models Start Training on Benchmark                     │
│  ├── Tasks appear in training data                              │
│  ├── Contamination begins                                       │
│  └── Scores start inflating                                     │
│                                                                  │
│  Month 6: Leaderboard Chasing                                    │
│  ├── Overfitting to benchmark artifacts                         │
│  ├── Prompt engineering for specific tasks                      │
│  └── Scores no longer meaningful                                │
│                                                                  │
│  Month 12: Benchmark is Obsolete                                 │
│  ├── All models "trained" on it                                 │
│  ├── Environments have drifted                                  │
│  ├── Repos have evolved                                         │
│  └── Need new benchmark                                         │
│                                                                  │
│  Result: 12-18 month useful lifetime                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Contamination Impact

| Time Since Release | Contamination Risk | Score Inflation |
|-------------------|-------------------|-----------------|
| 0-3 months | Low | 0-5% |
| 3-6 months | Medium | 5-15% |
| 6-12 months | High | 15-30% |
| 12+ months | Very High | 30-50%+ |

---

## SWE-bench-Live Architecture

### Continuous Update Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│              SWE-BENCH-LIVE UPDATE CYCLE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MONTHLY CYCLE:                                                  │
│                                                                  │
│  Week 1: Issue Mining                                            │
│  ┌────────────────────────────────────────────────┐             │
│  │ 1. Scan GitHub for new issues/PRs             │             │
│  │ 2. Filter by criteria:                        │             │
│  │    - Created after model cutoff               │             │
│  │    - Has failing test                         │             │
│  │    - Has merged PR fix                        │             │
│  │    - Deterministic reproduction               │             │
│  │ 3. Collect ~150-200 candidates                │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  Week 2: Environment Building (RepoLaunch)                       │
│  ┌────────────────────────────────────────────────┐             │
│  │ For each candidate:                           │             │
│  │ 1. Clone repository at issue state            │             │
│  │ 2. Build Docker environment                   │             │
│  │ 3. Install dependencies                       │             │
│  │ 4. Verify tests run                           │             │
│  │ 5. Test failure is reproducible               │             │
│  │ 6. Fix applies cleanly                        │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  Week 3: Validation                                              │
│  ┌────────────────────────────────────────────────┐             │
│  │ 1. Manual review of 10% sample               │             │
│  │ 2. Automated quality checks                   │             │
│  │ 3. Difficulty rating                          │             │
│  │ 4. Deduplication                              │             │
│  │ Result: ~100 high-quality instances           │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  Week 4: Release                                                 │
│  ┌────────────────────────────────────────────────┐             │
│  │ 1. Add to live benchmark                      │             │
│  │ 2. Freeze "lite" split for tracking           │             │
│  │ 3. Archive previous month's instances         │             │
│  │ 4. Update leaderboard                         │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  Next month: Repeat cycle                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## RepoLaunch: Automated Environment Creation

### The Environment Challenge

Traditional approach:
- Manual Docker file creation: 2-4 hours/repo
- Dependency resolution: Trial and error
- Test harness setup: Custom per repo
- Total: 4-8 hours per instance

RepoLaunch automation:
- Analyze repo structure: 5 minutes
- Generate environment recipe: 10 minutes
- Iterative repair on failures: 15 minutes
- Total: ~30 minutes per instance (10x faster)

### RepoLaunch Algorithm

```python
def repolaunch_pipeline(issue_url):
    """
    Automated environment creation for GitHub issue.
    """
    # 1. Extract metadata
    metadata = parse_issue(issue_url)
    repo = clone_repo(metadata.repo_url, metadata.commit_sha)

    # 2. Analyze repository
    analysis = analyze_repo_structure(repo)
    # - Language/framework detection
    # - Dependency file discovery
    # - Test framework identification
    # - Build system detection

    # 3. Generate environment recipe
    recipe = generate_docker_recipe(analysis)

    # 4. Iterative refinement
    max_attempts = 5
    for attempt in range(max_attempts):
        # Build environment
        env = build_environment(recipe)

        # Run tests
        result = run_tests(env, repo)

        if result.success:
            return env  # Success!

        # Failed: analyze error and refine recipe
        error_analysis = llm_analyze_error(result.error_log)
        recipe = refine_recipe(recipe, error_analysis)

    return None  # Failed to build

def llm_analyze_error(error_log):
    """
    Use LLM to diagnose environment failures.
    """
    prompt = f"""
Environment build failed with this error:
{error_log}

Common causes:
- Missing system dependency
- Wrong Python/Node version
- Incompatible package versions
- Missing environment variable

Diagnose the issue and suggest a fix for the Dockerfile.
"""

    diagnosis = llm.generate(prompt)
    return diagnosis
```

---

## Reported Results

### Dataset Statistics

```
SWE-bench-Live (Initial Release):

Total instances: ~1,319
Repositories: 93
Date range: Jan 2024 - May 2025
Languages: Python (primary), JavaScript, TypeScript

Instance Distribution:
├── Simple (1-5 line fix):      35%
├── Medium (5-20 line fix):     45%
├── Complex (20+ line fix):     15%
├── Multi-file (>1 file):       5%

Update Frequency: Monthly
New instances/month: ~80-120
Archived instances: Previous month freezes
```

### Performance Degradation vs Static Benchmark

```
Agent Performance Comparison:

System X (claims 45% on SWE-bench):
├── SWE-bench (static, 2023):       45%
├── SWE-bench-Live (May 2025):      28%  ← Real capability
└── Degradation:                    -17 points

System Y (claims 40% on SWE-bench):
├── SWE-bench (static, 2023):       40%
├── SWE-bench-Live (May 2025):      22%
└── Degradation:                    -18 points

Key finding: Most systems show 15-20 point drop on fresh tasks
```

---

## Why Fresh Tasks Reveal Reality

### Contamination Effects

```
┌─────────────────────────────────────────────────────────────────┐
│              CONTAMINATION vs GENERALIZATION                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Contaminated (Static Benchmark):                                │
│  ├── Model has seen similar tasks                               │
│  ├── Can pattern match to training data                         │
│  ├── May memorize solutions                                     │
│  └── Score doesn't reflect true capability                      │
│                                                                  │
│  Fresh (Live Benchmark):                                         │
│  ├── Post-cutoff tasks                                          │
│  ├── Must actually solve, not recall                            │
│  ├── Tests true generalization                                  │
│  └── Score reflects real-world performance                      │
│                                                                  │
│  Gap = Contamination + Overfitting                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Mathematical Model

### Benchmark Decay Function

```
Let S(t) = measured score at time t
Let C(t) = contamination level at time t
Let T = true capability

S(t) = T + C(t)

Where C(t) grows over time:
  C(t) = C_max × (1 - e^(-λt))

  C_max ≈ 0.15-0.30 (15-30 point inflation)
  λ ≈ 0.1-0.2 (half-life ~3-6 months)

Fresh benchmark provides:
  S_fresh ≈ T (since C(0) ≈ 0)

Therefore:
  True capability = S_live
  Contamination effect = S_static - S_live
```

---

## Integration with K&V Workflow

### Drift Detection and Adaptation

SWE-bench-Live validates the K&V principle: reality changes, plans must adapt.

| K&V Pattern | SWE-bench-Live Evidence |
|-------------|------------------------|
| Assumptions drift | Repo environments change monthly |
| Re-validate periodically | Fresh tasks reveal gaps |
| Temporal calibration | What worked last month may not work today |
| Evidence freshness | Post-cutoff data is higher quality |

### Continuous Calibration Pattern

```markdown
## K&V Calibration with Live Benchmarks

### Monthly Calibration Cycle:

**Week 1: Fresh Benchmark Available**
1. Run agents on new SWE-bench-Live instances
2. Measure performance vs previous month
3. Identify capability gaps

**Week 2: Analyze Failures**
1. What changed in repos/environments?
2. Which agent strategies failed?
3. What new patterns emerged?

**Week 3: Adapt Strategies**
1. Update prompts/tools for new patterns
2. Retrain on previous month's tasks
3. Refine agent protocols

**Week 4: Re-validate**
1. Test updated agents
2. Measure improvement
3. Document learnings

Result: Continuous improvement loop
```

---

## Critical Caveats

### What SWE-bench-Live Doesn't Solve

1. **Still Expensive**
   - Building environments is automated but not free
   - Validation requires human review
   - Monthly updates require infrastructure
   - Cost: ~$10k-20k/month to operate

2. **Language Scope**
   - Early versions Python-heavy
   - Multi-language coverage evolving
   - Different ecosystems have different challenges
   - JavaScript, Rust, Go need separate pipelines

3. **Monthly Updates Complicate Comparisons**
   - Scores change as benchmark evolves
   - Need frozen "lite" splits for tracking
   - Apples-to-apples comparison harder
   - Leaderboard instability

4. **Doesn't Prevent All Contamination**
   - Models can still train on prior months
   - Only protects against pre-cutoff contamination
   - Arms race with training data collection
   - Need explicit contamination tracking

---

## Practical Implications

### When to Use Live vs Static Benchmarks

```
Use Live Benchmark when:
✓ Need to measure true generalization
✓ Evaluating production deployment
✓ Comparing to fresh baselines
✓ Detecting contamination
✓ Tracking real capability trends

Use Static Benchmark when:
✓ Need reproducible comparisons
✓ Historical tracking of same instances
✓ Academic research requiring fixed sets
✓ Cost/infrastructure constraints
✓ Stable baseline for development
```

### Designing Temporal Validation

```python
TEMPORAL_VALIDATION_STRATEGY = """
1. CUTOFF DISCIPLINE
   - Never use data after model knowledge cutoff
   - Explicitly track cutoff dates
   - Document temporal boundaries

2. FRESHNESS TRACKING
   - Mark instances by creation date
   - Separate pre/post-cutoff evaluation
   - Report scores by temporal cohort

3. DRIFT MONITORING
   - Compare static vs live scores monthly
   - Track performance degradation
   - Alert on significant gaps

4. CONTAMINATION DETECTION
   - Test on held-out future data
   - Measure score drop on fresh tasks
   - Estimate contamination levels
"""
```

---

## Research Methodology

### Instance Selection Criteria

```
Automated Filters:
├── Issue created after Jan 2024 (temporal isolation)
├── Has merged PR with fix (ground truth available)
├── PR modifies code (not just docs/config)
├── Tests exist and fail before fix
├── Tests pass after fix (deterministic)
├── Single PR fixes issue (clear causation)
└── Environment buildable (via RepoLaunch)

Manual Review (sample):
├── Issue description is clear
├── Fix is reasonable in scope
├── Not a trivial typo/formatting
├── Representative of real work
└── Appropriate difficulty
```

### Quality Metrics

| Metric | Target | Actual |
|--------|--------|--------|
| Environment build success | >80% | 87% |
| Test determinism | >95% | 96% |
| Issue clarity (manual review) | >70% | 74% |
| Fix reasonableness | >80% | 83% |

---

## Key Metrics to Track

When using live benchmarks:

```json
{
  "evaluation_date": "2025-05-15",
  "benchmark_version": "live-2025-05",
  "model_cutoff_date": "2024-12-01",
  "results": {
    "total_instances": 1319,
    "attempted": 1200,
    "resolved": 336,
    "pass_rate": 0.28
  },
  "temporal_analysis": {
    "pre_cutoff_tasks": {"count": 0, "pass_rate": null},
    "post_cutoff_tasks": {"count": 1319, "pass_rate": 0.28},
    "contamination_estimate": 0.0
  },
  "comparison_to_static": {
    "static_score": 0.45,
    "live_score": 0.28,
    "gap": 0.17,
    "interpretation": "Likely contaminated on static"
  }
}
```

---

## Key Takeaways

1. **Static benchmarks rot** — Contamination inflates scores 15-30 points within months
2. **Fresh tasks reveal truth** — Post-cutoff evaluation shows real capability
3. **Monthly updates required** — Benchmarks must evolve as fast as models train
4. **Automation enables scale** — RepoLaunch makes continuous updates feasible
5. **Temporal discipline is critical** — Must track cutoff dates and contamination risk

---

## See Also

- `029-swe-rebench.md` — Decontaminated evaluation with standardized scaffolds
- `021-swe-bench-plus.md` — Quality filtering and decontamination
- `027-swe-smith.md` — Synthetic data generation (training side)
- `065-confucius-code-agent.md` — Long-horizon sessions need fresh validation
- `060-debugging-decay-index.md` — Why agents degrade over time (similar to benchmarks)
