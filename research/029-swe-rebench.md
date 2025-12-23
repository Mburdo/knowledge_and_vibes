# SWE-rebench: Decontaminated, Standardized SWE Evaluation

**Paper:** SWE-rebench: An Automated Pipeline for Task Collection and Decontaminated Evaluation of Software Engineering Agents
**URL:** https://arxiv.org/pdf/2505.20411
**Date:** May 2025
**Venue:** arXiv preprint

---

## Summary

Automated pipeline for continuous SWE agent evaluation that addresses **contamination** and **scaffold variability**. Generates **>21k Python tasks** with decontamination tracking and standardized minimal scaffold. Emphasizes statistical rigor: **pass@5 with mean/SEM** instead of single-run scores.

**Key innovation:** Separate model capability from scaffold engineering by standardizing evaluation harness and explicitly tracking contamination risk.

---

## The Two-Problem Crisis

### Problem 1: Contamination

```
┌─────────────────────────────────────────────────────────────────┐
│                  THE CONTAMINATION CRISIS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  2023: SWE-bench released                                        │
│  2024: Models train on SWE-bench data                            │
│  2025: "SOTA" scores include contaminated models                 │
│                                                                  │
│  Problem: Can't tell if improvement is:                          │
│  A) Better model capability                                      │
│  B) Memorized training data                                      │
│                                                                  │
│  Result: Leaderboard scores are meaningless                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Problem 2: Scaffold Variability

```
┌─────────────────────────────────────────────────────────────────┐
│              THE SCAFFOLD ENGINEERING PROBLEM                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Lab A: Custom agent scaffold                                    │
│  ├── 50+ specialized tools                                      │
│  ├── Multi-round planning                                       │
│  ├── Extensive prompt engineering                               │
│  └── Score: 45%                                                 │
│                                                                  │
│  Lab B: Different scaffold                                       │
│  ├── 20 different tools                                         │
│  ├── Single-shot approach                                       │
│  ├── Different prompts                                          │
│  └── Score: 40%                                                 │
│                                                                  │
│  Question: Is A better than B?                                   │
│  Answer: UNKNOWN - scaffolds incomparable                        │
│                                                                  │
│  Result: Can't compare systems across labs                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## SWE-rebench Solution

### Dual-Track Approach

```
┌─────────────────────────────────────────────────────────────────┐
│                    SWE-REBENCH ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TRACK 1: Contamination Control                                  │
│  ┌────────────────────────────────────────────────┐             │
│  │ 1. Continuously mine fresh tasks               │             │
│  │ 2. Track task date vs model cutoff            │             │
│  │ 3. Mark contamination risk explicitly          │             │
│  │ 4. Separate pre/post-cutoff evaluation         │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  TRACK 2: Scaffold Standardization                               │
│  ┌────────────────────────────────────────────────┐             │
│  │ 1. Define minimal ReAct-style scaffold         │             │
│  │ 2. Fixed tool set (read, write, bash, search)  │             │
│  │ 3. Standardized prompts                        │             │
│  │ 4. Report uncertainty (pass@5, mean±SEM)       │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  Result: Fair, reproducible comparisons                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Automated Task Collection Pipeline

### Four-Stage Generation

```
┌─────────────────────────────────────────────────────────────────┐
│               TASK COLLECTION PIPELINE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STAGE 1: Issue Mining (Automated)                               │
│  ┌────────────────────────────────────────────────┐             │
│  │ GitHub API scan:                               │             │
│  │ ├── Filter: Python repos, >100 stars           │             │
│  │ ├── Filter: Issue + merged PR                  │             │
│  │ ├── Filter: Tests exist                        │             │
│  │ └── Result: ~50k candidate pairs               │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  STAGE 2: Environment Recipe Generation (LLM-Assisted)           │
│  ┌────────────────────────────────────────────────┐             │
│  │ For each candidate:                           │             │
│  │ 1. Analyze repo structure                     │             │
│  │ 2. Generate Dockerfile (LLM)                  │             │
│  │ 3. Build environment                          │             │
│  │ 4. If fails: Diagnose with LLM               │             │
│  │ 5. Iteratively repair (max 5 rounds)          │             │
│  │ Result: ~35k buildable environments           │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  STAGE 3: Execution Validation                                   │
│  ┌────────────────────────────────────────────────┐             │
│  │ Run tests in environment:                     │             │
│  │ ├── Before fix: Must fail                    │             │
│  │ ├── After fix: Must pass                     │             │
│  │ ├── Determinism check (3 runs)                │             │
│  │ └── Quality scoring                           │             │
│  │ Result: ~21k high-quality tasks               │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  STAGE 4: Contamination Tagging                                  │
│  ┌────────────────────────────────────────────────┐             │
│  │ For each task:                                 │             │
│  │ ├── Record issue creation date                │             │
│  │ ├── Tag contamination risk level              │             │
│  │ ├── Group into temporal cohorts               │             │
│  │ └── Enable filtered evaluation                │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Standardized Minimal Scaffold

