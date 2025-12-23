# RepairAgent: Autonomous Program Repair Agent

**Paper:** RepairAgent: An Autonomous, LLM-Based Agent for Program Repair
**URL:** https://arxiv.org/abs/2403.17134
**Date:** March 2024
**Venue:** ICSE 2025

---

## Summary

RepairAgent treats program repair as an **agentic tool-using loop** rather than a single-shot prompt. It interleaves bug understanding, information gathering, fix generation, and validation through a structured finite-state machine that prevents aimless wandering and ensures systematic progress.

**Key innovation:** Structured tool loops + explicit state machine prevent the random exploration that causes repair failure in unstructured prompting approaches.

---

## The Repair Challenge

### Why Unstructured Repair Fails

```
┌─────────────────────────────────────────────────────────────────┐
│           UNSTRUCTURED REPAIR (COMMON FAILURE MODE)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Single Prompt: "Fix the failing test"                           │
│      │                                                           │
│      ├── Agent guesses at root cause (often wrong)               │
│      ├── Generates fix based on guess                            │
│      ├── May not even run tests                                  │
│      └── If fails, tries random variations                       │
│                                                                  │
│  Problems:                                                       │
│  - No systematic investigation                                   │
│  - Jumps to conclusions without evidence                         │
│  - Doesn't validate assumptions                                  │
│  - Can't track what's been tried                                 │
│  - No clear termination condition                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### RepairAgent Structured Approach

```
┌─────────────────────────────────────────────────────────────────┐
│              REPAIRAGENT (STRUCTURED TOOL LOOP)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  State 1: UNDERSTAND                                             │
│      ├── Read failing test                                       │
│      ├── Extract error message                                   │
│      ├── Identify error type and location                        │
│      └── → Transition to LOCALIZE                                │
│                                                                  │
│  State 2: LOCALIZE                                               │
│      ├── Search for relevant code                                │
│      ├── Read suspect functions/files                            │
│      ├── Trace data flow to error location                       │
│      └── → Transition to GENERATE_FIX                            │
│                                                                  │
│  State 3: GENERATE_FIX                                           │
│      ├── Synthesize patch based on evidence                      │
│      ├── Ensure minimal change                                   │
│      ├── Write fixed code                                        │
│      └── → Transition to VALIDATE                                │
│                                                                  │
│  State 4: VALIDATE                                               │
│      ├── Run failing test                                        │
│      ├── Run full test suite                                     │
│      ├── If pass → DONE                                          │
│      └── If fail → Analyze feedback, LOCALIZE or GENERATE_FIX    │
│                                                                  │
│  Stop Conditions:                                                │
│  - Tests pass (SUCCESS)                                          │
│  - Max iterations reached (FAILURE)                              │
│  - No progress after N attempts (STUCK)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## RepairAgent Architecture

### Finite State Machine

```
                    ┌──────────────┐
                    │   START      │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
            ┌──────▶│  UNDERSTAND  │
            │       └──────┬───────┘
            │              │
            │              ▼
            │       ┌──────────────┐
            │  ┌───▶│   LOCALIZE   │
            │  │    └──────┬───────┘
            │  │           │
            │  │           ▼
            │  │    ┌──────────────┐
            │  │    │ GENERATE_FIX │◀──┐
            │  │    └──────┬───────┘   │
            │  │           │            │
            │  │           ▼            │
            │  │    ┌──────────────┐   │
            │  └────│   VALIDATE   │───┘
            │       └──────┬───────┘
            │              │
            │         Pass │ Fail
            │              ├────────┘
            │              │
            │              ▼
            │       ┌──────────────┐
            └───────│   MAX ITER?  │
                    └──────┬───────┘
                           │
                      Yes  │  No
                    ┌──────┴──────┐
                    ▼             ▼
              ┌─────────┐   ┌─────────┐
              │ FAILURE │   │ SUCCESS │
              └─────────┘   └─────────┘
```

### Tool Inventory

| Tool | Purpose | State Used In |
|------|---------|---------------|
| `read_file` | Read source code | UNDERSTAND, LOCALIZE |
| `search_code` | Find relevant functions | LOCALIZE |
| `run_test` | Execute specific test | UNDERSTAND, VALIDATE |
| `run_all_tests` | Full test suite | VALIDATE |
| `get_stack_trace` | Extract error details | UNDERSTAND |
| `write_file` | Apply fix | GENERATE_FIX |
| `diff` | Compare before/after | VALIDATE |

---

## Benchmark Performance

### Defects4J Results (Reported)

| Metric | Value |
|--------|-------|
| **Total bugs fixed** | 164 |
| **Unique fixes** | 39 (not fixed by prior APR techniques) |
| **Average cost** | ~270k tokens per bug |
| **Average time** | ~15 minutes per bug |
| **Pass@1 rate** | 28.5% |

### Comparison with Other Techniques

