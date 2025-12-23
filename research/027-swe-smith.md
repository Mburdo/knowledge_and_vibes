# SWE-smith: Scaling Training Data for SWE Agents

**Paper:** SWE-smith: Scaling Data for Software Engineering Agents
**URL:** https://arxiv.org/abs/2504.21798
**Date:** April 2025
**Venue:** arXiv preprint

---

## Summary

Synthetic data generation system that creates **~50k SWE task instances** across 128 repositories by automatically introducing bugs that break existing tests. Enables training open-weight agents that achieve **~40% pass@1 on SWE-bench Verified**.

**Key innovation:** Instead of curating expensive real tasks, synthesize them at scale by breaking tests and packaging as solvable issues. Tests are the training signal.

---

## The Training Data Problem

### Why SWE Agent Training is Data-Hungry

```
┌─────────────────────────────────────────────────────────────────┐
│              THE SWE TRAINING DATA CHALLENGE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Real SWE-bench Task Requirements:                               │
│  ├── Working environment (Docker, dependencies)                 │
│  ├── Valid issue description                                    │
│  ├── Reproduction steps                                         │
│  ├── Test suite                                                 │
│  ├── Ground-truth patch                                         │
│  └── Manual validation                                          │
│                                                                  │
│  Cost per task: ~2-4 hours of human effort                      │
│  SWE-bench: 2,294 tasks (enormous manual effort)                │
│                                                                  │
│  For Training:                                                   │
│  Need: 10k-100k examples                                         │
│  Reality: ~2k available                                          │
│  Gap: 5-50x insufficient                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    SWE-SMITH SOLUTION                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Start with: Repo with passing tests                            │
│       ↓                                                          │
│  1. Introduce bug (automated)                                    │
│  2. Verify tests now fail                                        │
│  3. Package as issue                                             │
│  4. Store original code as ground truth                          │
│       ↓                                                          │
│  Result: Synthetic task instance                                 │
│                                                                  │
│  Cost per task: ~2-5 minutes (automated)                         │
│  SWE-smith: 50k tasks (minimal human effort)                     │
│  Improvement: 100x faster, 20x more data                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## The SWE-smith Pipeline

### Four-Stage Generation

```
┌─────────────────────────────────────────────────────────────────┐
│                  SWE-SMITH TASK GENERATION                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STAGE 1: Repository Selection                                   │
│  ┌────────────────────────────────────────────────┐             │
│  │ Criteria:                                      │             │
│  │ ├── Has test suite (>50 tests)                │             │
│  │ ├── Tests pass (baseline quality)             │             │
│  │ ├── Python codebase                           │             │
│  │ └── Reasonable size (1k-50k LOC)              │             │
│  │                                                │             │
│  │ Result: 128 repos selected                    │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  STAGE 2: Bug Injection                                          │
│  ┌────────────────────────────────────────────────┐             │
│  │ For each function/class in repo:              │             │
│  │ ├── LLM rewrites to introduce bug             │             │
│  │ ├── AST transformations (mutations)           │             │
│  │ ├── PR mirroring (revert real fixes)          │             │
│  │ └── Semantic perturbations                    │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  STAGE 3: Validation                                             │
│  ┌────────────────────────────────────────────────┐             │
│  │ Run test suite:                                │             │
│  │ ├── At least 1 test must fail                 │             │
│  │ ├── Not ALL tests fail (too broken)           │             │
│  │ ├── Failure is deterministic                  │             │
│  │ └── Original code passes (sanity check)       │             │
│  │                                                │             │
│  │ If valid: → Stage 4                           │             │
│  │ If invalid: → Discard, try again              │             │
│  └────────────────────────────────────────────────┘             │
│                   │                                              │
│                   ↓                                              │
│  STAGE 4: Task Packaging                                         │
│  ┌────────────────────────────────────────────────┐             │
│  │ Create task instance:                          │             │
│  │ {                                              │             │
│  │   "instance_id": "repo-123",                   │             │
│  │   "buggy_code": "<modified>",                  │             │
│  │   "oracle_code": "<original>",                 │             │
│  │   "test_patch": "<failing tests>",             │             │
│  │   "issue_description": "<generated>",          │             │
│  │   "environment": "repo-env"                    │             │
│  │ }                                              │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Bug Injection Strategies

### 1. LLM-Based Rewriting

```python
def llm_bug_injection(function_code):
    """
    Use LLM to introduce subtle logic bugs.
    """
    prompt = f"""
Rewrite this function to introduce a subtle bug:

Original (correct) code:
{function_code}

Requirements:
1. Introduce ONE logic error
2. Keep syntax valid
3. Make bug subtle (not obvious typo)
4. Preserve function signature
5. Bug should cause test failures

Buggy code:
"""

    buggy_code = llm.generate(prompt)
    return buggy_code

# Example:
# Original: if x > 0:
# Buggy:    if x >= 0:  (off-by-one)
```