### Design Principles

```
Minimal Scaffold = Just enough to be functional
Not maximal = Avoid over-engineering

Goals:
1. Fair comparison across models
2. Measure model capability, not prompt engineering
3. Reproducible across labs
4. Simple enough to understand
```

### The ReAct Loop

```python
STANDARD_SCAFFOLD = """
Agent loop (max 20 iterations):

1. Think: What should I do next?
2. Act: Choose tool + arguments
   Tools available:
   - read_file(path)
   - write_file(path, content)
   - bash(command)
   - search_code(query)
   - search_web(query)
3. Observe: See tool output
4. Repeat until done or max iterations

Fixed prompt template:
  - Issue description
  - Repository structure
  - Available tools
  - Previous observations
  - Next action request
"""
```

### Tool Set (Standardized)

| Tool | Purpose | Arguments |
|------|---------|-----------|
| `read_file` | Read file contents | `path` |
| `write_file` | Write/edit file | `path`, `content` |
| `bash` | Run shell command | `command` |
| `search_code` | Search codebase | `query` |
| `search_web` | Web search (optional) | `query` |

**No custom tools allowed** — Levels playing field.

---

## Contamination Tracking

### Temporal Cohorts

```
Task Cohorts by Creation Date:

Cohort 2021: Issues from 2021
├── GPT-4 (cutoff Apr 2023): CONTAMINATED
├── Claude 3 (cutoff Aug 2023): CONTAMINATED
└── Gemini (cutoff Nov 2023): CONTAMINATED

Cohort 2023: Issues from 2023
├── GPT-4 (cutoff Apr 2023): SOME CONTAMINATED
├── Claude 3 (cutoff Aug 2023): PARTIAL
└── Gemini (cutoff Nov 2023): MOSTLY CLEAN

Cohort 2024+: Issues from 2024 onwards
├── GPT-4 (cutoff Apr 2023): CLEAN
├── Claude 3 (cutoff Aug 2023): CLEAN
└── Gemini (cutoff Nov 2023): CLEAN
```

### Explicit Contamination Labels

```json
{
  "task_id": "repo-123",
  "issue_date": "2024-03-15",
  "contamination_risk": {
    "gpt-4": "low",      // Cutoff Apr 2023 < Mar 2024
    "claude-3": "low",   // Cutoff Aug 2023 < Mar 2024
    "gemini": "low",     // Cutoff Nov 2023 < Mar 2024
    "llama-3": "medium"  // May have seen via web crawl
  },
  "eval_recommendation": "Use for all models"
}
```

---

## Statistical Rigor: pass@k with Uncertainty

### Why Single-Run Scores Mislead

```
Traditional reporting:
  Model A: 45% (single run)
  Model B: 43% (single run)

  Conclusion: A > B?
  Reality: UNKNOWN (could be noise)

SWE-rebench reporting:
  Model A: 45.2% ± 2.1% (pass@5, mean ± SEM)
  Model B: 43.8% ± 1.9% (pass@5, mean ± SEM)

  Conclusion: Overlapping confidence intervals
              → No significant difference
```

