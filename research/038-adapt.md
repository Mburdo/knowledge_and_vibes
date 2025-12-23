# ADaPT: As-Needed Decomposition and Planning with Language Models

**Paper:** ADaPT: As-Needed Decomposition and Planning with Language Models
**URL:** https://arxiv.org/abs/2311.05772
**Date:** November 2023 (v2 April 2024)
**Venue:** arXiv preprint

---

## Summary

ADaPT introduces an **adaptive decomposition policy** for LLM task execution: attempt tasks directly with an Executor LLM, and only invoke a Planner LLM to decompose when the executor fails. This contrasts with rigid "plan-then-execute" (wasteful planning) and pure "ReAct" (no decomposition). The key insight: **defer planning until you have evidence it's needed**.

**Key finding:** ADaPT achieves **+28.3% on ALFWorld**, **+27% on WebShop**, and **+33% on TextCraft** over fixed plan-execute baselines by avoiding unnecessary decomposition and adapting decomposition to actual execution failures.

---

## The Core Problem

### The Decomposition Dilemma

```
┌─────────────────────────────────────────────────────────────────┐
│               THE DECOMPOSITION TRADE-OFF                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  APPROACH 1: Plan Everything Upfront                             │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                                                           │   │
│  │  Task → Plan all subtasks → Execute                       │   │
│  │                                                           │   │
│  │  PROBLEM:                                                 │   │
│  │  ✗ Wasted effort planning easy tasks                      │   │
│  │  ✗ Plans become outdated during execution                 │   │
│  │  ✗ Over-decomposition increases coordination overhead     │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  APPROACH 2: Never Plan (Direct Execution)                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                                                           │   │
│  │  Task → Execute directly                                  │   │
│  │                                                           │   │
│  │  PROBLEM:                                                 │   │
│  │  ✗ Complex tasks overwhelm single execution attempt       │   │
│  │  ✗ No systematic approach to hard problems                │   │
│  │  ✗ Fails on multi-step tasks requiring coordination       │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  THE TENSION:                                                    │
│  - Easy tasks don't need decomposition (planning is waste)       │
│  - Hard tasks require decomposition (no decomposition = fail)    │
│  - But you don't know which is which until you try!              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Key insight:** Task difficulty is often **unknowable a priori**—you only learn if decomposition is needed by attempting execution.

---

## The ADaPT Solution

### As-Needed Decomposition

```
┌─────────────────────────────────────────────────────────────────┐
│                    ADAPT ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Task: "Book a hotel in Paris for next weekend"          │   │
│  └────────────────┬─────────────────────────────────────────┘   │
│                   │                                              │
│                   ▼                                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  EXECUTOR LLM: Attempt task directly                      │   │
│  │  "Book Paris hotel for next weekend"                      │   │
│  └────────────────┬─────────────────────────────────────────┘   │
│                   │                                              │
│                   ├─► SUCCESS? → DONE ✓                         │
│                   │                                              │
│                   └─► FAILURE? → Decompose                       │
│                                  │                               │
│  ┌───────────────────────────────▼──────────────────────────┐   │
│  │  PLANNER LLM: Decompose task based on failure            │   │
│  │  "You tried to book directly but failed.                 │   │
│  │   Decompose into subtasks:                               │   │
│  │   1. Search available hotels                             │   │
│  │   2. Compare prices                                      │   │
│  │   3. Select hotel                                        │   │
│  │   4. Make booking"                                       │   │
│  └────────────────┬─────────────────────────────────────────┘   │
│                   │                                              │
│                   ▼                                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  FOR EACH SUBTASK: Execute (recursive)                    │   │
│  │  ├─ Subtask 1: Execute → Success? Yes → Continue          │   │
│  │  ├─ Subtask 2: Execute → Failure? → Decompose further     │   │
│  │  │               ├─ Subtask 2.1: ...                      │   │
│  │  │               └─ Subtask 2.2: ...                      │   │
│  │  ├─ Subtask 3: Execute → Success? Yes → Continue          │   │
│  │  └─ Subtask 4: Execute → Success? Yes → DONE ✓            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  KEY PROPERTIES:                                                 │
│  1. Try before planning (execution-first)                        │
│  2. Decompose only on failure (evidence-based)                   │
│  3. Recursive (subtasks can decompose further)                   │
│  4. Adaptive (plan changes based on execution outcomes)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Detailed Mechanics

