---
title: RLEF - Reinforcement Learning from Execution Feedback
tags: [reinforcement-learning, execution-feedback, repair, grounding, training, feedback-loop]
protocols: [P6, P8, P11]
---

# RLEF: Reinforcement Learning from Execution Feedback

**Paper:** RLEF: Grounding Code LLMs in Execution Feedback with Reinforcement Learning
**URL:** https://arxiv.org/abs/2410.02089
**Date:** October 2024 (updates in 2025)
**Venue:** arXiv preprint (later conference version)

---

## Summary

RLEF identifies and addresses a critical weakness in code LLMs: **they often ignore execution feedback** in multi-turn repair loops. Even strong models frequently repeat failed approaches, make irrelevant edits, or hallucinate fixes that don't address actual test failures.

**Key innovation:** End-to-end reinforcement learning (PPO) that trains models to meaningfully use execution feedback as a control signal, dramatically improving repair effectiveness with fewer attempts.

---

## The Execution Feedback Problem

### What Should Happen vs. What Actually Happens

```
┌─────────────────────────────────────────────────────────────────┐
│               IDEAL REPAIR LOOP (DOESN'T HAPPEN)                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Attempt 1: Generate code                                        │
│      ├── Run tests                                               │
│      └── Error: "IndexError: list index out of range at line 42" │
│                                                                  │
│  Attempt 2: Fix the specific error                               │
│      ├── Add bounds check at line 42                             │
│      ├── Run tests                                               │
│      └── Success!                                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│            ACTUAL REPAIR LOOP (COMMON FAILURE MODE)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Attempt 1: Generate code                                        │
│      ├── Run tests                                               │
│      └── Error: "IndexError: list index out of range at line 42" │
│                                                                  │
│  Attempt 2: Ignore error location, refactor entire function      │
│      ├── Run tests                                               │
│      └── Error: "IndexError: list index out of range at line 58" │
│           (same bug, different line number)                      │
│                                                                  │
│  Attempt 3: Try the same refactor again with minor changes       │
│      ├── Run tests                                               │
│      └── Error: "IndexError: list index out of range at line 61" │
│           (STILL the same underlying bug)                        │
│                                                                  │
│  Attempt 4: Hallucinate unrelated "fix"                          │
│      ├── Run tests                                               │
│      └── Error: "SyntaxError" (now it's worse)                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Why Models Ignore Feedback

| Cause | Description | Example |
|-------|-------------|---------|
| **Semantic drift** | Each attempt moves further from original intent | Fix logging → refactor auth → rewrite database |
| **Feedback blindness** | Model doesn't parse error messages effectively | Sees "TypeError" but doesn't understand which type |
| **Pattern repetition** | Falls into local minima of similar attempts | Tries 3 variants of same wrong approach |
| **Overcorrection** | Makes broader changes than feedback warrants | One line fails → rewrite entire module |
| **Hallucinated understanding** | Invents explanation not supported by error | "This must be a race condition" (it's a typo) |

---

## The RLEF Solution

### Training with Execution-Based Rewards

```
┌─────────────────────────────────────────────────────────────────┐
│                  RLEF TRAINING FRAMEWORK                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Environment: Code Generation Task                               │
│      │                                                           │
│      ├── State: Problem spec + test results                      │
│      ├── Action: Generate/modify code                            │
│      ├── Observation: Execution feedback                         │
│      └── Reward: Test pass rate (private tests)                  │
│                                                                  │
│  RL Training Loop (PPO):                                         │
│      │                                                           │
│      ├── 1. Model generates code                                 │
│      ├── 2. Run against public tests                             │
│      ├── 3. Provide execution feedback to model                  │
│      ├── 4. Model refines based on feedback                      │
│      ├── 5. Evaluate on private tests (reward signal)            │
│      ├── 6. Update model parameters to maximize reward           │
│      └── 7. Repeat for many episodes                             │
│                                                                  │
│  Result: Model learns which repair strategies actually work      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Mathematical Model

