---
title: DebateCoder - Test-Case-Driven Debate for Code Generation
tags: [multi-agent, debate, testing, adversarial, evidence-based, convergence]
protocols: [P6, P10, P11]
---

# DebateCoder: Test-Case-Driven Debate for Code Generation

**Paper:** DebateCoder: Towards Collective Intelligence of LLMs via Test Case Driven LLM Debate for Code Generation
**URL:** https://aclanthology.org/2025.acl-long.589/
**Date:** 2025
**Venue:** ACL 2025 (Long)

---

## Summary

DebateCoder transforms multi-agent debate from **rhetorical argumentation** to **executable evidence warfare**. Instead of models arguing through natural language about which solution is better, they generate adversarial test cases to break each other's code. The winning solution is determined by test execution outcomes, not by a judge model's opinion.

**Key innovation:** Tests become the medium of disagreement, replacing subjective debate with objective execution results.

---

## The Debate Problem

### Traditional Multi-Agent Debate

```
┌─────────────────────────────────────────────────────────────────┐
│              TRADITIONAL DEBATE (INEFFECTIVE)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Agent 1: "My solution is better because it handles edge cases"  │
│      ↓                                                           │
│  Agent 2: "No, my solution is more efficient"                    │
│      ↓                                                           │
│  Judge Model: "I think Agent 1 is correct" (subjective)          │
│                                                                  │
│  PROBLEMS:                                                       │
│  - Circular reasoning                                            │
│  - Judge may be wrong                                            │
│  - No objective validation                                       │
│  - Debate can go on indefinitely                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### DebateCoder Approach

```
┌─────────────────────────────────────────────────────────────────┐
│              DEBATECODER (EVIDENCE-BASED)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Agent 1: Generates solution_A                                   │
│      ↓                                                           │
│  Agent 2: Generates solution_B                                   │
│      ↓                                                           │
│  Agent 1: Creates test_X that breaks solution_B                  │
│      ↓                                                           │
│  Agent 2: Creates test_Y that breaks solution_A                  │
│      ↓                                                           │
│  EXECUTION: Run tests against both solutions                     │
│      ↓                                                           │
│  VERDICT: Solution passing more tests wins (objective)           │
│                                                                  │
│  ADVANTAGES:                                                     │
│  + Tests are executable evidence                                 │
│  + No need for judge model                                       │
│  + Converges naturally when no more breaks found                 │
│  + Strengthens test suite as byproduct                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## The DebateCoder Protocol

### Multi-Epoch Debate Flow

```
Epoch 0: Initial Generation
    ├── Agent 1 → solution_A
    └── Agent 2 → solution_B

Epoch 1: First Adversarial Round
    ├── Agent 1 → test_cases_against_B
    ├── Agent 2 → test_cases_against_A
    ├── Execute all tests
    ├── Refine solutions based on failures
    └── Check convergence

Epoch 2: Second Adversarial Round
    ├── Agent 1 → more_test_cases_against_B
    ├── Agent 2 → more_test_cases_against_A
    ├── Execute all tests
    ├── Refine solutions based on failures
    └── Check convergence

Epoch N: Convergence
    ├── No new breaking tests found
    └── Select solution passing most tests
```

### Convergence Criteria

| Condition | Meaning | Action |
|-----------|---------|--------|
| Both pass all tests | Solutions equivalent | Select either (or merge) |
| One passes more tests | Clear winner | Select winning solution |
| New tests still breaking | Debate continues | Refine + next epoch |
| Max epochs reached | Forced termination | Select best so far |

---

## Mathematical Model

### Test-Based Scoring

```
For solution S and test suite T:

Score(S, T) = Σ(pass(S, t)) for t in T
              ─────────────────────────
                    |T|

Where:
  pass(S, t) = 1 if solution S passes test t
             = 0 otherwise

Winner = argmax(Score(S, T))
         S ∈ {solution_A, solution_B}
```

### Debate Convergence

```
Convergence occurs when:

  ΔT(epoch_n) = 0

Where:
  ΔT(epoch_n) = number of new breaking tests discovered in epoch n

Practical limit:
  Max epochs = 3-5 (reported in paper)
```

---

## Benchmark Performance

### Reported Results

| Benchmark | Baseline | Multi-Agent Debate | DebateCoder | Improvement |
|-----------|----------|-------------------|-------------|-------------|
| APPS | 45.2% | 48.7% | 56.3% | +11.1% absolute |
| CodeContests | 28.4% | 31.2% | 37.8% | +9.4% absolute |
| HumanEval | 82.3% | 84.1% | 88.5% | +6.2% absolute |

### Convergence Speed

```
Percentage of Problems Converged

100% ┤                                    ████████████
 90% ┤                          ██████████
 80% ┤                    ██████
 70% ┤              ██████
 60% ┤        ██████
 50% ┤  ██████
 40% ┤██
  0% ┼──────────────────────────────────────────────
     0    1    2    3    4    5    6    7    8    9
                    Debate Epoch
```

Most problems converge within 3-5 epochs (reported).

---

## Integration with Knowledge & Vibes