```
Bugs Fixed on Defects4J

 200 ┤                        ████ RepairAgent (164)
 180 ┤                        ████
 160 ┤                        ████
 140 ┤                  ████  ████
 120 ┤            ████  ████  ████
 100 ┤      ████  ████  ████  ████
  80 ┤████  ████  ████  ████  ████
  60 ┤████  ████  ████  ████  ████
  40 ┤████  ████  ████  ████  ████
  20 ┤████  ████  ████  ████  ████
   0 ┼──────────────────────────────
     Prior  Single  ChatGPT Repair
     Best   Prompt  Direct  Agent

RepairAgent fixes 39 bugs that no prior technique fixed (reported).
```

---

## Integration with Knowledge & Vibes

### Repair Bead State Machine

```python
from enum import Enum
from dataclasses import dataclass
from typing import Optional, List

class RepairState(Enum):
    """RepairAgent-inspired states."""
    UNDERSTAND = "understand"
    LOCALIZE = "localize"
    GENERATE_FIX = "generate_fix"
    VALIDATE = "validate"
    DONE = "done"
    FAILED = "failed"

@dataclass
class RepairContext:
    """Shared context across repair states."""
    failing_test: str
    error_message: str
    error_location: Optional[tuple[str, int]] = None
    suspect_files: List[str] = None
    fix_attempted: Optional[str] = None
    iteration: int = 0
    max_iterations: int = 3

class RepairBead:
    """K&V repair bead with RepairAgent-style structure."""

    def __init__(self, max_iterations: int = 3):
        self.state = RepairState.UNDERSTAND
        self.context = RepairContext(
            failing_test="",
            error_message="",
            max_iterations=max_iterations
        )

    def execute(self):
        """Main repair loop."""
        while self.state not in [RepairState.DONE, RepairState.FAILED]:
            if self.context.iteration >= self.context.max_iterations:
                self.state = RepairState.FAILED
                break

            # Execute current state
            if self.state == RepairState.UNDERSTAND:
                self.understand()
            elif self.state == RepairState.LOCALIZE:
                self.localize()
            elif self.state == RepairState.GENERATE_FIX:
                self.generate_fix()
            elif self.state == RepairState.VALIDATE:
                self.validate()

            self.context.iteration += 1

    def understand(self):
        """State: UNDERSTAND - Analyze the failure."""
        print(f"[UNDERSTAND] Reading test and error...")

        # Tool: run_test
        test_result = run_test(self.context.failing_test)

        # Extract error information
        self.context.error_message = test_result.error
        self.context.error_location = parse_error_location(test_result.stack_trace)

        print(f"  Error: {self.context.error_message}")
        print(f"  Location: {self.context.error_location}")

        # Transition
        self.state = RepairState.LOCALIZE

    def localize(self):
        """State: LOCALIZE - Find relevant code."""
        print(f"[LOCALIZE] Finding suspect code...")

        file, line = self.context.error_location

        # Tool: read_file
        code_context = read_file(file, start=line-10, end=line+10)

        # Tool: search_code (find related functions)
        related = search_code_for_pattern(self.context.error_message)

        self.context.suspect_files = [file] + related

        print(f"  Suspect files: {self.context.suspect_files}")

        # Transition
        self.state = RepairState.GENERATE_FIX

    def generate_fix(self):
        """State: GENERATE_FIX - Synthesize patch."""
        print(f"[GENERATE_FIX] Creating patch...")

        # Read all suspect code
        context_code = ""
        for file in self.context.suspect_files:
            context_code += read_file(file)

        # Generate fix using LLM
        fix_prompt = f"""
Bug Report:
  Test: {self.context.failing_test}
  Error: {self.context.error_message}
  Location: {self.context.error_location}

Relevant Code:
{context_code}

Generate a MINIMAL fix for this specific error.
Only modify what is necessary to pass the test.
"""

        fix = call_llm(fix_prompt)
        self.context.fix_attempted = fix

        # Tool: write_file
        apply_fix(fix)

        print(f"  Applied fix to {self.context.error_location[0]}")

        # Transition
        self.state = RepairState.VALIDATE

    def validate(self):
        """State: VALIDATE - Test the fix."""
        print(f"[VALIDATE] Running tests...")

        # Tool: run_test (specific failing test)
        test_result = run_test(self.context.failing_test)

        if test_result.passed:
            # Tool: run_all_tests (full suite)
            all_results = run_all_tests()

            if all_results.all_passed:
                print("  ✓ All tests pass!")
                self.state = RepairState.DONE
            else:
                print("  ✗ Introduced regression")
                # Revert and try different approach
                self.state = RepairState.GENERATE_FIX
        else:
            print("  ✗ Test still fails")
            # Analyze new error
            self.context.error_message = test_result.error
            self.state = RepairState.LOCALIZE
```

---

## Key Insights from RepairAgent

