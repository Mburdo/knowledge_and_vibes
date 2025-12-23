---
title: MapCoder - Multi-Stage Pipeline
tags: [pipeline, retrieval, planning, debugging, multi-stage, code-generation]
protocols: [P3, P8]
---

# MapCoder: Multi-Agent Code Generation Pipeline

**Paper:** MapCoder: Multi-Agent Code Generation for Competitive Problem Solving
**URL:** https://arxiv.org/abs/2405.11403
**Date:** May 2024
**Venue:** ACL 2024 (Long Paper)

---

## Summary

Multi-agent prompting framework that decomposes code generation into specialized stages: Retrieval, Planning, Coding, and Debugging. Each stage is handled by a specialized "agent" (really a specialized prompt configuration). Approximates human programming workflow through explicit pipeline stages.

**Key finding:** Separating code generation into explicit stages achieves **93.9% on HumanEval** and **83.1% on MBPP** with GPT-4, demonstrating that **pipeline structure** (not agent autonomy) drives reliability.

---

## The Core Problem

### Monolithic Code Generation Fails

```
┌─────────────────────────────────────────────────────────────────┐
│           TRADITIONAL SINGLE-PROMPT GENERATION                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problem Description                                             │
│         │                                                        │
│         ▼                                                        │
│  "Write code that solves this problem"                           │
│         │                                                        │
│         ▼                                                        │
│  [ Single LLM Call ]                                             │
│         │                                                        │
│         ▼                                                        │
│  Code (hopefully correct)                                        │
│                                                                  │
│  FAILURE MODES:                                                  │
│  ✗ Misses similar solved problems (no retrieval)                │
│  ✗ Jumps to implementation (no planning)                         │
│  ✗ No systematic debugging (hope it works)                       │
│  ✗ Edge cases overlooked                                         │
│                                                                  │
│  RESULT: ~75-80% success on benchmarks                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Why it fails:** Skips critical steps humans take (research, plan, debug).

---

## The MapCoder Pipeline

### Four-Stage Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    MAPCODER PIPELINE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STAGE 1: RETRIEVAL                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Problem Description                                        │ │
│  │         │                                                   │ │
│  │         ▼                                                   │ │
│  │  Retrieval Agent:                                           │ │
│  │  "Find similar solved problems"                             │ │
│  │         │                                                   │ │
│  │         ├─► Search problem database (vector similarity)     │ │
│  │         ├─► Retrieve top-k similar problems                 │ │
│  │         └─► Extract solution patterns                       │ │
│  │         │                                                   │ │
│  │         ▼                                                   │ │
│  │  Output: Similar problems + solutions                       │ │
│  │                                                             │ │
│  └──────────────────────┬──────────────────────────────────────┘ │
│                         │                                        │
│  STAGE 2: PLANNING                                               │
│  ┌──────────────────────▼─────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Problem + Similar Solutions                                │ │
│  │         │                                                   │ │
│  │         ▼                                                   │ │
│  │  Planning Agent:                                            │ │
│  │  "Draft solution approach"                                  │ │
│  │         │                                                   │ │
│  │         ├─► Analyze problem constraints                     │ │
│  │         ├─► Consider edge cases                             │ │
│  │         ├─► Draft algorithm outline                         │ │
│  │         └─► Identify data structures needed                 │ │
│  │         │                                                   │ │
│  │         ▼                                                   │ │
│  │  Output: Solution plan (pseudocode + approach)              │ │
│  │                                                             │ │
│  └──────────────────────┬──────────────────────────────────────┘ │
│                         │                                        │
│  STAGE 3: CODING                                                 │
│  ┌──────────────────────▼─────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Problem + Plan + Examples                                  │ │
│  │         │                                                   │ │
│  │         ▼                                                   │ │
│  │  Coding Agent:                                              │ │
│  │  "Implement the plan"                                       │ │
│  │         │                                                   │ │
│  │         ├─► Follow pseudocode structure                     │ │
│  │         ├─► Use data structures from plan                   │ │
│  │         ├─► Handle edge cases identified                    │ │
│  │         └─► Write clean, documented code                    │ │
│  │         │                                                   │ │
│  │         ▼                                                   │ │
│  │  Output: Initial implementation                             │ │
│  │                                                             │ │
│  └──────────────────────┬──────────────────────────────────────┘ │
│                         │                                        │
│  STAGE 4: DEBUGGING                                              │
│  ┌──────────────────────▼─────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Code + Tests                                               │ │
│  │         │                                                   │ │
│  │         ▼                                                   │ │
│  │  Run Tests                                                  │ │
│  │         │                                                   │ │
│  │         ├─► Pass? → DONE                                    │ │
│  │         │                                                   │ │
│  │         └─► Fail? → Debugging Agent:                        │ │
│  │                     "Fix failing tests"                     │ │
│  │                     │                                       │ │
│  │                     ├─► Analyze error message               │ │
│  │                     ├─► Identify failing test case          │ │
│  │                     ├─► Generate fix                        │ │
│  │                     ├─► Re-run tests                        │ │
│  │                     └─► Iterate (max 3 times)               │ │
│  │                                                             │ │
│  │  Output: Debugged, passing code                             │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Stage Details

### Stage 1: Retrieval Agent

**Purpose:** Find similar solved problems to inform approach.

**Prompt Template:**
```
Given this problem:
{problem_description}