### Execution-First Policy

```python
def adapt(task, context):
    """
    ADaPT execution algorithm
    """
    # STEP 1: Try direct execution first
    result = executor_llm.execute(task, context)

    if result.success:
        # Task completed without decomposition
        return result

    # STEP 2: Execution failed → Decompose
    subtasks = planner_llm.decompose(
        task=task,
        failure_context=result.failure_info,
        execution_history=context.history
    )

    # STEP 3: Execute subtasks recursively
    subtask_results = []
    for subtask in subtasks:
        # Recursive: subtask may decompose further
        subtask_result = adapt(subtask, context)
        subtask_results.append(subtask_result)

    # STEP 4: Aggregate subtask results
    return aggregate(task, subtask_results)
```

### Executor LLM

**Role:** Attempt task execution directly.

**Prompt Template:**
```
Task: {task_description}

Context: {available_tools, environment_state}

Attempt to complete this task directly. If you can accomplish it
in one or a few steps, do so. If it seems too complex or you fail,
return a failure signal explaining what went wrong.

Success criteria: {task_success_condition}
```

**Example:**
```
Task: "Book a hotel in Paris for next weekend"

Executor attempts:
1. Query booking API with "Paris next weekend"
2. Select first result
3. Complete booking

Result: FAILURE
Reason: "Ambiguous dates ('next weekend' not resolved),
         no price comparison, no user preference considered"
```

### Planner LLM

**Role:** Decompose failed tasks into subtasks.

**Prompt Template:**
```
Task: {task_description}

Execution attempted but failed:
{failure_context}

Decompose this task into subtasks that address the failure.
Consider:
- What specific sub-problems caused the failure?
- What information is missing?
- What steps need to happen sequentially?

Return subtasks as a list.
```

**Example:**
```
Task: "Book a hotel in Paris for next weekend"
Failure: "Ambiguous dates, no price comparison, no preferences"

Decomposition:
1. "Resolve 'next weekend' to specific dates (YYYY-MM-DD)"
2. "Query user preferences (budget, location, amenities)"
3. "Search hotels in Paris for those dates"
4. "Compare hotel options by price and rating"
5. "Present top 3 options to user for selection"
6. "Book selected hotel"
```

---

## Performance Results

### Benchmark Performance

| Benchmark | ReAct (No Decomp) | Plan-Execute | ADaPT | Δ vs. Best Baseline |
|-----------|-------------------|--------------|-------|---------------------|
| **ALFWorld** (household tasks) | 48.2% | 54.6% | **82.9%** | **+28.3%** |
| **WebShop** (e-commerce) | 52.1% | 59.3% | **86.3%** | **+27.0%** |
| **TextCraft** (game solving) | 31.5% | 38.2% | **71.2%** | **+33.0%** |
| **Average** | 43.9% | 50.7% | **80.1%** | **+29.4%** |

### Why ADaPT Outperforms