### 2. AST Transformations

```python
def ast_mutation(code):
    """
    Apply AST-level mutations (zero-cost bug injection).
    """
    mutations = [
        # Operator mutations
        (">", ">="),   # Comparison
        ("==", "!="),  # Equality
        ("+", "-"),    # Arithmetic

        # Constant mutations
        ("0", "1"),
        ("True", "False"),

        # Control flow
        ("if", "if not"),
        ("and", "or"),
    ]

    tree = ast.parse(code)
    mutator = ASTMutator(mutations)
    buggy_tree = mutator.visit(tree)
    return ast.unparse(buggy_tree)
```

### 3. PR Mirroring

```python
def pr_mirroring(repo, pr_id):
    """
    Revert a real bug fix PR to recreate the bug.
    """
    # Get fix PR from repo history
    pr = repo.get_pr(pr_id)

    # Extract diff
    fixed_code = pr.after
    buggy_code = pr.before

    # Verify this was a bug fix:
    # - Original code (before) fails tests
    # - Fixed code (after) passes tests

    if validates_as_bug_fix(pr):
        return {
            "buggy": buggy_code,
            "oracle": fixed_code,
            "issue": pr.description
        }
```

### 4. Semantic Perturbations

```python
def semantic_perturbation(code):
    """
    Small semantic changes that break behavior.
    """
    perturbations = [
        # Edge case handling
        remove_null_check,
        remove_bounds_check,
        remove_type_check,

        # Logic simplification
        remove_special_case,
        simplify_conditional,

        # Resource management
        remove_cleanup,
        remove_lock,
    ]

    for perturb in perturbations:
        buggy = perturb(code)
        if causes_test_failure(buggy):
            return buggy
```

---

## Reported Results

### Dataset Scale

```
SWE-smith Generation Results:

Repositories: 128 Python repos
Attempts: ~200k bug injections
Valid bugs: ~50k (25% success rate)
Avg tasks/repo: ~390

Task Distribution:
├── Simple bugs (1-5 line fix):     60%
├── Medium bugs (5-15 line fix):    30%
├── Complex bugs (15+ line fix):    10%
└── Multi-file bugs:                5%
```

### Training Results

| Model | Training Data | SWE-bench Verified | SWE-bench Lite |
|-------|--------------|-------------------|----------------|
| GPT-4 (0-shot) | None | ~38% | ~45% |
| Open 32B (untrained) | None | ~12% | ~15% |
| **Open 32B (SWE-smith)** | **50k synthetic** | **~40%** | **~48%** |

**Key finding:** Synthetic data enables open-weight agents to match GPT-4 class performance.

---

## Why This Works: Tests as Training Signal

### The Test-Centric View

```
Traditional supervised learning:
  Input: Code + issue
  Output: Patch
  Signal: Human annotation

SWE-smith approach:
  Input: Failing tests + code
  Output: Code that passes tests
  Signal: Test execution (automated)

Key insight:
  Tests ARE the specification
  Passing tests = correct behavior
  No human annotation needed
```

### Training Loss

```python
def swe_training_loss(model, task):
    """
    Training loss based on test outcomes.
    """
    # Generate patch
    patch = model.generate(task.buggy_code, task.issue)

    # Apply patch
    patched_code = apply(task.buggy_code, patch)

    # Run tests
    test_results = run_tests(patched_code, task.tests)

    # Compute loss
    if test_results.all_passed:
        # Correct fix: low loss
        loss = -log_likelihood(patch | task)
    else:
        # Wrong fix: high loss
        loss = +penalty(test_results.failures)

    return loss
```

---

## Mathematical Model

### Synthetic Data Scaling

```
Let N = number of synthetic examples
Let D = diversity of bug types
Let Q = quality of bugs (test signal strength)

Agent performance ∝ N^α × D^β × Q^γ

Empirical findings:
  α ≈ 0.3 (diminishing returns after ~50k)
  β ≈ 0.5 (diversity matters greatly)
  γ ≈ 0.7 (quality dominates)

Implication:
  Better to have 10k high-quality diverse bugs
  Than 100k low-quality similar bugs
```

---

## Integration with K&V Workflow

### Building Project-Specific Training Data

SWE-smith enables project-level knowledge accumulation:

| K&V Pattern | SWE-smith Application |
|-------------|----------------------|
| Accumulate validated fixes | Each closed bead = potential training example |
| Test-driven development | Tests define correctness (same as SWE-smith) |
| Domain-specific patterns | Generate bugs in project's style |
| Continuous improvement | Re-train on new project data |

### Project Knowledge Pool Generation