```
Training Objective:

maximize E[R(τ)]
   θ

Where:
  θ     = Model parameters
  τ     = Trajectory (sequence of code + feedback + revisions)
  R(τ)  = Cumulative reward based on private test pass rate

Reward Function:

R(τ) = Σ w_t × pass_rate_t
       t=1..T

Where:
  T            = Number of repair attempts
  pass_rate_t  = Fraction of private tests passing at attempt t
  w_t          = Time discount (prefer faster fixes)

PPO Update (simplified):

θ_{k+1} = θ_k + α × ∇_θ E[min(
    r_θ(a|s) × A^π(s,a),
    clip(r_θ(a|s), 1-ε, 1+ε) × A^π(s,a)
)]

Where:
  r_θ(a|s) = Probability ratio (new policy / old policy)
  A^π(s,a) = Advantage (how much better than average)
  ε        = Clip parameter (prevents large updates)
```

---

## Benchmark Performance

### Reported Results

| Benchmark | Model Size | Baseline | RLEF | Improvement |
|-----------|-----------|----------|------|-------------|
| CodeContests | 8B | 12.4% | 28.7% | +131% relative |
| CodeContests | 70B | 27.6% | 45.2% | +64% relative |
| APPS | 8B | 18.3% | 31.9% | +74% relative |
| APPS | 70B | 32.1% | 48.6% | +51% relative |

### Sample Efficiency

```
Number of Samples Needed to Reach 40% Pass Rate

 10000 ┤
       │  ██████████████████████████████████ Baseline (need ~10k samples)
  5000 ┤  ██████████████████
       │  ██████████████████
  1000 ┤  ███
   500 ┤  ███  ▓▓▓ RLEF (need ~1k samples, 10× reduction)
   100 ┤  ███  ▓▓▓
    50 ┤  ███  ▓▓▓
     0 ┼─────────────────────────────────────
         0%          20%          40%
              Pass@1 Accuracy

RLEF reduces sampling requirements by approximately 10× (reported).
```

---

## Failure Mode Analysis

### Common Pre-RLEF Failure Patterns

| Failure Type | Frequency (Baseline) | Frequency (RLEF) | Reduction |
|--------------|---------------------|------------------|-----------|
| Repeating same error | 34% | 8% | 76% |
| Ignoring error location | 28% | 6% | 79% |
| Overcorrecting (too broad) | 22% | 5% | 77% |
| Irrelevant edits | 19% | 4% | 79% |
| Syntax introduction | 15% | 3% | 80% |

### RLEF Learned Behaviors

```
Effective Repair Strategies (Learned via RL):

1. Error Localization
   Before: "Error somewhere in function"
   After:  "Error at line 42, variable x undefined"

2. Minimal Edits
   Before: Refactor entire function
   After:  Add single line: x = []

3. Targeted Testing
   Before: Re-run all tests
   After:  Focus on failing test case first

4. Progressive Refinement
   Before: Attempt 1 → Attempt 3 (no learning)
   After:  Attempt 1 → Attempt 2 (builds on feedback)

5. Stop Condition Recognition
   Before: Keep trying random changes
   After:  Recognize when approach is fundamentally wrong
```

---

## Integration with Knowledge & Vibes

### Discipline Without Training

Since RLEF is a training method (not directly applicable to deployed models), K&V can adopt its **workflow insights**:

```
┌─────────────────────────────────────────────────────────────────┐
│            RLEF-INSPIRED REPAIR PROTOCOL (K&V)                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Execute and Capture Feedback                            │
│      ├── Run test suite                                          │
│      ├── Parse error messages                                    │
│      ├── Extract specific failure information                    │
│      └── Identify failing test names and locations               │
│                                                                  │
│  Step 2: Require Explicit Error Citation                         │
│      ├── Agent MUST quote specific error message                 │
│      ├── Agent MUST identify error location (file:line)          │
│      └── Agent MUST explain why this error occurred              │
│                                                                  │
│  Step 3: Plan Minimal Fix                                        │
│      ├── What is the smallest change to fix this specific error? │
│      ├── How to verify this fix addresses the root cause?        │
│      └── What other tests might break from this change?          │
│                                                                  │
│  Step 4: Implement and Validate                                  │
│      ├── Make ONLY the planned minimal change                    │
│      ├── Run the specific failing test                           │
│      ├── If pass → Run full suite                                │
│      └── If fail → Analyze NEW feedback (don't repeat)           │
│                                                                  │
│  Step 5: Track Repair History                                    │
│      ├── Log each attempt and its outcome                        │
│      ├── If attempt N similar to attempt N-1 → STOP              │
│      └── If 3 attempts fail → ADaPT decompose                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Bead Template: Evidence-Based Repair

```markdown
## Repair Bead Template

