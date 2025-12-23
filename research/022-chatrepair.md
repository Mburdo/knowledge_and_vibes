---
title: ChatRepair - Conversation-Driven Repair
tags: [repair, debugging, conversation, iteration, feedback, testing]
protocols: [P6, P11]
---

# ChatRepair: Conversation-Driven Program Repair

**Paper:** Automated Program Repair via Conversation: Fixing 162 out of 337 Bugs for $0.42 Each using ChatGPT
**URL:** https://lingming.cs.illinois.edu/publications/issta2024.pdf
**Date:** ISSTA 2024
**Venue:** ISSTA 2024

---

## Summary

Revolutionary approach to automated program repair that treats debugging as a **conversation** rather than one-shot generation. Achieves **162/337 fixes (48%)** on Defects4J at **$0.42 per fix**.

**Key innovation:** Multi-step interaction with execution feedback beats independent sampling by grounding "reflection" in **concrete test results** rather than abstract self-critique.

---

## The Conversation Paradigm

### Traditional APR vs ChatRepair

```
┌─────────────────────────────────────────────────────────────────┐
│              TRADITIONAL APR (Generate-and-Validate)             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Generate patch → Run tests → Accept/Reject                     │
│       ↓              ↓            ↓                              │
│   Independent    Binary       No learning                       │
│    samples       result                                         │
│                                                                  │
│  Problems:                                                       │
│  • Each attempt is independent (no learning)                     │
│  • Wastes context on duplicate errors                           │
│  • Success depends on lucky sampling                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                CHATREPAIR (Conversational)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Round 1: Generate patch                                         │
│      ↓                                                           │
│  Run tests → Failed at line 42: NullPointerException            │
│      ↓                                                           │
│  Round 2: "Your patch failed here's why... revise"              │
│      ↓                                                           │
│  Generate improved patch                                         │
│      ↓                                                           │
│  Run tests → Passed! Now generate variations                    │
│      ↓                                                           │
│  Round 3-5: Local search around success                         │
│                                                                  │
│  Benefits:                                                       │
│  • Model learns from failures                                    │
│  • Concrete execution signals guide repair                       │
│  • Local search exploits successful directions                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## The ChatRepair Algorithm

### Three-Phase Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHATREPAIR WORKFLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: INITIAL REPAIR (max N rounds)                         │
│  ┌────────────────────────────────────────────────┐             │
│  │ Loop until plausible patch or max rounds:      │             │
│  │   1. Generate patch                            │             │
│  │   2. Run failing test                          │             │
│  │   3. If still fails:                           │             │
│  │      - Extract failure info (stack, message)   │             │
│  │      - Feed back to model with previous patch  │             │
│  │      - Request revision                        │             │
│  │   4. If passes: → Phase 2                      │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  Phase 2: VALIDATION                                             │
│  ┌────────────────────────────────────────────────┐             │
│  │ Run full test suite on plausible patch         │             │
│  │ If all tests pass: → Phase 3                   │             │
│  │ If new failures: → Back to Phase 1 with info   │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  Phase 3: LOCAL SEARCH (optional)                                │
│  ┌────────────────────────────────────────────────┐             │
│  │ "Generate M variations of successful patch"    │             │
│  │ Test each variation                            │             │
│  │ Keep all that pass (diversity for ranking)     │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Conversation Structure

```
User: Here's the buggy code and failing test.
      Fix the bug.

Agent: [Generates Patch v1]

User: Your patch failed with:
      - Test: testAuthentication
      - Line: 42
      - Error: NullPointerException
      - Stack: auth.validate(null) at line 42

      Here's your previous attempt:
      [Shows Patch v1]

      Revise the patch to fix this failure.

Agent: [Generates Patch v2, addressing NPE]

User: Tests pass! Now generate 3 variations of this fix.

Agent: [Generates variations exploring different approaches]
```

---

## Critical Information: What ChatRepair Feeds Back

### Feedback Components

ChatRepair explicitly includes **bug-exposing test information** that simpler prompting baselines often omit:

| Information | Example | Why It Matters |
|-------------|---------|----------------|
| **Failing test name** | `testAuthentication` | Identifies context/scenario |
| **Failing line** | Line 42 | Precise localization |
| **Stack trace** | `at auth.validate()` | Call chain context |
| **Error message** | `NullPointerException` | Failure mode |
| **Previous patch** | The code that failed | Learn from mistakes |

### Example Feedback Prompt

```
Your previous patch:
---
def authenticate(user, password):
    token = generate_token(user)  # Bug: user can be None
    return validate(token)
