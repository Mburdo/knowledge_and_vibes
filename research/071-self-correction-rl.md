---
title: Self-Correction via Reinforcement Learning 2025
tags: [self-correction, rl, feedback, distribution-mismatch, score, repair]
protocols: [P8, P11]
---

# Self-Correction via Reinforcement Learning 2025

**Sources:**
- SCoRe: Self-Correction via Reinforcement Learning (ICLR 2025)
- FTR: Feedback-Triggered Regeneration (September 2025)
- CoCoS: Self-Correcting Code using Small LMs (EMNLP 2025)
- SuperCorrect: Thought Template Distillation (ICLR 2025)

---

## Summary

2025 research consolidating approaches for training LLMs to **self-correct effectively**. Key insight: prompting-based self-correction often fails; RL-trained correction works.

**Critical finding:** Standard self-correction methods suffer from distribution mismatch — models learn to correct others' errors, not their own.

---

## The Self-Correction Problem

### Why Prompting Fails

| Approach | Problem |
|----------|---------|
| "Check your work" | No new information; often makes things worse |
| "Are you sure?" | 58% of correct answers overturned |
| Multi-turn prompting | Distribution mismatch |
| External feedback | Expensive; not always available |

### Distribution Mismatch

```
Training: Model learns to correct errors from dataset X
Deployment: Model makes different errors (distribution Y)
Result: Correction strategies don't transfer

Example:
- Training data has off-by-one errors
- Model learns to fix off-by-one
- At deployment, model makes logic errors
- Off-by-one fix doesn't help logic errors
```

---

## SCoRe: Self-Correction via RL

### Core Innovation

Train on **self-generated** data to eliminate distribution mismatch:

```
┌─────────────────────────────────────────────────────────────────┐
│                      SCoRe TRAINING LOOP                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Model generates response (attempt 1)                         │
│  2. Check if correct                                             │
│     ├── Correct → No training needed                             │
│     └── Incorrect → Continue                                     │
│  3. Model generates correction (attempt 2)                       │
│  4. Check if correction is correct                               │
│  5. RL update:                                                   │
│     ├── If attempt 2 correct: Reward correction                  │
│     ├── If attempt 1 was better: Penalize regression             │
│     └── Shape rewards to avoid collapse                          │
│  6. Repeat on self-generated data                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Results

| Model | Base | + SFT | + SCoRe |
|-------|------|-------|---------|
| Gemini 1.0 Pro | 63.4% | 67.5% | 73.1% |
| Gemma 2 27B | 74.5% | 76.0% | 78.8% |

**Key:** SCoRe significantly outperforms supervised fine-tuning on correction traces.

---

## FTR: Feedback-Triggered Regeneration

### When to Correct

Don't self-assess — wait for external signal:

```
Standard Self-Correction:
├── Generate response
├── Self-assess (unreliable)
├── Self-correct (often wrong)
└── Errors compound

FTR Approach:
├── Generate response
├── Present to user/environment
├── IF negative feedback received:
│   └── Trigger regeneration with LTM decoding
├── ELSE:
│   └── Accept response (no unnecessary correction)
```

### Long-Term Multipath Decoding

When correction is triggered, explore multiple paths:

```
Failed Response
    │
    ├── Path A: Different approach entirely
    ├── Path B: Fix identified error
    ├── Path C: Reformulate problem
    └── Path D: Request clarification

Evaluate all paths → Select best
```

---

## CoCoS: Self-Correction for Small Models

### The Small Model Challenge

| Model Size | Self-Correction Ability |
|------------|------------------------|
| >70B | Often works with prompting |
| 7-13B | Sometimes works |
| **1-3B** | **Rarely works with prompting** |

### CoCoS Solution

Online RL with accumulated rewards:

```python
# CoCoS reward structure
def compute_reward(attempts: List[str], correct: str) -> float:
    accumulated = 0
    for i, attempt in enumerate(attempts):
        if is_correct(attempt, correct):
            # Earlier correct = higher reward
            accumulated += 1.0 / (i + 1)
        else:
            # Penalty for incorrect
            accumulated -= 0.1
    return accumulated
```

### Results on Small Models

| Model | Base | + CoCoS | Improvement |
|-------|------|---------|-------------|
| DeepSeek 1.3B | 28% | 41% | +13% |
| CodeLlama 1B | 24% | 38% | +14% |

---

## SuperCorrect: Thought Templates

### Hierarchical Reasoning

Use Buffer of Thought (BoT) templates:

```
Problem: {problem}

Meta Buffer (strategy selection):
├── Is this arithmetic? → Use calculation template
├── Is this logic? → Use deduction template
└── Is this code? → Use execution template

Template Buffer (domain-specific):
├── [CALCULATION] Step 1: ... Step 2: ...
├── [DEDUCTION] Premise: ... Conclusion: ...
└── [EXECUTION] Input: ... Output: ...

Reasoning:
{Apply selected template}

Self-Correction:
{Check against template constraints}
```

---

## Practical Implications

### For Knowledge & Vibes

| Research Pattern | K&V Application |
|------------------|-----------------|
| Don't self-assess | Wait for test results |
| External feedback triggers | TDD provides signal |
| Multi-path on failure | ADaPT sub-beads |
| Accumulated rewards | Track across iterations |
| Distribution matching | Agent corrects own errors |

### Correction Protocol

```
┌─────────────────────────────────────────────────────────────────┐
│                    K&V CORRECTION FLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Generate code                                                │
│  2. Run tests (external feedback)                                │
│      │                                                           │
│      ├── Tests pass → Done                                       │
│      └── Tests fail → Continue                                   │
│                                                                  │
│  3. Analyze failure (use test output, not self-assessment)       │
│  4. Generate correction based on failure                         │
│      │                                                           │
│      ├── Tests pass → Done                                       │
│      └── Tests fail → Continue                                   │
│                                                                  │
│  5. Attempt 3 with different approach                            │
│      │                                                           │
│      ├── Tests pass → Done                                       │
│      └── Tests fail → ADaPT (fresh context)                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Prompting-based correction often fails** — Especially for small models
2. **RL-trained correction works** — Train on self-generated errors
3. **External feedback > self-assessment** — Tests, not introspection
4. **Distribution matching matters** — Correct your own error types
5. **Multi-path exploration** — Try different approaches on failure
6. **Small models need RL** — Prompting insufficient for <7B

---

## See Also

- `006-dark-side-self-correction.md` — Why self-correction can hurt
- `060-debugging-decay-index.md` — Iteration limits
- `038-adapt.md` — Decomposition on failure
