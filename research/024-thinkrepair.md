---
title: ThinkRepair - Self-Directed Program Repair
tags: [knowledge-pool, exemplars, chain-of-thought, retrieval, repair, few-shot]
protocols: [P3, P8]
---

# ThinkRepair: Self-Directed Automated Program Repair

**Paper:** ThinkRepair: Self-Directed Automated Program Repair
**URL:** https://arxiv.org/html/2407.20898v1
**Date:** July 2024
**Venue:** arXiv preprint

---

## Summary

Revolutionary approach that builds a **self-generated knowledge pool** of validated (bug, fix, reasoning) exemplars, then uses retrieval and chain-of-thought prompting for repair. Achieves **~98 fixes on Defects4J v1.2**.

**Key innovation:** Turn weak prompting into strong performance by building a validated exemplar pool through test execution, not hand-crafted templates.

---

## The Knowledge Pool Paradigm

### Traditional Prompting vs ThinkRepair

```
┌─────────────────────────────────────────────────────────────────┐
│              TRADITIONAL PROMPTING (Hand-Crafted)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Engineer: "Fix this bug: {bug_description}"                     │
│       ↓                                                          │
│  Model: [Generates fix]                                          │
│       ↓                                                          │
│  Results: Inconsistent, context-dependent                        │
│                                                                  │
│  Problems:                                                       │
│  • No examples of successful fixes                               │
│  • No chain-of-thought reasoning                                │
│  • Performance depends on prompt wording                         │
│  • Can't improve without manual prompt engineering              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                THINKREPAIR (Knowledge Pool)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PHASE 1: Build Knowledge Pool                                   │
│  ┌────────────────────────────────────────────────┐             │
│  │ For each known bug:                            │             │
│  │   1. Generate fix + CoT reasoning              │             │
│  │   2. Run tests                                 │             │
│  │   3. If pass: Add (bug, fix, CoT) to pool      │             │
│  │   4. If fail: Discard                          │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  PHASE 2: Fix New Bugs                                           │
│  ┌────────────────────────────────────────────────┐             │
│  │ For new bug:                                   │             │
│  │   1. Retrieve relevant examples from pool      │             │
│  │   2. Few-shot prompt with CoT exemplars        │             │
│  │   3. Generate fix with reasoning               │             │
│  │   4. Test feedback loop (limited rounds)       │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  Benefits:                                                       │
│  • Validated examples (tests passed)                             │
│  • Reasoning traces (chain-of-thought)                           │
│  • Retrieval-based adaptation                                    │
│  • Accumulating knowledge over time                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Two-Phase Workflow

### Phase 1: Knowledge Pool Collection

```
┌─────────────────────────────────────────────────────────────────┐
│                  KNOWLEDGE POOL COLLECTION                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Input: Dataset of known bugs with tests                        │
│         │                                                        │
│         ↓                                                        │
│  For each bug in dataset:                                        │
│  ┌────────────────────────────────────────────────┐             │
│  │ 1. GENERATE fix + reasoning                   │             │
│  │    Prompt: "Fix bug. Explain your reasoning." │             │
│  │         ↓                                      │             │
│  │    Output: Fix code + Chain-of-Thought        │             │
│  │                                                │             │
│  │ 2. VALIDATE fix                                │             │
│  │    Run full test suite                        │             │
│  │         ↓                                      │             │
│  │    Pass? → Add to pool                        │             │
│  │    Fail? → Discard                            │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  Knowledge Pool Entry:                                           │
│  {                                                               │
│    "bug_id": "Defects4J_Math_42",                                │
│    "buggy_code": "<original code>",                              │
│    "fixed_code": "<corrected code>",                             │
│    "reasoning": "The bug occurs because...",                     │
│    "validated": true,                                            │
│    "test_results": "all_passed"                                  │
│  }                                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Phase 2: Fixing with Retrieval