### Calibration Enhancement

Current K&V calibration allows rhetorical disagreement. DebateCoder suggests upgrading:

```python
# OLD CALIBRATION (Rhetorical)
if agent_disagrees:
    agent.argue_in_text()
    human_decides()

# NEW CALIBRATION (Evidence-Based)
if agent_disagrees:
    if has_discriminating_test:
        agent.run_test()
        test_result_decides()
    else:
        agent.write_discriminating_test()
        agent.run_test()
        test_result_decides()
```

### Multi-Agent Calibration Protocol

```
┌─────────────────────────────────────────────────────────────────┐
│          DEBATECODER-STYLE CALIBRATION PROTOCOL                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: Implementation                                         │
│      ├── Agent A implements approach_A                           │
│      └── Agent B implements approach_B                           │
│                                                                  │
│  Phase 2: Adversarial Testing                                    │
│      ├── Agent A writes tests targeting approach_B               │
│      ├── Agent B writes tests targeting approach_A               │
│      └── Execute all tests against both approaches               │
│                                                                  │
│  Phase 3: Refinement                                             │
│      ├── Each agent fixes failures in their approach             │
│      └── Re-run all tests                                        │
│                                                                  │
│  Phase 4: Convergence Check                                      │
│      ├── If new breaking tests found → Phase 2                   │
│      ├── If no new tests AND both pass → Either approach OK      │
│      └── If no new tests AND one better → Select winner          │
│                                                                  │
│  Escalation: Max 3 debate rounds, then human decides             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practical Implementation Strategies

### For Bead-Level Disagreements

When two agents have different implementations for a bead:

1. **Require adversarial tests:** Each agent must write tests that might break the other's approach
2. **Execute objectively:** Run all tests against all implementations
3. **Select by results:** Choose implementation passing most tests
4. **Preserve test suite:** Keep all discriminating tests for regression

### For Design Debates

When agents disagree on architecture/approach:

| Question Type | DebateCoder-Style Answer |
|--------------|--------------------------|
| Performance? | Write benchmark test, measure both |
| Correctness? | Write edge-case tests, execute both |
| Maintainability? | Write refactoring test, see which holds |
| Compatibility? | Write integration tests, check both |

### Code Example: Adversarial Test Generation

```python
# Agent A's implementation
def parse_date(date_string):
    """Parse ISO 8601 date string."""
    return datetime.fromisoformat(date_string)

# Agent B's adversarial test (trying to break Agent A)
def test_parse_date_edge_cases():
    """Test cases that might break naive ISO parsing."""

    # Edge case 1: Timezone offset variations
    assert parse_date("2025-01-15T10:30:00+00:00")
    assert parse_date("2025-01-15T10:30:00-05:00")
    assert parse_date("2025-01-15T10:30:00Z")

    # Edge case 2: Leap seconds
    assert parse_date("2025-12-31T23:59:60Z")

    # Edge case 3: Invalid but plausible formats
    with pytest.raises(ValueError):
        parse_date("2025/01/15")  # Wrong separator

    with pytest.raises(ValueError):
        parse_date("15-01-2025")  # Wrong order

    # Edge case 4: Extreme dates
    assert parse_date("9999-12-31T23:59:59Z")
    assert parse_date("0001-01-01T00:00:00Z")

# Result: Agent A's implementation FAILS on leap seconds and extreme dates
# Agent A must refine implementation or argue test is invalid
```

---

## Caveats and Limitations

### Benchmark vs. Real-World

| Benchmark Context | Real-World Context | Implication |
|-------------------|-------------------|-------------|
| Self-contained problems | Multi-file changes | Tests must span interfaces |
| Clear specifications | Ambiguous requirements | Tests encode assumptions |
| Public test suites | Unknown edge cases | Adversarial tests find gaps |
| Single correct solution | Multiple valid approaches | Tests distinguish approaches |

### When DebateCoder Doesn't Apply

- **Ambiguous requirements:** Tests can't discriminate if spec is unclear
- **Non-functional concerns:** Hard to test aesthetics, readability
- **Performance trade-offs:** Need benchmarks, not just correctness tests
- **Integration issues:** May need environment-specific tests

---

## Key Takeaways

1. **Tests > Rhetoric:** Replace verbal debate with executable evidence
2. **Adversarial tests strengthen code:** Breaking attempts find real edge cases
3. **Objective convergence:** Debate ends when no new breaks found
4. **No judge needed:** Test execution outcomes are self-evident
5. **Byproduct value:** Adversarial tests improve overall test suite
6. **Fast convergence:** Most debates resolve in 3-5 rounds
7. **Calibration upgrade:** K&V should prefer test-based disagreement resolution

---

## See Also

- `042-rankef.md` — Ranking code candidates using execution feedback
- `043-rlef.md` — Training models to use execution feedback
- `006-dark-side-self-correction.md` — Limits of self-correction without feedback
- `022-chatrepair.md` — Iterative repair approaches
- `060-debugging-decay-index.md` — Why iteration limits matter
- `038-adapt.md` — Decomposition when approaches fail