Find similar problems from this database:
{problem_database}

Return the top 3 most similar problems with their solutions.
Focus on problems with similar:
- Input/output patterns
- Algorithmic approach
- Complexity constraints
```

**Output Example:**
```
Similar Problem 1: "Find longest substring without repeating characters"
  Approach: Sliding window with hash map
  Time: O(n), Space: O(min(n, m))

Similar Problem 2: "Minimum window substring"
  Approach: Two pointers + frequency map
  Time: O(n), Space: O(k) where k = charset size

Similar Problem 3: "Permutation in string"
  Approach: Sliding window + frequency matching
  Time: O(n), Space: O(1) (fixed charset)

Pattern: All use sliding window + hash map for efficient substring operations
```

**Value:** Retrieval provides **patterns** and **proven approaches**, reducing trial-and-error.

### Stage 2: Planning Agent

**Purpose:** Draft solution approach before coding.

**Prompt Template:**
```
Problem: {problem_description}

Tests: {test_cases}

Similar approaches: {retrieved_solutions}

Create a solution plan:
1. Algorithm choice (and why)
2. Data structures needed
3. Edge cases to handle
4. Time/space complexity
5. Step-by-step pseudocode
```

**Output Example:**
```
PROBLEM: Find longest substring without repeating characters

ALGORITHM: Sliding window with hash set

DATA STRUCTURES:
- Set: Track characters in current window
- Two pointers: window_start, window_end

EDGE CASES:
- Empty string → return 0
- Single character → return 1
- All unique characters → return len(string)
- All same character → return 1

COMPLEXITY:
- Time: O(n) - each char visited at most twice
- Space: O(min(n, charset_size))

PSEUDOCODE:
1. Initialize: window_start=0, max_len=0, char_set={}
2. For window_end in range(len(s)):
     a. While s[window_end] in char_set:
          - Remove s[window_start] from set
          - Increment window_start
     b. Add s[window_end] to set
     c. Update max_len = max(max_len, window_end - window_start + 1)
3. Return max_len
```

**Value:** Planning **prevents implementation errors** and **documents intent**.

### Stage 3: Coding Agent

**Purpose:** Implement the plan in code.

**Prompt Template:**
```
Implement this solution plan:
{plan}

Use this programming language: {language}

Follow these style guidelines:
- Clear variable names
- Add comments for complex logic
- Handle edge cases explicitly
- Write clean, readable code

Do NOT deviate from the plan unless you find an error in the plan itself.
```

**Output Example:**
```python
def length_of_longest_substring(s: str) -> int:
    """
    Find length of longest substring without repeating characters.

    Args:
        s: Input string

    Returns:
        Length of longest substring with all unique characters

    Time: O(n), Space: O(min(n, charset_size))
    """
    # Edge case: empty string
    if not s:
        return 0

    window_start = 0
    max_length = 0
    char_set = set()

    # Sliding window: expand window_end, contract window_start as needed
    for window_end in range(len(s)):
        # Contract window until no duplicates
        while s[window_end] in char_set:
            char_set.remove(s[window_start])
            window_start += 1

        # Add current character
        char_set.add(s[window_end])

        # Update max length
        current_length = window_end - window_start + 1
        max_length = max(max_length, current_length)

    return max_length
```

**Value:** Following the plan ensures **implementation matches design**.

### Stage 4: Debugging Agent

**Purpose:** Fix failing tests through systematic debugging.

**Prompt Template:**
```
Code: {generated_code}

Test results:
{test_results}

Failing test:
Input: {failing_input}
Expected: {expected_output}
Actual: {actual_output}
Error: {error_message}

Debug this code:
1. Identify the root cause of the failure
2. Propose a minimal fix
3. Verify the fix doesn't break passing tests

Return: Fixed code + explanation of the fix
```

**Output Example:**
```
ROOT CAUSE:
The code doesn't handle the edge case where the string is a single character.
The while loop never executes, but we still need to add the character to char_set.

FIX:
Move `char_set.add(s[window_end])` before the max_length calculation.

FIXED CODE:
[...code with fix...]

EXPLANATION:
The original code only added characters after the while loop, which meant
for single-character strings, the char_set remained empty and max_length
stayed 0. Moving the add operation before the calculation ensures all
characters are properly tracked.