### Current Failure
**Failing Test:** `test_user_auth_flow`
**Error Message:** (MUST be verbatim quote)
```
AssertionError: Expected 200, got 401
  File "tests/test_auth.py", line 87, in test_user_auth_flow
    assert response.status_code == 200
```

**Error Location:** `src/auth.py:142`

### Root Cause Analysis
Why does this error occur?
- (Agent must explain based on code + error, not speculation)

### Proposed Fix
What is the minimal change to fix this specific error?
- (Single focused change, not refactor)

### Validation Plan
1. Run `test_user_auth_flow` (should pass)
2. Run related tests: `test_*_auth_*` (ensure no regressions)
3. Run full suite (final validation)

### Repair History
(Track attempts to prevent repetition)
- Attempt 1: Added auth check → Still 401 (wrong check)
- Attempt 2: Fixed token validation → SUCCESS
```

---

## Code Example: Feedback-Driven Repair

```python
from dataclasses import dataclass
from typing import Optional, List

@dataclass
class ExecutionFeedback:
    """Structured execution feedback."""
    test_name: str
    passed: bool
    error_type: Optional[str]
    error_message: Optional[str]
    error_location: Optional[tuple[str, int]]  # (file, line)
    stack_trace: Optional[str]

@dataclass
class RepairAttempt:
    """Track repair attempts to prevent repetition."""
    attempt_num: int
    change_description: str
    files_modified: List[str]
    feedback: List[ExecutionFeedback]
    outcome: str  # 'pass', 'fail', 'worse'

class RLEFStyleRepair:
    """RLEF-inspired repair loop with feedback grounding."""

    def __init__(self, max_attempts: int = 3):
        self.max_attempts = max_attempts
        self.history: List[RepairAttempt] = []

    def repair(self, code: str, test_suite: str) -> str:
        """
        Attempt to repair code using execution feedback.

        Returns: Fixed code or raises if cannot fix
        """
        for attempt_num in range(1, self.max_attempts + 1):
            # Step 1: Execute and get feedback
            feedback = self.execute_and_capture(code, test_suite)

            if all(f.passed for f in feedback):
                return code  # Success!

            # Step 2: Check for repetition (RLEF insight)
            if self.is_repeating_pattern(feedback):
                raise RepairFailure(
                    f"Repetitive pattern detected at attempt {attempt_num}. "
                    "Need different approach (ADaPT decompose)."
                )

            # Step 3: Require explicit error citation
            error_analysis = self.analyze_errors(feedback)

            # Step 4: Generate targeted fix (minimal change)
            fixed_code = self.generate_minimal_fix(
                code,
                error_analysis,
                previous_attempts=self.history
            )

            # Step 5: Record attempt
            self.history.append(RepairAttempt(
                attempt_num=attempt_num,
                change_description=self.describe_change(code, fixed_code),
                files_modified=self.get_modified_files(code, fixed_code),
                feedback=feedback,
                outcome=self.classify_outcome(code, fixed_code, feedback)
            ))

            code = fixed_code

        # Max attempts reached
        raise RepairFailure(
            f"Failed to fix after {self.max_attempts} attempts. "
            "Likely need ADaPT decomposition."
        )

    def analyze_errors(self, feedback: List[ExecutionFeedback]) -> dict:
        """
        Analyze errors to extract actionable information.

        This is what RLEF trains models to do better.
        """
        failing_tests = [f for f in feedback if not f.passed]

        # Group by error type
        by_type = {}
        for f in failing_tests:
            error_type = f.error_type or "Unknown"
            if error_type not in by_type:
                by_type[error_type] = []
            by_type[error_type].append(f)

        # Extract common patterns
        common_locations = self.find_common_error_locations(failing_tests)

        return {
            'total_failures': len(failing_tests),
            'error_types': by_type,
            'common_locations': common_locations,
            'should_focus_on': self.prioritize_errors(failing_tests)
        }

    def is_repeating_pattern(self, current_feedback: List[ExecutionFeedback]) -> bool:
        """
        Detect if we're repeating the same failed approach.

        RLEF trains models to avoid this; we enforce it explicitly.
        """
        if len(self.history) < 2:
            return False

        # Check if last 2 attempts had similar error patterns
        prev_errors = {f.error_message for f in self.history[-1].feedback if not f.passed}
        curr_errors = {f.error_message for f in current_feedback if not f.passed}

        # If error messages are identical, we're repeating
        if prev_errors == curr_errors:
            return True

        # If error types and locations match, we're likely repeating
        prev_sigs = {(f.error_type, f.error_location) for f in self.history[-1].feedback}
        curr_sigs = {(f.error_type, f.error_location) for f in current_feedback}

        return len(prev_sigs & curr_sigs) / max(len(curr_sigs), 1) > 0.8

    def generate_minimal_fix(self,
                            code: str,
                            error_analysis: dict,
                            previous_attempts: List[RepairAttempt]) -> str:
        """
        Generate a minimal fix targeting specific errors.

        RLEF learns to do this via RL; we enforce it via prompting.
        """
        # Focus on highest-priority error
        target_error = error_analysis['should_focus_on']

        # Build context that emphasizes feedback grounding
        repair_prompt = f"""
Fix ONLY the following specific error:

Error Type: {target_error.error_type}
Error Message: {target_error.error_message}
Error Location: {target_error.error_location}

Requirements:
1. Make the MINIMAL change to fix this specific error
2. Do NOT refactor unrelated code
3. Explain why your change addresses the error message above

Previous attempts (DO NOT REPEAT):
{self.format_previous_attempts(previous_attempts)}

Code to fix:
{code}
"""

        # Call LLM with feedback-grounded prompt
        fixed_code = call_llm_with_prompt(repair_prompt)

        return fixed_code