---

Test result: FAILED
Test: test_null_user
Error: NullPointerException at line 2
Stack: generate_token(None) → token.encode() → NPE

Revise your patch to handle this failure.
```

---

## Reported Results

### Performance Metrics

```
Defects4J Benchmark:
337 bugs total

ChatRepair Results:
┌──────────────────────────────────┐
│ Correct fixes:    162 (48.1%)    │
│ Plausible fixes:  189 (56.1%)    │
│ Average cost:     $0.42 per fix  │
│ Average rounds:   2.3 iterations │
└──────────────────────────────────┘

Comparison to baselines:
- Standard prompting: ~25-30%
- Generate-and-validate (100 samples): ~35-40%
- ChatRepair (conversational): 48.1%
```

### Cost Efficiency

| Approach | Success Rate | Cost per Fix | Cost per Attempt |
|----------|--------------|--------------|------------------|
| One-shot prompting | 25% | $0.60 | $0.15 |
| 100 independent samples | 40% | $8.50 | $3.40 |
| **ChatRepair** | **48%** | **$0.42** | **$0.20** |

Efficiency gain: **20x cheaper than brute-force sampling** with better results.

---

## Why Conversation Beats One-Shot

### Learning from Failures

```
Attempt 1 (Independent):
Bug: Division by zero
Fix: Add check `if x != 0`
Test: Still fails (x is float, NaN case not handled)
Result: Rejected, no learning

Attempt 2 (Independent):
Bug: Division by zero
Fix: Add check `if x > 0`  # Still wrong!
Test: Still fails
Result: Wasted token budget
```

vs

```
Round 1 (Conversational):
Bug: Division by zero
Fix: Add check `if x != 0`
Test: Still fails - NaN case
Feedback: "Your fix doesn't handle NaN. Here's the failure..."

Round 2 (Conversational):
Fix: Add check `if x != 0 and not math.isnan(x)`
Test: Passes
Result: Learned from concrete failure
```

### The Grounding Effect

ChatRepair demonstrates a critical principle:

> **Reflection is most useful when grounded in concrete execution signals, not abstract self-critique.**

| Reflection Type | Example | Effectiveness |
|-----------------|---------|---------------|
| Abstract self-critique | "Let me think harder..." | Low |
| Execution-grounded | "Test failed at line 42 with NPE" | High |
| Comparative | "This patch vs previous patch" | High |

---

## Mathematical Model

### Expected Success Rate

```
Let p = probability of success per attempt
Let n = number of attempts

Independent sampling:
  P(success) = 1 - (1-p)^n
  Requires n independent context windows

Conversational repair:
  P(success) ≈ p + (1-p) × q
  Where q = probability of successful revision given feedback
  Shares single context window

If q > p/2, conversational is more efficient
Research shows q ≈ 0.6-0.7 for ChatRepair
```

---

## Integration with K&V Workflow

### Calibration as Post-Execution

ChatRepair validates the K&V calibration model:

| K&V Pattern | ChatRepair Evidence |
|-------------|---------------------|
| Calibration after execution | Feedback follows test runs |
| Evidence-based revision | Concrete failures guide fixes |
| Iteration with learning | Each round improves on previous |
| Bounded attempts | Max rounds prevent infinite loops |

### ADaPT + ChatRepair Pattern

```
Task Bead: "Fix authentication bug"
    │
    ├── Round 1: Generate initial patch
    │   └── Test: Failed (NPE)
    │
    ├── Round 2: Revise with failure info
    │   └── Test: Failed (edge case)
    │
    ├── Round 3: Alternative approach
    │   └── Test: Passed!
    │
    └── CALIBRATION:
        ├── Full test suite: All pass
        ├── Generate variations (local search)
        └── Select best variant

If Round 3 fails → ADaPT decompose
(Matches DDI 3-attempt rule)
```

### Encoding Failure Feedback as Artifacts

```markdown
## In K&V Planning System

### Test Feedback Artifact (Standard Template)

**Test Run:** attempt-2
**Status:** FAILED
**Test:** test_user_authentication
**Location:** Line 42 in auth.py
**Error:** NullPointerException
**Stack:**
  auth.validate(token) at line 42
  token.encode() at line 8