VERIFICATION:
- Failing test now passes: length_of_longest_substring("a") → 1
- All previously passing tests still pass
```

---

## Performance Results

### Benchmark Performance

| Benchmark | Baseline (Single Prompt) | MapCoder | Improvement |
|-----------|-------------------------|----------|-------------|
| **HumanEval** | 84.1% | **93.9%** | **+9.8%** |
| **MBPP** | 75.2% | **83.1%** | **+7.9%** |
| **APPS** | 18.3% | **22.0%** | **+3.7%** |
| **CodeContests** | 24.1% | **28.5%** | **+4.4%** |
| **xCodeEval** | 39.8% | **45.3%** | **+5.5%** |

### Why Pipeline Beats Single-Shot

| Aspect | Single Prompt | MapCoder Pipeline | Benefit |
|--------|---------------|-------------------|---------|
| **Retrieval** | No | Yes | +8-12% (learns from similar problems) |
| **Planning** | Implicit | Explicit | +5-7% (prevents logic errors) |
| **Debugging** | Hope it works | Systematic | +4-6% (fixes edge cases) |
| **Edge cases** | Often missed | Identified in planning | +3-5% |
| **Documentation** | None | Comments from plan | Maintainability |

---

## Practical Implications

### For Knowledge & Vibes

MapCoder validates the structured bead workflow:

| MapCoder Stage | K&V Phase | Implementation |
|----------------|-----------|----------------|
| **Retrieval** | Pre-bead research | CASS search for similar beads |
| **Planning** | Bead decomposition | ADR + approach selection |
| **Coding** | Bead implementation | Agent generates code from plan |
| **Debugging** | Test-repair loop | 3-iteration rule with test feedback |

### Bead Template (MapCoder-Inspired)

```markdown
## BEAD-101: Implement substring search

### STAGE 1: RETRIEVAL (Automated)
Similar solved beads (from CASS):
- BEAD-085: String pattern matching (KMP algorithm)
- BEAD-092: Sliding window for array problems
- BEAD-078: Two-pointer technique for strings

Patterns identified:
- Use sliding window for substring problems
- Hash map/set for O(1) lookups
- Two pointers for efficiency

### STAGE 2: PLANNING (LLM + Operator Review)
Algorithm: Sliding window with hash set

Data structures:
- Set<char> for current window
- Two indices: start, end

Edge cases:
- Empty string → return 0
- Single char → return 1
- All unique → return len(s)

Pseudocode:
1. Initialize window and set
2. Expand end pointer
3. Contract start if duplicate found
4. Track maximum window size

Complexity: O(n) time, O(min(n, k)) space

OPERATOR CHECKPOINT: Plan approved ✓

### STAGE 3: CODING (Agent Implementation)
[Agent implements following the plan]

### STAGE 4: DEBUGGING (Automated Test Loop)
Iteration 1:
  Tests: 8/10 pass
  Failures: Empty string, single char
  Fix: Add edge case handling

Iteration 2:
  Tests: 10/10 pass ✓

RESULT: Implementation complete
```

---

## Implementation Checklist

### For Retrieval Stage
- [ ] Build CASS database of solved beads with embeddings
- [ ] Implement semantic search (vector similarity)
- [ ] Extract patterns from similar solutions
- [ ] Present top-k similar beads to planning stage

### For Planning Stage
- [ ] Prompt template for plan generation
- [ ] Include: algorithm, data structures, edge cases, pseudocode
- [ ] Operator review checkpoint (approve/reject plan)
- [ ] Store approved plans in CASS for future retrieval

### For Coding Stage
- [ ] Prompt: "Implement this plan" + plan details
- [ ] Include constraints from ADR (frameworks, patterns)
- [ ] Add interfaces (input/output types)
- [ ] Generate code following plan structure

### For Debugging Stage
- [ ] Run tests automatically after coding
- [ ] On failure: feed back test results + error
- [ ] Prompt: "Fix this specific failure"
- [ ] Iterate max 3 times (DDI limit)
- [ ] If still failing: trigger ADaPT decomposition

---

## Key Takeaways

1. **Pipeline beats monolithic** — Explicit stages (retrieve, plan, code, debug) achieve 93.9% vs. 84% baseline
2. **Retrieval adds value** — Finding similar problems provides +8-12% boost
3. **Planning prevents errors** — Explicit planning phase reduces logic mistakes
4. **Debugging is a stage** — Systematic debug loop (not hope) fixes edge cases
5. **Separation of concerns** — Each stage has clear input/output, easier to optimize

---

## Limitations

### Research Scope
- **Function-level problems** — HumanEval, MBPP are single functions, not repo-scale
- **Clear specifications** — Problems have unambiguous requirements and test cases
- **No integration** — Standalone functions, not system integration

### Practical Constraints
- **Sequential stages** — Pipeline is linear, can't backtrack to planning if coding fails
- **Token cost** — 4 LLM calls per problem vs. 1 for baseline
- **Retrieval quality** — Depends on quality of problem database

### Open Questions
- **Optimal stage design?** — Are 4 stages optimal, or should there be more/fewer?
- **Parallel stages?** — Can planning and retrieval happen in parallel?
- **Adaptive pipeline?** — Should stage sequence change based on problem complexity?

---

## See Also

- `031-alphacodium.md` — Flow engineering with test generation (similar multi-stage approach)
- `038-adapt.md` — Adaptive decomposition when stages fail
- `037-requirements-to-code-practices.md` — How practitioners structure prompts
- `039-tdag.md` — Dynamic task decomposition and agent generation
- `059-multi-agent-orchestrator-2025.md` — Multi-agent coordination patterns
