---
title: Debugging Decay Index 2025
tags: [debugging, decay, iteration, self-correction, effectiveness, repair]
protocols: [P8, P11]
---

# Debugging Decay Index 2025

**Paper:** Measuring and mitigating debugging effectiveness decay in code language models
**URL:** https://www.nature.com/articles/s41598-025-27846-5
**Date:** December 2025

**Related:** The Debugging Decay Index: Rethinking Debugging Strategies for Code LLMs (arXiv:2506.18403)

---

## Summary

Critical 2025 research quantifying how LLM debugging effectiveness degrades with repeated attempts. Introduces the **Debugging Decay Index (DDI)** mathematical framework.

**Key finding:** Most models lose **60-80% of their debugging capability within just 2-3 attempts**.

---

## The Debugging Decay Index (DDI)

### Mathematical Model

Debugging effectiveness follows **exponential decay**:

```
E(t) = E₀ × e^(-λt)

Where:
  E(t) = Effectiveness at attempt t
  E₀   = Initial effectiveness (first attempt)
  λ    = Decay constant (model-specific)
  t    = Attempt number
```

### Observed Decay Rates

| Model Category | λ (Decay Constant) | Effectiveness at Attempt 3 |
|----------------|--------------------|-----------------------------|
| GPT-4 class | 0.3-0.4 | 35-45% of initial |
| GPT-3.5 class | 0.5-0.7 | 15-25% of initial |
| Open source | 0.4-0.6 | 20-35% of initial |

### Visual Representation

```
Effectiveness
100% ┤████████████████████████████████████████
 90% ┤█████████████████████████████
 80% ┤███████████████████████
 70% ┤█████████████████
 60% ┤████████████
 50% ┤████████
 40% ┤█████
 30% ┤███
 20% ┤██
 10% ┤█
  0% ┼────────────────────────────────────────
     1    2    3    4    5    6    7    8    9   10
                    Attempt Number
```

---

## Why Decay Happens

### Root Causes

1. **Semantic drift** — Each edit moves further from original intent
2. **Error compounding** — Fixes introduce new bugs
3. **Context pollution** — Failed attempts pollute context window
4. **Hallucination amplification** — Incorrect assumptions reinforce
5. **Loss of signal** — Original error signal diluted by noise

### The Overcorrection Problem

From Stanford research (2025):

```
Attempt 1: Fix line 42 (correct target)
Attempt 2: Also "fix" lines 40-45 (overcorrection)
Attempt 3: Refactor entire function (catastrophic overcorrection)
Attempt 4: Code barely resembles original (unrecoverable)
```

---

## Practical Implications

### The 3-Attempt Rule

**After 3 failed repair attempts: STOP and change strategy.**

```
┌─────────────────────────────────────────────────────────────┐
│                    DEBUGGING PROTOCOL                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Attempt 1: Fix the bug                                      │
│      │                                                       │
│      ├── Success? → Done                                     │
│      └── Failure? → Continue                                 │
│                                                              │
│  Attempt 2: Fix with additional context                      │
│      │                                                       │
│      ├── Success? → Done                                     │
│      └── Failure? → Continue                                 │
│                                                              │
│  Attempt 3: Different approach                               │
│      │                                                       │
│      ├── Success? → Done                                     │
│      └── Failure? → STOP ITERATING                          │
│                                                              │
│  After 3 failures:                                           │
│      ├── Option A: Fresh start (new context)                 │
│      ├── Option B: ADaPT sub-bead (isolate problem)          │
│      └── Option C: Escalate to human/coordinator             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Fresh Start Strategy

The research proposes a "strategic fresh start approach":

1. **Detect decay threshold** — When effectiveness drops below useful
2. **Archive context** — Save what was learned
3. **Reset context** — Start with clean slate
4. **Inject learnings** — Add only essential insights to new context

---

## Mitigation Strategies

### From 2025 Research

| Strategy | Effectiveness | Cost |
|----------|---------------|------|
| **Fresh start at attempt 3** | High | Medium (restart overhead) |
| **ADaPT decomposition** | High | Low (targeted split) |
| **Stronger model feedback** | Medium | High (API costs) |
| **AST-diff constraints** | Medium | Low |
| **Test-guided repair** | Medium | Low |

### AST-Based Constraints

Limit the scope of changes per attempt:

```python
# Constraint: Changes must be within N AST nodes of error location
max_ast_distance = 3  # Lines of related code

# If proposed fix exceeds constraint, reject and ask for smaller fix
if ast_distance(original, proposed) > max_ast_distance:
    reject("Fix too broad. Focus on the specific error location.")
```

---

## Integration with ADaPT

The decay research validates ADaPT's core insight:

| ADaPT Principle | DDI Support |
|-----------------|-------------|
| Decompose on failure | Decay makes continued iteration harmful |
| 3-iteration limit | Matches 60-80% decay threshold |
| Fresh context for sub-bead | Resets decay curve |
| Isolate specific problem | Prevents overcorrection |

### ADaPT + DDI Flow

```
Task Bead: "Fix authentication bug"
    │
    ├── Attempt 1: Fail (E = 100%)
    ├── Attempt 2: Fail (E = 55%)
    ├── Attempt 3: Fail (E = 30%)
    │
    └── STOP: Create ADaPT Sub-Bead
            │
            ├── Isolate: "JWT validation fails for expired tokens"
            ├── Fresh context (E resets to 100%)
            └── Focused problem (higher success rate)
```

---

## Relevance to Protocols

| Protocol | Impact |
|----------|--------|
| P7 Bead Packaging | Max 3 iterations before decompose |
| Execution agents | Hard stop at iteration 3 |
| ADaPT Pattern | Core justification |
| Calibration | Don't iterate through failures |

---

## Key Metrics to Track

When implementing, track:

```json
{
  "bead_id": "bd-101",
  "iterations": 3,
  "success": false,
  "action": "adapt_decompose",
  "sub_bead_created": "bd-101.1",
  "notes": "DDI threshold reached, fresh context needed"
}
```

---

## See Also

- `038-adapt.md` — Adaptive decomposition pattern
- `006-dark-side-self-correction.md` — Early self-correction limits research
- `022-chatrepair.md` — Iterative repair approaches
- `024-thinkrepair.md` — Think-before-repair strategies
