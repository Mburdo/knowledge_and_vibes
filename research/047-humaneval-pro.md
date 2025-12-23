# HumanEval Pro: Self-Invoking Code Generation

**Paper:** HumanEval Pro and MBPP Pro: Evaluating Large Language Models on Self-invoking Code Generation
**URL:** https://arxiv.org/abs/2412.21199
**Date:** December 2024
**Venue:** arXiv preprint (later ACL Findings)

---

## Summary

HumanEval Pro introduces **self-invoking code generation**: models must solve a base programming task and then use the generated solution as a tool to solve a related, harder task. This tests whether models can build on their own work without drift—a critical capability for multi-phase development workflows.

**Key innovation:** Benchmarks progressive reasoning by requiring models to reuse their own code, exposing brittleness that standard benchmarks miss.

---

## The Self-Invoking Challenge

### Standard Benchmark (HumanEval)

```
Task: Write function is_palindrome(s: str) -> bool

Model generates:
def is_palindrome(s):
    return s == s[::-1]

Test: ✓ PASS
Score: 1.0
```

### Self-Invoking Benchmark (HumanEval Pro)

```
Task 1 (Base): Write function is_palindrome(s: str) -> bool

Model generates:
def is_palindrome(s):
    return s == s[::-1]

Test 1: ✓ PASS

---

Task 2 (Pro): Use is_palindrome to solve
"Find longest palindromic substring"

Model must now:
  1. Recall it has is_palindrome available
  2. Use it correctly in new solution
  3. Handle edge cases in composition

Model generates:
def longest_palindrome(s):
    # Common failure: Doesn't use is_palindrome!
    # Or: Uses it wrong (off-by-one errors)
    # Or: Forgets it exists
    max_len = 0
    result = ""
    for i in range(len(s)):
        for j in range(i, len(s)):
            if is_palindrome(s[i:j+1]):  # Must invoke previous function
                if len(s[i:j+1]) > max_len:
                    max_len = len(s[i:j+1])
                    result = s[i:j+1]
    return result

Test 2: Often FAIL (10-15% drop vs. standalone)
```

---

## Performance Degradation

### Reported Results

| Model | HumanEval | HumanEval Pro | Drop |
|-------|-----------|---------------|------|
| GPT-4 | 87.2% | 74.8% | -12.4% |
| Claude 3 Opus | 84.1% | 71.3% | -12.8% |
| GPT-3.5 | 72.6% | 58.1% | -14.5% |
| CodeLlama 34B | 68.9% | 54.2% | -14.7% |

### Performance Visualization

```
Pass Rate: Standard vs. Self-Invoking

100% ┤████ Standard HumanEval
 90% ┤████
 80% ┤████  ▓▓▓▓ HumanEval Pro
 70% ┤████  ▓▓▓▓
 60% ┤████  ▓▓▓▓
 50% ┤████  ▓▓▓▓
 40% ┤████  ▓▓▓▓
 30% ┤████  ▓▓▓▓
 20% ┤████  ▓▓▓▓
 10% ┤████  ▓▓▓▓
  0% ┼────────────────
     GPT-4  Claude  GPT   CodeL
            Opus   3.5   34B

Average 10-15% absolute drop when requiring self-invocation.
```

---

## Common Failure Modes

### 1. Forgetting Available Tools

```python
# Task 1: Implemented is_palindrome
def is_palindrome(s):
    return s == s[::-1]

# Task 2 (Pro): Find all palindromic substrings
def find_palindromes(s):
    result = []
    for i in range(len(s)):
        for j in range(i+1, len(s)+1)):
            # FAILURE: Re-implements palindrome check instead of using is_palindrome
            if s[i:j] == s[i:j][::-1]:  # Should call is_palindrome(s[i:j])
                result.append(s[i:j])
    return result
```

### 2. Incorrect Invocation

```python
# Task 2 (Pro): Count palindromes of specific length
def count_palindromes_length(s, length):
    count = 0
    for i in range(len(s) - length + 1):
        substring = s[i:i+length]
        # FAILURE: Wrong parameters or invocation pattern
        if is_palindrome():  # Missing argument!
            count += 1
    return count
```

### 3. Name/Signature Drift

```python
# Task 1: Base implementation
def is_palindrome(text):  # Note: parameter named 'text'
    return text == text[::-1]

# Task 2 (Pro):
def palindrome_filter(words):
    # FAILURE: Uses wrong parameter name
    return [w for w in words if is_palindrome(s)]  # 's' not defined, should be 'w'
```

---

## Integration with Knowledge & Vibes