### The pass@k Protocol

```python
def evaluate_with_uncertainty(model, tasks, k=5):
    """
    Evaluate with multiple runs and report uncertainty.
    """
    results = []

    for task in tasks:
        successes = 0
        for run in range(k):
            # Different random seed per run
            result = model.solve(task, seed=run)
            if result.passed_tests:
                successes += 1

        pass_at_k = (successes > 0)  # At least 1 success
        results.append(pass_at_k)

    # Compute mean and SEM
    mean = np.mean(results)
    sem = np.std(results) / np.sqrt(len(results))

    return {
        "pass@k": k,
        "mean": mean,
        "sem": sem,
        "ci_95": (mean - 1.96*sem, mean + 1.96*sem)
    }
```

---

## Reported Results

### Dataset Scale

```
SWE-rebench Collection (May 2025):

Total tasks: 21,347
Repositories: ~800 Python repos
Success rate: ~43% (from 50k candidates)

Temporal distribution:
├── 2018-2020: 3,247 tasks (likely contaminated)
├── 2021-2022: 5,891 tasks (partially contaminated)
├── 2023: 6,102 tasks (mostly fresh)
└── 2024-2025: 6,107 tasks (completely fresh)

Difficulty distribution:
├── Easy (1-5 lines): 40%
├── Medium (5-20 lines): 45%
├── Hard (20+ lines): 15%
```

### Scaffold Impact on Scores

| Model | Custom Scaffold | Standard Scaffold | Difference |
|-------|-----------------|-------------------|------------|
| GPT-4 | 48% | 42% | -6 points |
| Claude 3 Opus | 45% | 39% | -6 points |
| Gemini 1.5 Pro | 42% | 36% | -6 points |

**Key finding:** Scaffold engineering accounts for ~6 points (~15% relative).

---

## Mathematical Model

### Contamination + Scaffold Effect

```
Reported Score = True Capability + Scaffold Boost + Contamination

S_reported = T + B_scaffold + C_contamination

Empirical estimates:
  B_scaffold ≈ 6 points (from ablation)
  C_contamination ≈ 10-15 points (from temporal analysis)

True capability:
  T = S_standard_scaffold_fresh_data

Example:
  Reported: 48% (custom scaffold, old benchmark)
  Standard scaffold: 42% (B=6 removed)
  Fresh data: 32% (C=10 removed)
  True capability: 32%
```

---

## Integration with K&V Workflow

### Standardized Calibration

SWE-rebench validates the K&V principle: measure capability, not prompt tricks.

| K&V Pattern | SWE-rebench Evidence |
|-------------|---------------------|
| Evidence-based claims | Statistical uncertainty required |
| Reproducibility | Standardized scaffold enables reproduction |
| Temporal awareness | Contamination tracking prevents false confidence |
| Fair comparison | Level playing field across approaches |

### Calibration Reporting Template

```markdown
## Agent Evaluation (SWE-rebench Protocol)

**Model:** GPT-4 (Apr 2023 cutoff)
**Scaffold:** Standard minimal (no custom tools)
**Data:** Fresh cohort (2024+ tasks only)

**Results:**
- Tasks attempted: 1,247
- pass@5: 38.2% ± 1.4% (mean ± SEM)
- 95% CI: [35.5%, 40.9%]

**Contamination Check:**
- Pre-cutoff tasks: 0 (excluded)
- Post-cutoff tasks: 1,247 (100%)
- Risk level: LOW

**Comparison to Baseline:**
- vs Random agent: 2.1% ± 0.3%
- Improvement: +36.1 points (p < 0.001)
```

---

## Critical Caveats

### What SWE-rebench Doesn't Solve

1. **Minimal Scaffold is Still a Choice**
   - No such thing as "no scaffold"
   - Different minimums possible
   - Tool set selection matters
   - Prompt wording has impact

