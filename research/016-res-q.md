---
title: RES-Q - Repository-Scale Benchmark
tags: [benchmark, repository-scale, multi-file, ambiguity, evaluation]
protocols: [P6, P8]
---

# RES-Q: Repository-Scale Evaluation Benchmark

**Paper:** RES-Q: Evaluating Code-Editing Large Language Model Systems at the Repository Scale
**URL:** https://arxiv.org/html/2406.16801v2
**Date:** June 2024
**Venue:** arXiv preprint

---

## Summary

Critical benchmark research addressing the gap between toy coding tasks and real repository-level editing. RES-Q provides 100 carefully curated tasks derived from real GitHub commits, each requiring multi-file edits from vague, realistic instructions.

**Key insight:** Benchmarks must measure ambiguity resolution + repo navigation + coherent multi-file changes, not just code generation.

---

## The Benchmark Gap

### What Existing Benchmarks Miss

```
┌─────────────────────────────────────────────────────────────────┐
│              BENCHMARK COMPARISON                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  HumanEval (164 tasks):                                         │
│  ├─ Scope: Single function                                      │
│  ├─ Input: Precise docstring                                    │
│  ├─ Output: Function implementation                             │
│  └─ Skills: Code generation from spec                           │
│                                                                  │
│  SWE-bench (2,294 tasks):                                       │
│  ├─ Scope: Real GitHub issues                                   │
│  ├─ Input: Issue description                                    │
│  ├─ Output: Patch that passes tests                             │
│  └─ Skills: Bug localization + repair                           │
│                                                                  │
│  RES-Q (100 tasks):                                             │
│  ├─ Scope: Repository-level feature/refactor                    │
│  ├─ Input: Vague instruction ("add logging")                    │
│  ├─ Output: Multi-file coherent changes                         │
│  └─ Skills: Ambiguity resolution + planning + editing           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Saturation Problem

```
HumanEval Performance Over Time:

100% ┤
     │                          ●  GPT-4 (90%)
 80% ┤                    ●  Claude 3.5 (92%)
     │              ●  GPT-3.5 (72%)
 60% ┤         ●  CodeGen (58%)
     │    ●  GPT-3 (38%)
 40% ┤
     │
 20% ┤
     │
  0% ┼────────────────────────────────────────
    2021  2022  2023  2024  2025

Problem: Can't differentiate modern models (all > 85%)

RES-Q Performance:

100% ┤
     │
 80% ┤
     │
 60% ┤
     │              ●  Claude 3.5 (58%)
 40% ┤        ●  GPT-4o (46%)
     │   ●  GPT-4 (34%)
 20% ┤
     │
  0% ┼────────────────────────────────────────
    2024  2025