```
┌─────────────────────────────────────────────────────────────────┐
│                      REPAIR NEW BUGS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  New Bug: "NPE in authentication"                                │
│      │                                                           │
│      ↓                                                           │
│  1. RETRIEVE relevant examples from knowledge pool               │
│     ┌─────────────────────────────────────────┐                 │
│     │ Query: Bug description + code context   │                 │
│     │ Method: Semantic similarity (embedding) │                 │
│     │ Result: Top-k most similar fixes        │                 │
│     └─────────────────────────────────────────┘                 │
│      │                                                           │
│      ↓                                                           │
│  2. SELECT diverse examples                                      │
│     ┌─────────────────────────────────────────┐                 │
│     │ Clustering/contrastive selection        │                 │
│     │ Ensure variety in fix strategies        │                 │
│     │ Example count: 3-5 typically            │                 │
│     └─────────────────────────────────────────┘                 │
│      │                                                           │
│      ↓                                                           │
│  3. FEW-SHOT PROMPT with CoT                                     │
│     ┌─────────────────────────────────────────┐                 │
│     │ Example 1: Bug → Reasoning → Fix        │                 │
│     │ Example 2: Bug → Reasoning → Fix        │                 │
│     │ Example 3: Bug → Reasoning → Fix        │                 │
│     │ Now fix: [New bug description]          │                 │
│     └─────────────────────────────────────────┘                 │
│      │                                                           │
│      ↓                                                           │
│  4. GENERATE fix with reasoning                                  │
│      │                                                           │
│      ↓                                                           │
│  5. TEST and iterate (if needed, limited rounds)                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Chain-of-Thought Reasoning Structure

### Example Knowledge Pool Entry

```markdown
## Bug: Defects4J_Math_42

### Buggy Code:
```java
public double evaluate(double x) {
    return numerator.value(x) / denominator.value(x);
}
```

### Reasoning:
1. **Problem identification:** Division by zero when denominator evaluates to 0
2. **Root cause:** No check for zero denominator before division
3. **Impact:** ArithmeticException thrown, crashes program
4. **Fix strategy:** Add guard clause to check denominator
5. **Edge cases:** Also need to handle NaN and infinity results

### Fixed Code:
```java
public double evaluate(double x) {
    double denom = denominator.value(x);
    if (denom == 0.0) {
        return Double.NaN;
    }
    return numerator.value(x) / denom;
}
```

### Validation:
- All tests passed
- Edge cases covered: denom=0, denom=NaN, denom=infinity
- Behavioral preservation: Non-zero cases unchanged
```

---

## Retrieval and Selection Strategy

### Semantic Similarity Retrieval

```python
def retrieve_examples(new_bug, knowledge_pool, k=10):
    """
    Retrieve most relevant examples from knowledge pool.
    """
    # Embed new bug
    new_bug_embedding = embed(new_bug.description + new_bug.code)

    # Compute similarity to all pool entries
    similarities = []
    for entry in knowledge_pool:
        entry_embedding = embed(entry.bug_description + entry.buggy_code)
        sim = cosine_similarity(new_bug_embedding, entry_embedding)
        similarities.append((sim, entry))

    # Return top-k most similar
    return sorted(similarities, reverse=True)[:k]
```

### Contrastive Diversity Selection

```python
def select_diverse_examples(candidates, n=5):
    """
    Select n diverse examples from candidates.
    Uses clustering to ensure variety in fix strategies.
    """
    # Embed fix strategies
    fix_embeddings = [embed(c.fixed_code + c.reasoning) for c in candidates]

    # Cluster into n groups
    clusters = kmeans_clustering(fix_embeddings, n_clusters=n)

    # Select one example from each cluster
    diverse_examples = []
    for cluster in clusters:
        # Pick most confident/central example from cluster
        exemplar = select_cluster_center(cluster)
        diverse_examples.append(exemplar)

    return diverse_examples
```

---

## Reported Results

### Performance Metrics

```
Defects4J v1.2 Benchmark:

ThinkRepair Results:
┌──────────────────────────────────────┐
│ Bugs fixed:     ~98                  │
│ Success rate:   ~25% (of 395 bugs)   │
│ Improvement:    +15-20% over SOTA    │
│ Knowledge pool: ~300 validated fixes │
└──────────────────────────────────────┘

Comparison to baselines:
- Standard prompting:        ~50-60 fixes
- Generate-and-validate:     ~70-75 fixes
- ChatRepair (iterative):    ~75-80 fixes
- ThinkRepair (knowledge pool): ~98 fixes
```

### Ablation Studies

| Configuration | Success Rate | Key Factor |
|---------------|--------------|------------|
| Zero-shot | 15% | Baseline |
| Random few-shot (no CoT) | 18% | Examples help slightly |
| Retrieved few-shot (no CoT) | 21% | Relevance matters |
| **Retrieved + CoT** | **25%** | Reasoning is critical |

---

## Why Knowledge Pools Work

### Learning from Validated Success

```
Traditional approach:
Bug → Prompt → Fix → Test
       ↑
   Hand-crafted, static

