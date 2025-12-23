---
title: Context Length Hurts Performance
tags: [context, long-context, retrieval, degradation, chunking]
protocols: [P5]
---

# Context Length Alone Hurts Performance

**Paper:** Context Length Alone Hurts LLM Performance Despite Perfect Retrieval
**URL:** https://arxiv.org/abs/2510.05381
**Date:** October 2025

---

## Summary

LLM performance degrades with longer inputs **even when the model can perfectly retrieve all relevant information**. This happens even when irrelevant tokens are replaced with whitespace or completely masked.

**Key finding:** The problem is processing long input, not finding information in it.

---

## The Experimental Setup

### Methodology

Split short problems into evidence + question, insert distractors between them:

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXPERIMENTAL DESIGN                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [Evidence]                                                      │
│  "Given: x = 5, y = 3"                                          │
│                                                                  │
│  [Distractors: 0-30K tokens]                                    │
│  Essays, whitespace, or masked tokens                           │
│                                                                  │
│  [Question]                                                      │
│  "What is x + y?"                                               │
│                                                                  │
│  Measure: Can model solve, not just retrieve?                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Performance Degradation

### With Essay Distraction (0-30K tokens)

| Model | Task | Degradation |
|-------|------|-------------|
| Llama3 8B | HumanEval (code) | 47.6% drop |
| Llama3 8B | VarSum (synthetic) | 85% drop |
| Mistral 7B | GSM8K (math) | 34.2% drop |

### With Whitespace Only

| Model | Task | Degradation |
|-------|------|-------------|
| Llama3 8B | VarSum | 48% drop at 30K |
| Mistral 7B | GSM8K | 30% drop at 30K |

### With Complete Masking (Zero Distraction)

Still **21-50% drops**. Even when the model can only attend to relevant tokens.

```
Performance vs Context Length:
────────────────────────────────────────────────────────────────────
100% ████████████████████████████████████████  (0K tokens)
 80% ██████████████████████████████████        (5K tokens)
 60% ██████████████████████████                (10K tokens)
 40% ████████████████                          (20K tokens)
 20% ████████                                  (30K tokens)
────────────────────────────────────────────────────────────────────
```

---

## Why This Happens

### Not a Retrieval Problem

The model can **find** the relevant information. The problem is using it:

```
What works:                    What breaks:
──────────────────            ──────────────────
"Find x in text"              "Use x to solve Y"
Model finds x                 Model finds x but
                              reasoning degrades
```

### Hypothesized Causes

| Cause | Mechanism |
|-------|-----------|
| Attention dilution | Attention spread across more tokens |
| Working memory limits | Too much to hold during reasoning |
| Position encoding artifacts | Distant tokens harder to relate |
| Interference | Irrelevant tokens create noise |

---

## Task Type Matters

### Degradation by Task Type

```
Degradation Severity:
────────────────────────────────────────────────────────────
Synthetic (VarSum)      ██████████████████████████████████████████  85%
Real code (HumanEval)   ██████████████████████████  47.6%
Math (GSM8K)            █████████████████  34.2%
────────────────────────────────────────────────────────────
```

| Task Type | Severity | Notes |
|-----------|----------|-------|
| Synthetic (VarSum) | 85% drop | Designed to prevent guessing |
| Code (HumanEval) | 47.6% drop | Real-world coding tasks |
| Math (GSM8K) | 34% drop | Multi-step arithmetic |

The 85% number is worst-case: a small model on a task designed to be hard.

---

## Model Size Matters

### Larger Models More Resilient

| Model Class | Degradation | Notes |
|-------------|-------------|-------|
| Small open source | Severe (50-85%) | Llama3 8B, Mistral 7B |
| Large open source | Moderate (30-50%) | Llama3 70B |
| Frontier (GPT-4o, Claude) | Lower (20-30%) | Closed-source models |

**Implication:** If using frontier models, the problem is less severe but still present.

---

## Mitigation: Retrieve Then Solve

### The Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│                  RETRIEVE THEN SOLVE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: RECITE                                                  │
│  ├── Model extracts relevant evidence from long context         │
│  └── Writes it into short output                                │
│                                                                  │
│  Step 2: SOLVE                                                   │
│  ├── Use only the short recited output                          │
│  └── Solve with minimal context                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Results

| Method | GSM8K at 26K tokens |
|--------|---------------------|
| Direct solve | 35.5% |
| Retrieve then solve | 66.7% |

**+31.2% improvement** by extracting relevant info first, then solving with short context.

---

## Practical Implications

### For Knowledge & Vibes

| Finding | Application |
|---------|-------------|
| Long context hurts reasoning | Phase chunking is correct |
| Task type affects severity | Code less affected than synthetic |
| Frontier models more resilient | Use capable models |
| Retrieve-then-solve works | Extract info before reasoning |

### What This Validates

```markdown
## K&V Patterns Supported by This Research

1. Phase documents (chunked context)
   - Don't load entire project context
   - Load current phase only

2. Bead-level work
   - Focus on single task at a time
   - Minimal context per task

3. CASS/cm pattern
   - Retrieve relevant past solutions
   - Use short excerpts, not full sessions

4. Calibration checkpoints
   - Fresh context between phases
   - Prevents accumulated degradation
```

### What Not to Do

- Don't set arbitrary line limits based on this paper
- The 85% is worst-case on synthetic tasks
- Frontier models show smaller degradation
- Context length itself isn't evil; unstructured long context is

---

## Key Takeaways

1. **Long context degrades reasoning** - Even with perfect retrieval
2. **85% drop is worst-case** - Small model, synthetic task
3. **Task type matters** - Code less affected than synthetic
4. **Model size helps** - Frontier models more resilient
5. **Retrieve-then-solve works** - Extract first, then reason

---

## See Also

- `005-lost-in-middle-code.md` - Position effects in code
- `057-anthropic-context-engineering.md` - Minimal context principle
- `065-confucius-code-agent.md` - Progressive context loading