```markdown
## Using SWE-smith Patterns in K&V

### After Each Phase:

1. **Collect completed beads**
   - Buggy code: State before fix
   - Oracle code: State after fix
   - Tests: Validation suite

2. **Generate variations**
   - Mutate fixes to create more bugs
   - Different fix approaches for same bug
   - Edge cases discovered during work

3. **Build project training set**
   - Store as (bug, fix, test) tuples
   - Embed for retrieval (like ThinkRepair)
   - Use for few-shot examples

4. **Re-train or fine-tune**
   - Project-specific agent
   - Specialized for codebase patterns
   - Bootstrapped from validated work
```

---

## Critical Caveats

### What SWE-smith Doesn't Solve

1. **Synthetic ≠ Real**
   - Generated bugs may not match real bug distribution
   - Real issues have vague descriptions
   - Real bugs often lack good tests
   - Deployment gaps remain

2. **Test Dependence**
   - Only works for repos with tests
   - Test quality determines bug quality
   - Weak tests → weak training signal
   - See: `021-swe-bench-plus.md`

3. **Language/Ecosystem Scope**
   - Currently Python-focused
   - Generalization to other languages open
   - Different ecosystems need different strategies
   - JavaScript, Rust, Go have different patterns

4. **Bug Distribution Bias**
   - Easier to generate simple bugs
   - Complex, architectural bugs harder to synthesize
   - May overtrain on easy patterns
   - Real-world bugs may be different

---

## Practical Implications

### When to Generate Synthetic Data

```
Generate synthetic training data when:
✓ Have repos with good test suites
✓ Need large-scale training data
✓ Can't afford manual curation
✓ Want domain-specific agent
✓ Iterative training is feasible

Manual curation may be better when:
○ Small-scale training (<1k examples)
○ Very specific bug types needed
○ Test quality is poor
○ Quality >>> quantity
```

### Designing Bug Injection

```python
BUG_INJECTION_PRINCIPLES = """
1. VALIDITY
   - Must cause test failures
   - Must be fixable
   - Must have deterministic behavior

2. DIVERSITY
   - Cover different bug types
   - Various complexity levels
   - Multiple fix strategies

3. REALISM
   - Bugs that could occur naturally
   - Not arbitrary nonsense
   - Realistic issue descriptions

4. BALANCE
   - 60% simple, 30% medium, 10% complex
   - Mix of bug categories
   - Different parts of codebase
"""
```

---

## Research Methodology

### Experimental Setup

- **Repositories:** 128 Python repos from GitHub
- **Selection criteria:** >50 tests, >90% pass rate, 1k-50k LOC
- **Generation time:** ~2 weeks on 64-core cluster
- **Validation:** Manual review of 1000 random samples
- **Training:** 32B parameter model on 8x H100 GPUs

### Quality Control

```
Automated Quality Checks:
├── Syntax valid (100% required)
├── At least 1 test fails (100% required)
├── Not all tests fail (<90% fail)
├── Deterministic failures (3 runs identical)
└── Original code passes (sanity check)

Manual Review (sample):
├── Bug is realistic: 82%
├── Issue description clear: 76%
├── Fix difficulty appropriate: 88%
└── Overall quality: Good
```

---

## Key Metrics to Track

When generating synthetic training data:

```json
{
  "generation_stats": {
    "repos_processed": 128,
    "bugs_attempted": 204891,
    "bugs_valid": 51247,
    "success_rate": 0.25,
    "avg_time_per_bug_sec": 12.3
  },
  "quality_metrics": {
    "syntax_valid": 1.0,
    "test_failures_appropriate": 0.96,
    "deterministic": 0.98,
    "manual_review_score": 0.82
  },
  "distribution": {
    "simple_bugs": 0.60,
    "medium_bugs": 0.30,
    "complex_bugs": 0.10,
    "bug_categories": {
      "logic_error": 0.35,
      "edge_case": 0.25,
      "off_by_one": 0.15,
      "null_handling": 0.15,
      "other": 0.10
    }
  }
}
```

---

## Key Takeaways

1. **Tests enable training at scale** — Automated validation replaces human annotation
2. **Synthetic data closes the gap** — 50k examples enable GPT-4-class open models
3. **Diversity matters more than volume** — Better to have varied high-quality bugs
4. **Repos with tests are goldmines** — Each repo can generate 100s of tasks
5. **Training is the future of SWE agents** — Prompting alone has limits

---

## See Also

- `024-thinkrepair.md` — Knowledge pool construction from validated examples
- `028-swe-bench-live.md` — Continuous benchmark updates (complementary to training)
- `021-swe-bench-plus.md` — Test quality determines training signal quality
- `022-chatrepair.md` — Test feedback during repair (similar principle)
- `029-swe-rebench.md` — Decontaminated evaluation of trained agents