ThinkRepair approach:
Bug → Retrieve validated examples → Fix → Test
              ↑
      Built from successful fixes
      Includes reasoning traces
      Grows over time
```

### The Exemplar Effect

| Aspect | Zero-Shot | ThinkRepair (Exemplars) |
|--------|-----------|------------------------|
| Fix strategy | Must invent from scratch | Shown successful patterns |
| Reasoning | Implicit | Explicit (from CoT) |
| Edge cases | May miss | Learned from examples |
| Confidence | Low | High (similar bugs solved) |

---

## Mathematical Model

### Knowledge Pool Effectiveness

```
Let S = set of successful fixes in pool
Let R(b) = retrieved examples for bug b
Let Q = quality of reasoning in examples

P(fix succeeds | ThinkRepair) = f(|S|, relevance(R(b)), Q)

Where:
- |S| grows over time (accumulating knowledge)
- relevance(R(b)) depends on retrieval quality
- Q depends on CoT depth and clarity

Empirical observations:
- Diminishing returns after ~300 examples
- Retrieval precision matters more than recall
- Reasoning quality has multiplicative effect
```

---

## Integration with K&V Workflow

### Multi-Session Learning

ThinkRepair validates the K&V pattern of accumulating project knowledge:

| K&V Pattern | ThinkRepair Evidence |
|-------------|---------------------|
| Build knowledge over sessions | Knowledge pool grows with each validation |
| Reuse validated patterns | Retrieval-based few-shot |
| Evidence-based exemplars | Only test-passing fixes in pool |
| Reasoning traces as artifacts | CoT becomes retrievable knowledge |

### Bead-Level Exemplar Storage

```markdown
## In K&V System

### Completed Bead as Exemplar

**Bead ID:** bd-123
**Type:** Bug fix
**Status:** Closed

**Problem:**
User authentication fails for null input

**Reasoning:**
1. Root cause: No null check before token generation
2. Impact: NullPointerException crashes auth flow
3. Fix strategy: Add guard clause at function entry
4. Edge cases: null user, null password, both null

**Solution:**
```python
def authenticate(user, password):
    if user is None or password is None:
        return AuthResult.INVALID
    # ... rest of auth logic
```

**Validation:**
- All tests passed
- Security scan passed
- Code review approved

**Metadata:**
- Similar to: bd-087, bd-201
- Pattern: null-check guard clause
- Domain: authentication
```

### Retrieval for New Beads

```python
# When starting new bead
new_bead = get_bead("bd-456")

# Retrieve similar solved beads
similar_beads = retrieve_similar(
    new_bead.description,
    filter_by=["bug_fix", "authentication"],
    limit=5
)

# Use as few-shot examples in planning
planning_prompt = build_prompt(
    new_bead=new_bead,
    examples=similar_beads,
    include_reasoning=True
)
```

---

## Critical Caveats

### What ThinkRepair Doesn't Solve

1. **Benchmark Leakage Risk**
   - Exemplar pools can memorize dataset artifacts
   - May not generalize to truly novel bugs
   - Need separation between pool-building and evaluation data

2. **Test Quality Bottleneck**
   - Still bounded by "passes tests" as correctness
   - Weak tests → wrong fixes in knowledge pool
   - Garbage in pool → garbage retrieved
   - See: `021-swe-bench-plus.md`

3. **Single-Function Focus**
   - Most examples are isolated function fixes
   - Repo-level issues need different organization
   - Multi-file coordination not addressed

4. **Retrieval Quality Dependency**
   - Bad retrieval → irrelevant examples
   - Embedding quality matters
   - Domain mismatch reduces effectiveness

---

## Practical Implications

### When to Build Knowledge Pools

```
Build knowledge pool when:
✓ You have validated (problem, solution) pairs
✓ Tasks have repeated patterns
✓ Domain is consistent
✓ Can run validation (tests)
✓ Multi-session work