### Why This Matters for Multi-Phase Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│          K&V MULTI-PHASE = SELF-INVOKING AT SCALE                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: Build foundation                                       │
│      ├── Create utility functions                                │
│      ├── Define data models                                      │
│      └── Tests pass                                              │
│                                                                  │
│  Phase 2: Build on Phase 1 (Self-Invoking!)                      │
│      ├── Must use Phase 1 code correctly                         │
│      ├── Must not re-implement Phase 1 logic                     │
│      ├── Must handle Phase 1 edge cases                          │
│      └── Tests pass                                              │
│                                                                  │
│  Phase 3: Build on Phase 1 + 2 (More Self-Invoking!)             │
│      └── Compounds the difficulty                                │
│                                                                  │
│  HumanEval Pro models this pattern at small scale                │
│  K&V faces it at LARGE scale across many phases                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Calibration Points Justified

HumanEval Pro validates K&V's between-phase calibration:

| K&V Design Choice | HumanEval Pro Evidence |
|-------------------|------------------------|
| **Calibration between phases** | 10-15% drop shows phase transitions are fragile |
| **Explicit AC verification** | Tests catch self-invocation failures |
| **Interface contracts** | Prevent signature drift between phases |
| **Documentation requirements** | Help next phase understand prior work |
| **Progressive integration tests** | Catch composition errors early |

---

## Mitigation Strategies

### 1. Explicit Tool Inventory

```python
# At start of Phase 2 bead
"""
Available Tools from Phase 1:
  - is_palindrome(s: str) -> bool
    Purpose: Check if string is palindrome
    Example: is_palindrome("racecar") -> True

Task: Use is_palindrome to implement find_palindromes()
"""
```

### 2. Reuse Tests

```python
# Phase 2 bead should include Phase 1 tests
def test_phase2_uses_phase1():
    """Verify Phase 2 correctly invokes Phase 1 code."""
    # This test would FAIL if Phase 2 re-implements instead of reusing
    with patch('module.is_palindrome') as mock_palindrome:
        mock_palindrome.return_value = True
        result = find_palindromes("test")
        assert mock_palindrome.called, "Must use is_palindrome from Phase 1"
```

### 3. Interface Stability

```markdown
## Phase 1 AC (Acceptance Criteria)

Functions must have stable signatures:
- is_palindrome(s: str) -> bool  [LOCKED SIGNATURE]
  No changes allowed in future phases
```

---

## Code Example: Testing Self-Invocation

```python
import pytest
from unittest.mock import patch, call

def test_self_invocation_pattern():
    """
    Test that Phase 2 correctly uses Phase 1 code.

    This is the HumanEval Pro pattern applied to K&V.
    """

    # Phase 1: Base function
    def is_palindrome(s: str) -> bool:
        return s == s[::-1]

    # Phase 2: Should use Phase 1
    def find_palindromes(text: str) -> list[str]:
        words = text.split()
        return [w for w in words if is_palindrome(w)]

    # Test 1: Functional correctness
    result = find_palindromes("racecar hello level world")
    assert set(result) == {"racecar", "level"}

    # Test 2: Actually uses is_palindrome (not reimplemented)
    with patch(__name__ + '.is_palindrome', wraps=is_palindrome) as mock:
        find_palindromes("test noon test")
        # Verify is_palindrome was called for each word
        assert mock.call_count == 4
        mock.assert_any_call("test")
        mock.assert_any_call("noon")
```

---

## Key Metrics to Track

```json
{
  "phase": 2,
  "self_invocation_check": {
    "phase_1_functions_available": ["is_palindrome", "validate_input"],
    "phase_1_functions_used": ["is_palindrome"],
    "phase_1_functions_reimplemented": [],  // Should be empty!
    "invocation_test_pass": true,
    "signature_compatibility": true
  },
  "outcome": "PASS"
}
```

---

## Caveats and Limitations

### Benchmark vs. Real-World

| HumanEval Pro | K&V Multi-Phase | Implication |
|---------------|-----------------|-------------|
| 2-task sequence | 5-10 phase pipeline | More opportunities for drift |
| Same session | Cross-session | Context reset between phases |
| Single file | Multi-file modules | Import/dependency management |
| Minutes apart | Days/weeks apart | Memory fade (but Git helps) |

---

## Key Takeaways

1. **Self-invocation is hard:** 10-15% drop even for best models
2. **Reuse != correctness:** Models may try to use prior code but do it wrong
3. **Phase boundaries are fragile:** Each phase compounds difficulty
4. **Calibration is justified:** Between-phase checks catch self-invocation errors
5. **Tool inventory helps:** Explicit list of available functions reduces forgetting
6. **Tests must verify reuse:** Not just correctness, but composition
7. **K&V faces this at scale:** Multi-phase workflow multiplies the challenge

---

## See Also

- `038-adapt.md` — Phase decomposition strategy
- `041-debatecoder.md` — Test-based validation
- `060-debugging-decay-index.md` — Why iteration limits matter
- `050-swe-bench-pro.md` — Realistic benchmark challenges
- `065-confucius-code-agent.md` — Long-horizon session management