```
Performance Breakdown:

┌────────────────────────────────────────────────────────────┐
│                                                             │
│  EASY TASKS (40% of benchmark)                              │
│  ├─ ReAct:        ████████████ 75% (direct execution)       │
│  ├─ Plan-Execute: ████████ 50% (over-planning hurts)        │
│  └─ ADaPT:        ████████████ 75% (direct like ReAct)      │
│                                                             │
│  MEDIUM TASKS (35% of benchmark)                            │
│  ├─ ReAct:        ████ 25% (too complex for direct)         │
│  ├─ Plan-Execute: ████████ 55% (planning helps)             │
│  └─ ADaPT:        ████████████ 78% (decomposes on failure)  │
│                                                             │
│  HARD TASKS (25% of benchmark)                              │
│  ├─ ReAct:        █ 5% (no systematic approach)             │
│  ├─ Plan-Execute: ████████ 52% (static plan fragile)        │
│  └─ ADaPT:        ████████████████ 95% (adaptive decomp)    │
│                                                             │
│  OVERALL:                                                   │
│  ReAct:        ████████ 43.9%                               │
│  Plan-Execute: ██████████ 50.7%                             │
│  ADaPT:        ████████████████████ 80.1%                   │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Key finding:** ADaPT matches ReAct on easy tasks (no planning overhead) and beats Plan-Execute on hard tasks (adaptive decomposition).

---

## Practical Implications

### For Knowledge & Vibes

ADaPT validates the "try-then-decompose" principle:

| ADaPT Principle | K&V Implementation | Status |
|-----------------|-------------------|--------|
| **Execute-first** | Attempt bead before decomposing | ✓ Core workflow |
| **Decompose on failure** | After 3 iterations, decompose | ✓ DDI research |
| **Evidence-based planning** | Use test failures to guide sub-beads | ✓ Bead protocol |
| **Recursive decomposition** | Sub-beads can decompose further | ✓ Supported |
| **Adaptive, not static** | Plan adjusts to execution outcomes | ✓ Philosophy |

### Bead Workflow (ADaPT-Aligned)

```markdown
## K&V Bead Execution (ADaPT-Inspired)

PHASE 1: Direct Execution
├─ Receive bead (REQ + AC + context)
├─ Attempt implementation (Executor role)
├─ Run tests
└─ Check outcome

PHASE 2: Outcome-Based Decision
├─ IF all tests pass:
│   └─ Close bead, move to next
│
├─ IF tests fail (iteration 1-2):
│   ├─ Analyze failures
│   ├─ Fix bugs
│   └─ Retry (Executor continues)
│
└─ IF tests still fail (iteration 3):
    └─ TRIGGER: Decomposition

PHASE 3: Adaptive Decomposition (Planner role)
├─ Analyze failure context:
│   ├─ What specific tests failed?
│   ├─ What constraints were discovered?
│   ├─ What assumptions were wrong?
│
├─ Decompose bead based on evidence:
│   ├─ Create sub-beads for failure aspects
│   ├─ Include execution context (what was tried)
│   ├─ Narrow scope (make sub-beads easier)
│
└─ Execute sub-beads (recursive ADaPT)

PHASE 4: Aggregation
├─ All sub-beads complete
├─ Integrate results
├─ Verify original AC
└─ Close parent bead
```

---

## Example: ADaPT in Action

### Scenario

**Bead:** Implement data import pipeline
**AC:** Import customer data from CSV, validate, insert to database

### Execution

**Iteration 1:**
```
Executor attempts:
- Read CSV
- Validate rows
- Insert to DB

Result: FAIL
Error: "CSV has 500K rows, memory overflow"
```

**Iteration 2:**
```
Executor attempts:
- Read CSV in chunks (50K rows)
- Validate chunks
- Insert chunks

Result: FAIL
Error: "Validation takes 30 minutes (timeout)"
```

**Iteration 3:**
```
Executor attempts:
- Read in smaller chunks (10K rows)
- Simplified validation
- Batch insert

Result: FAIL
Error: "Still times out, validation logic is O(n²)"
```

**Decomposition Triggered (After 3 Iterations):**

Planner analyzes failures:
- **Problem 1:** File too large for single-pass processing
- **Problem 2:** Validation algorithm is inefficient
- **Problem 3:** Database inserts not optimized

Planner decomposes:

```markdown
BEAD-100.1: Streaming CSV reader
  AC: Process 500K row CSV in constant memory
  Approach: Use pandas.read_csv(chunksize=1000)
  Dependency: None
  Priority: P0 (blocks others)

BEAD-100.2: Optimized validation
  AC: Validate 500K rows in <60 seconds
  Approach: Vectorized pandas operations, not row-by-row
  Dependency: None
  Priority: P0