Result: Clear differentiation, room to grow
```

---

## RES-Q Task Design

### Task Construction Process

```
┌─────────────────────────────────────────────────────────────────┐
│                RES-Q TASK CONSTRUCTION                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Commit Selection                                       │
│  ├─ Mine GitHub for high-quality repos                          │
│  ├─ Filter commits:                                             │
│  │  ├─ 2-5 files changed                                        │
│  │  ├─ 50-300 lines changed                                     │
│  │  ├─ Has comprehensive tests                                  │
│  │  └─ Self-contained (not part of larger feature)             │
│  └─ Result: 500 candidate commits                               │
│                                                                  │
│  Step 2: Human Annotation                                       │
│  ├─ Developers write "lazy instruction"                         │
│  │  (What you'd say to colleague, not spec)                    │
│  ├─ Examples:                                                    │
│  │  ├─ "Add logging to the API handlers"                       │
│  │  ├─ "Make database connection pool configurable"            │
│  │  └─ "Refactor user authentication to use JWT"               │
│  ├─ Write test suite that validates change                      │
│  └─ Document expected files to modify                           │
│                                                                  │
│  Step 3: Quality Control                                        │
│  ├─ Independent developers verify:                              │
│  │  ├─ Instruction is realistic (vague but achievable)         │
│  │  ├─ Tests are comprehensive                                 │
│  │  └─ Ground truth is correct                                 │
│  └─ Result: 100 curated tasks                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Task Characteristics

| Property | Value | Rationale |
|----------|-------|-----------|
| **Tasks** | 100 | Manageable for thorough evaluation |
| **Files per task** | 2-5 | Multi-file but focused |
| **Lines changed** | 50-300 | Substantive but not massive |
| **Instruction length** | 10-50 words | Realistic vagueness |
| **Languages** | Python (70%), JavaScript (30%) | Popular, well-tested |
| **Test coverage** | > 80% | Strong validation signal |

---

## Example RES-Q Task

### Task #42: Add Request Logging

```
┌─────────────────────────────────────────────────────────────────┐
│                       RES-Q TASK #42                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INSTRUCTION (what model receives):                             │
│  "Add structured logging to all API request handlers.           │
│   Log request method, path, status code, and duration."         │
│                                                                  │
│  REPOSITORY STATE:                                              │
│  ├─ app/                                                        │
│  │  ├─ api/                                                     │
│  │  │  ├─ users.py (UserHandler, ProfileHandler)               │
│  │  │  ├─ posts.py (PostHandler, CommentHandler)               │
│  │  │  └─ __init__.py                                          │
│  │  ├─ middleware/                                              │
│  │  │  └─ auth.py                                              │
│  │  └─ utils/                                                   │
│  │     └─ logging.py (basic setup, no structured logging)      │
│  └─ tests/                                                      │
│     ├─ test_api.py                                             │
│     └─ test_logging.py                                         │
│                                                                  │
│  EXPECTED CHANGES:                                              │
│  ├─ app/utils/logging.py                                        │
│  │  └─ Add structured logger setup                             │
│  ├─ app/api/users.py                                            │
│  │  └─ Add logging calls to all handlers                       │
│  ├─ app/api/posts.py                                            │
│  │  └─ Add logging calls to all handlers                       │
│  └─ tests/test_logging.py                                       │
│     └─ Add tests for structured logging                        │
│                                                                  │
│  VALIDATION:                                                     │
│  ├─ All tests pass                                             │
│  ├─ Logs contain required fields (method, path, status, time)  │
│  ├─ Logs are JSON-formatted                                    │
│  └─ No duplicate logging (avoid double-logging)                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Challenges in This Task

```
1. AMBIGUITY RESOLUTION
   ├─ "Structured logging" → Must infer JSON format
   ├─ "All handlers" → Must discover which files have handlers
   └─ "Duration" → Must add timing logic

2. NAVIGATION
   ├─ Find all API handler files
   ├─ Identify which functions are handlers
   └─ Locate logging utility module

3. COHERENT MULTI-FILE EDITS
   ├─ Consistent logging format across files
   ├─ Shared logging utility (DRY)
   └─ Tests that validate all handlers

4. EDGE CASES
   ├─ Don't log sensitive data (passwords, tokens)
   ├─ Handle exceptions during logging
   └─ Avoid performance impact
```

---

## Evaluation Metrics

### Pass@k Definition

```
Pass@k: Probability that at least one of k attempts succeeds

For RES-Q:
├─ Generate k candidate patches (k=1, 3, 5, 10)
├─ Apply each patch to clean repo
├─ Run test suite for each
├─ Success if ANY patch passes all tests
└─ Report pass@k percentage

Example:
├─ Task has 100 instances
├─ Generate k=3 patches per instance
├─ 58 instances have ≥1 passing patch
└─ Pass@3 = 58%
```

### Beyond Binary Pass/Fail

```
Additional Metrics (not always reported):

1. File Coverage
   ├─ % of expected files modified
   ├─ Precision: Modified files that should be
   └─ Recall: Expected files that were modified

2. Test Coverage
   ├─ % of tests that pass
   ├─ New tests added when appropriate
   └─ No regressions in existing tests

3. Code Quality
   ├─ Maintains style consistency
   ├─ No linter errors introduced
   └─ Follows repo patterns

4. Instruction Adherence
   ├─ Manual review: Did it do what was asked?
   ├─ No over-engineering
   └─ No under-delivery
```

---

## Performance Results

### Model Comparison (Pass@1)

```
┌────────────────────────────────────────────────────────────────┐
│         RES-Q LEADERBOARD (June 2024)                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Claude 3.5 Sonnet:        58%  ██████████████████████████████│
│  GPT-4o:                   46%  ███████████████████████        │
│  Claude 3 Opus:            42%  █████████████████████          │
│  GPT-4 Turbo:              34%  █████████████████              │
│  GPT-3.5 Turbo:            18%  █████████                      │
│  Gemini 1.5 Pro:           28%  ██████████████                 │
│  Open-source best:         12%  ██████                         │
│                                                                │
└────────────────────────────────────────────────────────────────┘

Key finding: 12% gap between #1 and #2 (Claude 3.5 vs GPT-4o)
HumanEval gap: Only 2% between top models
```

### Context Window Impact

```
Effect of Context Window Size on Performance:

Pass@1
  │
60├─────────────●  Full context (128K)
  │
50├────────●  8K context
  │
40├───●  4K context
  │
30├●  2K context
  │
20├
  │
10├
  │
 0└────────────────────────────────────
  2K   4K    8K    Full

Observation:
├─ Larger context helps significantly
├─ But even with full context, tasks are hard
└─ Suggests planning/navigation problems, not just capacity
```

---

## Analysis: Why RES-Q is Hard

### Failure Mode Analysis

```
Manual Analysis of 100 Failures (Claude 3.5):

┌────────────────────────────────┬────────┬────────┐
│     Failure Mode               │ Count  │   %    │
├────────────────────────────────┼────────┼────────┤
│ Wrong files modified           │   28   │  28%   │
│ Incomplete (missed files)      │   22   │  22%   │
│ Over-modification (too broad)  │   15   │  15%   │
│ Test failures (wrong logic)    │   18   │  18%   │
│ Ambiguity misresolved          │   12   │  12%   │
│ Style/pattern mismatch         │    5   │   5%   │
└────────────────────────────────┴────────┴────────┘

Biggest issue: Navigation and scoping (65% of failures)
```

### Example Failure

```
Task: "Add caching to database queries"

Model Output:
├─ Modified: app/database/connection.py (✓ correct)
├─ Modified: app/database/queries.py (✓ correct)
├─ Modified: app/api/users.py (✗ over-modification)
├─ Modified: app/api/posts.py (✗ over-modification)
└─ Missing: tests/test_cache.py (✗ incomplete)

Failure: Test suite fails (cache not thread-safe)

Root Cause:
├─ Correctly identified database layer needs caching
├─ Incorrectly modified API layer (unnecessary)
├─ Missed that caching requires new tests
└─ Implementation has concurrency bug

Correct Approach:
├─ Add cache utility in app/utils/cache.py
├─ Modify database layer to use cache
├─ Add comprehensive cache tests
└─ Don't touch API layer (abstraction boundary)
```

---

## Integration with K&V Workflow

### Using RES-Q for Validation

```yaml
# RES-Q-style validation in K&V

# After completing a feature bead chain
- id: bd-validation
  title: "Validate feature completeness"
  phase: verification
  criteria:
    file_coverage:
      expected_files: [file1.py, file2.py, tests/test_feature.py]
      precision: "> 0.8"  # 80% of modified files should be expected
      recall: "> 0.9"     # 90% of expected files should be modified

    test_coverage:
      all_tests_pass: true
      new_tests_added: true
      no_regressions: true

    instruction_adherence:
      manual_review_required: true
      criteria: "Does solution match vague instruction?"
```

### Ambiguity Resolution Protocol

```python
# From RES-Q: Always clarify before executing

def handle_vague_instruction(instruction, repo):
    """Handle vague instructions RES-Q style."""

    # Identify ambiguities
    ambiguities = identify_ambiguities(instruction)

    if ambiguities:
        # Generate clarifying questions
        questions = []
        for ambiguity in ambiguities:
            questions.append(f"Q: {ambiguity.question}")
            questions.append(f"Options: {ambiguity.options}")
            questions.append(f"Recommendation: {ambiguity.recommendation}")

        # Present to user or make informed decision
        resolution = resolve_ambiguities(questions, repo_context=repo)

        # Update instruction with resolved ambiguities
        instruction = apply_resolution(instruction, resolution)

    return instruction

# Example:
# Input: "Add logging to API handlers"
# Ambiguities detected:
# - What format? (JSON, text, custom)
# - Which handlers? (all, public only, specific module)
# - What to log? (request only, request + response, timing)
# Resolution:
# - Format: JSON (standard for structured logging)
# - Handlers: All in app/api/ directory
# - Content: Request method, path, status, duration
```

---

## Key Takeaways

1. **Repository-scale tasks are qualitatively different** — Not just "harder code generation"
2. **Ambiguity resolution matters** — Vague instructions are realistic and challenging
3. **Navigation is a bottleneck** — Finding right files/functions is hard
4. **Multi-file coherence is hard** — Consistent changes across files require planning
5. **System evaluation, not just models** — Scaffolding and tools matter enormously
6. **Small benchmarks can be valuable** — 100 well-crafted tasks > 10,000 noisy ones
7. **Room for improvement** — Even best models only pass 58% of tasks

---

## See Also

- `011-agentless.md` — Localization and validation strategies
- `012-autocoderover.md` — Structure-aware navigation
- `017-context-retrieval-repo-editing.md` — Context sufficiency research
- `018-livecodebench.md` — Contamination-free evaluation
- `065-confucius-code-agent.md` — Scaling to large repositories