Don't build when:
✗ Every task is completely novel
✗ No validation available
✗ Can't afford pool collection phase
✗ One-off tasks only
```

### Designing Effective CoT Templates

```python
COT_TEMPLATE = """
## Problem Analysis
{problem_identification}

## Root Cause
{root_cause_analysis}

## Fix Strategy
{approach_description}

## Edge Cases Considered
{edge_cases}

## Implementation
{code_solution}

## Validation Evidence
{test_results}
"""
```

---

## Implementation Details

### Knowledge Pool Collection

```python
def build_knowledge_pool(bug_dataset, model, max_attempts=3):
    """
    Build validated knowledge pool from bug dataset.
    """
    knowledge_pool = []

    for bug in bug_dataset:
        # Generate fix with CoT reasoning
        prompt = f"""
Fix this bug and explain your reasoning step-by-step.

Buggy code:
{bug.code}

Bug description:
{bug.description}

Provide:
1. Problem analysis
2. Root cause
3. Fix strategy
4. Fixed code
"""

        response = model.generate(prompt)
        fix, reasoning = parse_response(response)

        # Validate fix
        test_result = run_tests(bug, fix)

        if test_result.all_passed:
            knowledge_pool.append({
                "bug_id": bug.id,
                "buggy_code": bug.code,
                "fixed_code": fix,
                "reasoning": reasoning,
                "validated": True,
                "test_results": test_result
            })

    return knowledge_pool
```

### Fixing with Retrieved Examples

```python
def fix_with_knowledge_pool(new_bug, knowledge_pool, model):
    """
    Fix new bug using knowledge pool retrieval.
    """
    # Retrieve relevant examples
    examples = retrieve_examples(new_bug, knowledge_pool, k=10)

    # Select diverse exemplars
    diverse_examples = select_diverse_examples(examples, n=5)

    # Build few-shot prompt
    prompt = build_few_shot_prompt(new_bug, diverse_examples)

    # Generate fix
    fix = model.generate(prompt)

    # Iterative refinement if needed (limited rounds)
    max_rounds = 3
    for round in range(max_rounds):
        test_result = run_tests(new_bug, fix)
        if test_result.all_passed:
            return fix

        # Refine with feedback
        feedback_prompt = f"""
Your previous fix:
{fix}

Test result: FAILED
{test_result.failures}

Revise your fix based on this feedback.
"""
        fix = model.generate(feedback_prompt)

    return fix
```

---

## Research Methodology

### Experimental Setup

- **Dataset:** Defects4J v1.2 (395 bugs), QuixBugs
- **Model:** GPT-3.5/GPT-4 class models
- **Pool size:** ~300 validated examples
- **Retrieval:** Embedding-based cosine similarity
- **Selection:** K-means clustering for diversity

### Why This Matters for Research

- **Reproducible:** Knowledge pool can be shared
- **Analyzable:** Can study what makes good examples
- **Improvable:** Can refine pool over time
- **Transparent:** Retrieval decisions are inspectable

---

## Key Metrics to Track

When building knowledge pools:

```json
{
  "pool_stats": {
    "total_entries": 300,
    "success_rate_building": 0.75,
    "avg_reasoning_length": 250,
    "domains": ["auth", "parsing", "math", "io"]
  },
  "repair_stats": {
    "bugs_attempted": 95,
    "bugs_fixed": 68,
    "avg_examples_used": 4.2,
    "avg_retrieval_time_ms": 150
  },
  "quality_metrics": {
    "exemplar_relevance_score": 0.82,
    "diversity_score": 0.71,
    "reasoning_completeness": 0.88
  }
}
```

---

## Key Takeaways

1. **Validated exemplars beat hand-crafted prompts** — Test-passing examples are gold
2. **Reasoning traces are retrievable knowledge** — CoT becomes part of memory
3. **Retrieval + diversity is critical** — Both relevance and variety matter
4. **Knowledge accumulates over time** — Pool grows with each validation
5. **Still bounded by test quality** — Weak tests poison the knowledge pool

---

## See Also

- `022-chatrepair.md` — Iterative repair with test feedback
- `021-swe-bench-plus.md` — Test quality determines pool quality
- `026-flames.md` — Test-guided search for repair
- `027-swe-smith.md` — Synthetic data generation for training
- `065-confucius-code-agent.md` — Session state persistence and memory