BEAD-100.3: Batch database insertion
  AC: Insert 500K rows in <120 seconds
  Approach: Use bulk_insert_mappings, not row-by-row
  Dependency: BEAD-100.1 (needs chunked data)
  Priority: P1

BEAD-100.4: Integration orchestration
  AC: Coordinate streaming, validation, insertion
  Approach: Pipeline pattern (read → validate → insert)
  Dependency: BEAD-100.1, BEAD-100.2, BEAD-100.3
  Priority: P2 (after all others)
```

**Sub-Bead Execution:**

```
BEAD-100.1: Streaming CSV reader
  Iteration 1: ✓ PASS (all AC met)

BEAD-100.2: Optimized validation
  Iteration 1: ✗ FAIL (still slow)
  Iteration 2: ✓ PASS (vectorization helped)

BEAD-100.3: Batch insertion
  Iteration 1: ✓ PASS (bulk_insert works)

BEAD-100.4: Integration orchestration
  Iteration 1: ✓ PASS (pipeline integrates all pieces)

RESULT: Parent BEAD-100 closed (all sub-beads complete)
```

**Lessons:**
- Direct execution failed 3 times (evidence decomposition needed)
- Decomposition targeted specific failure aspects
- Each sub-bead was simpler (higher success rate)
- Recursive: If sub-bead failed 3× it could decompose further

---

## Implementation Checklist

### For Bead Execution
- [x] Attempt bead implementation first (Executor role)
- [x] Run tests after each iteration
- [x] Track iteration count (stop at 3)
- [ ] On 3rd failure: Trigger decomposition (Planner role)
- [ ] Analyze failure context (tests, errors, constraints discovered)
- [ ] Generate sub-beads targeting failure aspects

### For Planner Role
- [ ] Prompt template for decomposition based on failure
- [ ] Include execution context (what was tried, what failed)
- [ ] Generate sub-beads with:
  - Specific failure aspect to address
  - Success criteria (AC)
  - Dependencies between sub-beads
- [ ] Sub-bead granularity: narrow enough to succeed in <3 iterations

### For Recursion
- [ ] Sub-beads can themselves decompose (recursive ADaPT)
- [ ] Track decomposition depth (prevent infinite recursion)
- [ ] Max depth: 3 levels (bead → sub-bead → sub-sub-bead)
- [ ] If depth exceeded: escalate to operator

---

## Key Takeaways

1. **Execute before planning** — Try direct execution first; only decompose on evidence it's needed
2. **Decompose on failure, not upfront** — 3-iteration rule triggers decomposition (not arbitrary decision)
3. **Evidence-based decomposition** — Use execution failures to guide sub-task creation
4. **Recursive, not flat** — Sub-beads can decompose further if they fail
5. **Adaptive, not static** — Plan adjusts to actual execution outcomes, not upfront guesses

---

## Limitations

### Research Scope
- **Interactive benchmarks** — ALFWorld, WebShop, TextCraft (not repo-scale SWE)
- **Success metrics** — Task completion (not code quality or maintainability)
- **Decomposition granularity** — No guidance on optimal sub-task size

### Practical Constraints
- **Decomposition cost** — Analyzing failures and decomposing adds overhead
- **Infinite recursion** — Need stopping condition (max depth, iteration limit)
- **Aggregation complexity** — Combining sub-bead results can be non-trivial

### Open Questions
- **Optimal iteration threshold?** — Is 3 iterations the right trigger for decomposition?
- **Decomposition heuristics?** — Can we predict which failures benefit from decomposition?
- **Parallel execution?** — Can independent sub-beads execute in parallel?

---

## See Also

- `060-debugging-decay-index.md` — Why 3-iteration limit (security degrades after 3 attempts)
- `039-tdag.md` — Dynamic task decomposition and agent generation (similar adaptive principle)
- `040-mapcoder.md` — Multi-agent pipeline for code generation
- `031-alphacodium.md` — Flow engineering with iterative refinement
- `032-tdd-code-generation.md` — Test-driven development for code generation (iteration + tests)