2. **Automation Introduces Biases**
   - LLM-generated environments may favor certain repos
   - Build success correlates with ecosystem popularity
   - Python-first bias in automation
   - Missing complex setup cases

3. **Statistical Rigor is Expensive**
   - pass@5 costs 5x compute
   - Many labs can't afford it
   - Still underestimates variance
   - Need even more runs for significance

4. **Contamination Detection is Imperfect**
   - Only tracks issue creation date
   - Models may see via web crawl
   - Training data provenance unclear
   - Arms race with data collection

---

## Practical Implications

### When to Use Standardized Evaluation

```
Use standardized scaffold when:
✓ Comparing models fairly
✓ Academic publication
✓ Reproducibility required
✓ Measuring model capability
✓ Avoiding prompt engineering

Use custom scaffold when:
✓ Production deployment
✓ Maximizing performance
✓ Domain-specific needs
✓ Engineering for specific use case
```

### Designing Minimal Scaffolds

```python
MINIMAL_SCAFFOLD_PRINCIPLES = """
1. COMPLETENESS
   - Include all necessary tools
   - But no more than necessary
   - Document what's excluded

2. TRANSPARENCY
   - Public implementation
   - Exact prompts shared
   - Seed for reproducibility

3. FAIRNESS
   - Same tools for all models
   - No model-specific optimizations
   - Level playing field

4. SIMPLICITY
   - Easy to understand
   - Easy to reproduce
   - Minimal moving parts
"""
```

---

## Research Methodology

### Pipeline Automation Details

```python
# Simplified pipeline
def rebench_pipeline():
    """
    End-to-end automated task collection.
    """
    # 1. Mine issues
    issues = github_api.search(
        language="python",
        has_pr=True,
        has_tests=True,
        stars=">100"
    )

    # 2. Build environments
    environments = []
    for issue in issues:
        env = repolaunch(issue)
        if env.builds_successfully:
            environments.append(env)

    # 3. Validate
    tasks = []
    for env in environments:
        if validate_task(env):
            task = package_task(env)
            task["contamination_risk"] = assess_risk(task)
            tasks.append(task)

    return tasks
```

### Quality Control

```
Automated Checks:
├── Environment builds (100% required)
├── Tests run (100% required)
├── Deterministic failures (>95%)
├── PR fixes issue (>90%)
└── Appropriate difficulty (<5% trivial)

Manual Review (10% sample):
├── Issue clarity: 78%
├── Fix quality: 82%
├── Test coverage: 71%
└── Overall quality: Good
```

---

## Key Metrics to Track

When using standardized evaluation:

```json
{
  "model": "gpt-4",
  "cutoff_date": "2023-04-01",
  "scaffold": "minimal-react",
  "evaluation": {
    "total_tasks": 1247,
    "cohort": "2024-fresh",
    "contamination_filtered": true,
    "runs_per_task": 5
  },
  "results": {
    "pass_at_5": 0.382,
    "mean": 0.382,
    "sem": 0.014,
    "ci_95_lower": 0.355,
    "ci_95_upper": 0.409
  },
  "comparison": {
    "custom_scaffold_score": 0.48,
    "scaffold_boost_estimate": 0.10,
    "contaminated_score": 0.52,
    "contamination_estimate": 0.14
  }
}
```

---

## Key Takeaways

1. **Contamination inflates scores 10-15 points** — Temporal filtering is mandatory
2. **Scaffolds matter 5-10 points** — Standardization enables fair comparison
3. **Uncertainty must be reported** — Single-run scores hide variance
4. **Automation enables scale** — 21k+ tasks with minimal human effort
5. **True capability requires both** — Clean data + standard scaffold

---

## See Also

- `028-swe-bench-live.md` — Continuous updates (complementary approach)
- `021-swe-bench-plus.md` — Quality filtering and test strength
- `027-swe-smith.md` — Synthetic data for training (not evaluation)
- `060-debugging-decay-index.md` — Why statistical rigor matters
- `065-confucius-code-agent.md` — Scaffold design for scale