### 1. State Machines Prevent Wandering

```
Without State Machine:
  Attempt 1: Try random fix
  Attempt 2: Try different random fix
  Attempt 3: Try another random fix
  Result: No systematic progress

With State Machine:
  Attempt 1: UNDERSTAND → LOCALIZE → FIX → VALIDATE (fail)
  Attempt 2: Re-LOCALIZE with new info → FIX → VALIDATE (fail)
  Attempt 3: Different LOCALIZE strategy → FIX → VALIDATE (pass)
  Result: Converges by narrowing search space
```

### 2. Tool Use is Disciplined

```
Each state has specific tools:

UNDERSTAND:
  ✓ run_test
  ✓ get_stack_trace
  ✗ write_file (not allowed yet)

LOCALIZE:
  ✓ search_code
  ✓ read_file
  ✗ write_file (not allowed yet)

GENERATE_FIX:
  ✓ read_file
  ✓ write_file
  ✗ run_test (comes later)

VALIDATE:
  ✓ run_test
  ✓ run_all_tests
  ✓ diff
```

### 3. Stop Conditions are Explicit

| Condition | Action |
|-----------|--------|
| All tests pass | DONE (success) |
| Max iterations reached | FAILED (give up, escalate) |
| Same error 2x in a row | FAILED (stuck, need different approach) |
| Regression introduced | Revert, try different fix |

---

## Bead Template: RepairAgent-Style

```markdown
## Repair Bead: bd-repair-123

### State: UNDERSTAND

**Failing Test:** `test_user_authentication`

**Tools Used:**
- `run_test("test_user_authentication")`

**Observations:**
```
AssertionError: Expected user to be authenticated
  File "tests/test_auth.py", line 45
    assert user.is_authenticated == True
```

**Error Analysis:**
- Type: AssertionError
- Location: src/auth.py:127
- Expected: user.is_authenticated == True
- Actual: user.is_authenticated == False

**Next State:** LOCALIZE

---

### State: LOCALIZE

**Tools Used:**
- `read_file("src/auth.py", lines=120-135)`
- `search_code("is_authenticated")`

**Suspect Code:**
```python
# src/auth.py:127
def authenticate(self, credentials):
    if self.validate_token(credentials.token):
        return True  # BUG: Doesn't set is_authenticated flag
    return False
```

**Hypothesis:** `authenticate()` validates but doesn't update state

**Next State:** GENERATE_FIX

---

### State: GENERATE_FIX

**Proposed Fix:**
```python
# src/auth.py:127
def authenticate(self, credentials):
    if self.validate_token(credentials.token):
        self.is_authenticated = True  # FIX: Set the flag
        return True
    return False
```

**Change Summary:**
- Added line 129: `self.is_authenticated = True`
- Minimal change, targets exact failure

**Tools Used:**
- `write_file("src/auth.py", fix)`

**Next State:** VALIDATE

---

### State: VALIDATE

**Tools Used:**
- `run_test("test_user_authentication")`
- `run_all_tests()`

**Results:**
- Specific test: ✓ PASS
- Full suite: ✓ PASS (15/15)
- No regressions introduced

**Next State:** DONE

---

### Summary

- **Iterations:** 1
- **Outcome:** SUCCESS
- **Files Modified:** src/auth.py (1 line added)
- **Tests Fixed:** test_user_authentication
- **Tests Passed:** 15/15
```

---

## Cost Analysis

### Token Usage (from paper)

```
Average per Defects4J bug:
  Understanding: 50k tokens
  Localization: 80k tokens
  Fix generation: 60k tokens
  Validation: 80k tokens
  Total: ~270k tokens

At $0.01/1k tokens:
  Cost per bug: ~$2.70
  164 bugs: ~$443
```

### Time Analysis

```
Average wall-clock time per bug:
  ~15 minutes

Breakdown:
  Tool execution: 8 minutes (tests, file I/O)
  LLM calls: 5 minutes (API latency)
  State transitions: 2 minutes (control logic)
```

---

## Key Takeaways

1. **Structure beats prompting:** State machines prevent aimless exploration
2. **Tools are first-class:** Each state has specific allowed tools
3. **Validation is the loop:** Tests provide the feedback signal
4. **Stop conditions matter:** Explicit termination prevents infinite loops
5. **Cost is reasonable:** ~$3 per bug fixed at scale
6. **Generalize beyond repair:** Same pattern works for any multi-step task
7. **K&V alignment:** RepairAgent validates beads + tools + state tracking

---

## See Also

- `043-rlef.md` — Learning from execution feedback
- `041-debatecoder.md` — Test-based validation
- `042-rankef.md` — Selecting among repair candidates
- `060-debugging-decay-index.md` — Why iteration limits matter
- `038-adapt.md` — When to decompose instead of repair
- `046-d4c.md` — Direct debugging vs. localization
