# Lost-in-Middle for Code

**Paper:** Sense and Sensitivity: Examining the Influence of Semantic Recall on Long Context Code Reasoning
**URL:** https://arxiv.org/abs/2505.13353
**Date:** May 2025

---

## Summary

There's a "lost-in-the-middle" effect for code, but the severity depends on whether you're testing **lexical recall** (can you reproduce the code?) vs **semantic recall** (do you understand what it does?).

**Key finding:** Semantic understanding suffers much more than lexical retrieval.

---

## The Two Types of Recall

### Lexical vs Semantic

```
┌─────────────────────────────────────────────────────────────────┐
│              LEXICAL vs SEMANTIC RECALL                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LEXICAL RECALL                                                  │
│  "Reproduce the function"                                        │
│  ├── Can find it in context                                     │
│  ├── Can copy it accurately                                     │
│  └── Works well even with distractors                           │
│                                                                  │
│  SEMANTIC RECALL                                                 │
│  "What does the function do?"                                   │
│  ├── Must understand logic                                      │
│  ├── Must trace execution                                       │
│  └── Severely affected by position                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Experimental Setup

### Methodology

Embed target function within 20-80 distractor functions, vary position, measure performance.

```
Context Structure:
────────────────────────────────────────────────────────────────────
[Distractors: 0-40 functions]
[Target Function]
[Distractors: 0-40 functions]
[Question about target]

Position: Start / Middle / End
Total distractors: 20 / 40 / 80
────────────────────────────────────────────────────────────────────
```

---

## Performance Results

### Lexical Recall (Reproducing Code)

| Condition | Performance |
|-----------|-------------|
| Function-level | High accuracy across positions |
| Line-level, 20 distractors | 16% drop |
| Line-level, 80 distractors | 36% drop |

```
Lexical Recall by Position:
────────────────────────────────────────────────────────────────────
Start    ████████████████████████████████████████  ~95%
Middle   ████████████████████████████████████████  ~90%
End      ████████████████████████████████████████  ~93%
────────────────────────────────────────────────────────────────────
Position matters less for lexical tasks
```

### Semantic Recall (Understanding Code)

| Model | Task | Drop |
|-------|------|------|
| Qwen 2.5 7B | CruxEval Input | 79.55% |
| Llama 3.1 8B | CruxEval Input | 46.67% |
| Small models | SemTrace | 94% (near-zero accuracy) |

```
Semantic Recall by Position:
────────────────────────────────────────────────────────────────────
Start    ████████████████████████████████████████  ~80%
Middle   █████████████████                          ~35%
End      ██████████████████████████████████        ~70%
────────────────────────────────────────────────────────────────────
Middle position is catastrophic for semantic tasks
```

---

## The Independence Finding

### Lexical ≠ Semantic

```
Model Behavior Matrix:
────────────────────────────────────────────────────────────────────
                    Good Lexical    Bad Lexical
                    ──────────────  ──────────────
Good Semantic       Ideal           Rare
Bad Semantic        Gemma 3 27B     Worst case

Gemma 3 27B: Great at finding code, bad at understanding it
────────────────────────────────────────────────────────────────────
```

Some models can perfectly reproduce code they don't understand. Others understand code they can't accurately reproduce.

---

## The 94% Number in Context

### SemTrace Task

The 94% drop is on SemTrace, a synthetic task:
- Randomized arithmetic operations
- Guessing is impossible
- Designed to isolate semantic reasoning

### Real-World Comparison

| Task | Drop | Context |
|------|------|---------|
| SemTrace | 94% | Synthetic, guessing impossible |
| CruxEval | 30-80% | Real code reasoning |
| Function reproduction | 16-36% | Lexical only |

**Takeaway:** 94% is worst-case. Real code reasoning shows 30-50% drops.

---

## Position Effects

### The U-Shaped Curve

```
Performance
    │
100%┤██                                          ██
 80%┤████                                      ████
 60%┤██████                                  ██████
 40%┤████████                              ████████
 20%┤██████████████████████████████████████████████
    └──────────────────────────────────────────────
     Start                              Middle                              End
```

Information at the start and end is processed better than information in the middle.

### Why This Happens

| Position | Attention | Performance |
|----------|-----------|-------------|
| Start | High (primacy) | Good |
| Middle | Diluted | Poor |
| End | High (recency) | Good |

---

## Practical Implications

### For Knowledge & Vibes

| Finding | Application |
|---------|-------------|
| Position matters for reasoning | Put critical code at section boundaries |
| Function retrieval works | Locating functions is reliable |
| Semantic understanding suffers | Planning/reasoning needs care |
| Larger models more stable | Use capable models for reasoning |

### Positioning Strategy

```markdown
## Context Organization

Good:
- Critical requirements at start of phase doc
- Key decisions at section boundaries
- Important code at start/end of file excerpts

Bad:
- Important info buried in middle of long section
- Critical decisions mixed with routine content
- Key code surrounded by unrelated functions
```

### What This Means for Codemaps

```
Codemap Structure:
────────────────────────────────────────────────────────────────────
[Most important: Key files, entry points]        ← Start (high attention)
[Supporting info: Dependencies, patterns]        ← Middle (lower attention)
[Critical warnings: Gotchas, constraints]        ← End (high attention)
────────────────────────────────────────────────────────────────────
```

---

## Key Takeaways

1. **Position matters for reasoning** - Middle position is worst
2. **Lexical recall is robust** - Finding functions works
3. **Semantic recall suffers** - Understanding code is harder
4. **94% drop is worst-case** - Real code shows 30-50% drops
5. **Larger models help** - GPT-4 class more resilient

---

## See Also

- `004-context-length-hurts.md` - Length effects beyond position
- `057-anthropic-context-engineering.md` - Minimal context principle
- `017-context-retrieval-repo-editing.md` - Context retrieval strategies