```

---

## Key Metrics to Track

When implementing RLEF-style repair in K&V:

```json
{
  "bead_id": "bd-401",
  "repair_session": {
    "total_attempts": 2,
    "outcome": "success",
    "first_failure": {
      "error_type": "AssertionError",
      "error_location": "tests/test_auth.py:87"
    },
    "attempts": [
      {
        "num": 1,
        "error_cited": true,
        "change_scope": "minimal",
        "outcome": "fail",
        "new_error": "Same error, different line"
      },
      {
        "num": 2,
        "error_cited": true,
        "change_scope": "minimal",
        "outcome": "pass",
        "tests_passed": "15/15"
      }
    ],
    "efficiency_score": 0.5,  // (max_attempts - actual + 1) / max_attempts
    "feedback_utilization": "high"  // Did agent cite errors?
  }
}
```

---

## Caveats and Limitations

### Training vs. Inference

| Aspect | RLEF (Training) | K&V (Inference) |
|--------|-----------------|-----------------|
| Method | RL on many examples | Prompting + structure |
| Feedback learning | Automatic (gradient descent) | Manual (prompt engineering) |
| Generalization | Learned across tasks | Per-task enforcement |
| Cost | High (training) | Low (inference only) |
| Applicability | Model builders | Model users |

### When Workflow Discipline Isn't Enough

Even with RLEF-inspired protocols, some repairs fail because:
- Error messages are misleading or incorrect
- Root cause is distant from error location
- Multiple interacting bugs (fix one, reveal another)
- Environmental issues (config, dependencies)

In these cases: **ADaPT decompose** rather than iterate.

---

## Key Takeaways

1. **Feedback loops aren't free:** Models don't automatically use execution feedback effectively
2. **Citation is critical:** Require agents to quote specific errors and locations
3. **Minimal edits win:** Broad refactors often miss the actual bug
4. **Track repair history:** Detect and prevent repetitive failed attempts
5. **Stop at 3 attempts:** RLEF validates debugging decay research (3-attempt limit)
6. **Grounding prevents drift:** Every repair must cite new feedback, not rehash old ideas
7. **Workflow > prompting alone:** Structure and enforcement beat "try harder" prompts

---

## See Also

- `041-debatecoder.md` — Test-driven multi-agent debate
- `042-rankef.md` — Selection using execution feedback
- `045-repairagent.md` — Tool-based repair loops
- `060-debugging-decay-index.md` — Why repair attempts decay
- `006-dark-side-self-correction.md` — Self-correction limits
- `022-chatrepair.md` — Conversational repair approaches
- `038-adapt.md` — When to decompose instead of repair