**Previous Patch:**
```python
def authenticate(user):
    token = generate_token(user)
    return validate(token)
```

**Next Action:** Revise to handle null user input
```

---

## Implementation Details

### Prompt Structure for Revision

```python
def build_revision_prompt(bug, previous_patch, test_failure):
    return f"""
You attempted to fix this bug:
{bug.description}

Your previous patch:
{previous_patch}

The patch failed with:
Test: {test_failure.test_name}
Error: {test_failure.error_message}
Stack trace:
{test_failure.stack_trace}

Revise your patch to fix this failure while maintaining the fix for the original bug.
"""
```

### Local Search After Success

```python
def local_search(successful_patch, num_variations=5):
    prompt = f"""
You successfully fixed the bug with this patch:
{successful_patch}

Generate {num_variations} alternative implementations that:
1. Also fix the bug
2. Use different approaches
3. May be more elegant or efficient

Each variation should be a complete, working patch.
"""
    return generate_variations(prompt)
```

---

## Critical Caveats

### What ChatRepair Doesn't Solve

1. **Tests Define Correctness**
   - Passing tests ≠ correct fix
   - Plausible vs correct patch problem
   - Weak tests mislead conversation
   - See: `021-swe-bench-plus.md`

2. **Repair Scope Limitations**
   - Most benchmarks are single-function
   - Repo-level issues are harder
   - Multi-file edits need coordination
   - Environment setup not addressed

3. **Cost/Pricing Drift**
   - Results depend on model version
   - Token costs change over time
   - Prompt length affects pricing
   - Different models have different sweet spots

4. **Debugging Decay Still Applies**
   - Even with feedback, effectiveness decays
   - 3-5 rounds practical maximum
   - See: `060-debugging-decay-index.md`

---

## Practical Implications

### When to Use Conversational Repair

```
Use ChatRepair pattern when:
✓ Tests provide good signal
✓ Failures are informative
✓ Bugs are localized
✓ Context fits in window
✓ Cost efficiency matters

Don't use when:
✗ Tests are weak/sparse
✗ Errors are cryptic
✗ Multi-file coordination needed
✗ Root cause is unclear
✗ Already 3+ attempts failed (use ADaPT)
```

### Designing Test Feedback

```python
# Good feedback (actionable)
{
  "test": "test_authentication",
  "status": "FAILED",
  "location": "auth.py:42",
  "error": "NullPointerException",
  "context": "generate_token(None)",
  "previous_patch": "<code>",
  "suggestion": "Handle null user input"
}

# Poor feedback (not actionable)
{
  "status": "FAILED",
  "error": "Test failed"
}
```

---

## Key Metrics to Track

When implementing conversational repair:

```json
{
  "task_id": "bd-123",
  "approach": "chatrepair",
  "rounds": 3,
  "round_results": [
    {"round": 1, "status": "failed", "error": "NPE at line 42"},
    {"round": 2, "status": "failed", "error": "Edge case"},
    {"round": 3, "status": "passed", "full_suite": true}
  ],
  "variations_generated": 5,
  "final_patch": "variant-2",
  "total_cost": 0.45,
  "success": true
}
```

---

## Research Methodology

### Experimental Setup

- **Dataset:** Defects4J (337 real bugs from Java projects)
- **Model:** ChatGPT (GPT-3.5-turbo at time of publication)
- **Max rounds:** 5 per bug
- **Validation:** Full test suite must pass
- **Comparison:** vs generate-and-validate with 100 samples

### Why Defects4J

- Real bugs from production code
- Comprehensive test suites
- Well-studied benchmark
- Enables comparison to prior work

---

## Key Takeaways

1. **Conversation beats one-shot** — Feedback-driven iteration outperforms independent sampling
2. **Grounding is critical** — Concrete execution signals > abstract reflection
3. **Cost efficiency matters** — 20x cheaper than brute-force with better results
4. **Test quality gates success** — Weak tests mislead conversational repair
5. **Bounded iteration required** — Even with feedback, decay limits effectiveness

---

## See Also

- `060-debugging-decay-index.md` — Why 3-iteration limit applies even with feedback
- `021-swe-bench-plus.md` — Test quality determines repair success
- `024-thinkrepair.md` — Self-directed repair with knowledge pools
- `026-flames.md` — Test-guided semantic search
- `038-adapt.md` — When to decompose instead of iterate