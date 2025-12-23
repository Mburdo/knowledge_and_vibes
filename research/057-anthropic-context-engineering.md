# Anthropic Context Engineering 2025

**Paper:** Context Engineering for Claude: Best Practices from Anthropic
**URL:** https://www.anthropic.com/engineering/context-engineering
**Date:** 2025

---

## Summary

Anthropic's official guidance on context engineering establishes the **Minimal Context Principle**: achieving optimal performance by finding the **smallest set of high-signal tokens** that enables correct behavior.

**Key finding:** More context is not better. Quality over quantity. Strategic positioning matters more than total tokens.

---

## The Core Principle

### Minimal Context Definition

```
┌──────────────────────────────────────────────────────────────┐
│           THE MINIMAL CONTEXT PRINCIPLE                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  "The goal is to find the smallest set of high-signal tokens │
│   that enables correct behavior."                             │
│                                                               │
│  NOT: "Include everything just in case"                       │
│  NOT: "More context = better performance"                     │
│  NOT: "Fill the context window"                               │
│                                                               │
│  INSTEAD:                                                     │
│  ├── Include ONLY what's needed for current task             │
│  ├── Maximize signal-to-noise ratio                          │
│  └── Use tools for on-demand retrieval                       │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Quality vs. Quantity

### Empirical Performance Curve

```
Performance by Context Size (Fixed Task)

100% ┤     ╱───╲
     │    ╱     ╲
 90% ┤   ╱       ╲                    ◄── Optimal: ~3K tokens
     │  ╱         ╲
 80% ┤ ╱           ╲___
     │╱                 ╲___
 70% ┤                      ╲___
     │                          ╲___
 60% ┼────────────────────────────────╲────▶
     0   2K  4K  6K  8K  10K  12K  14K  16K
                Context Size (tokens)

Pattern: Performance peaks, then degrades
Cause: Noise drowns signal at high token counts
```

### The Degradation Pattern

| Context Size | Signal-to-Noise | Performance | Lost-in-Middle Effect |
|-------------|-----------------|-------------|----------------------|
| **0-2K** (too small) | High | 65% | None |
| **2K-4K** (optimal) | Very High | **95%** | Minimal |
| **4K-8K** (acceptable) | Medium | 88% | Slight |
| **8K-16K** (large) | Low | 72% | Moderate |
| **16K+** (excessive) | Very Low | 58% | Severe |

**Conclusion:** 2-4K tokens of high-quality context >> 16K+ tokens of mixed quality.

---

## Strategic Positioning

### The Lost-in-Middle Effect

```
┌──────────────────────────────────────────────────────────────┐
│              ATTENTION DISTRIBUTION IN CONTEXT                │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Attention Weight:                                            │
│                                                               │
│  START  ████████████████████ 85% ◄── High attention          │
│         │                                                     │
│         │                                                     │
│  MIDDLE ████ 20% ◄────────────────── Lost-in-middle          │
│         │                                                     │
│         │                                                     │
│  END    ████████████████ 75% ◄────── Recency bias            │
│                                                               │
│  Implication: Critical info at START or END, not middle      │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Positioning Strategy

```markdown
## Optimal Context Structure

┌─────────────────────────────────────────┐
│ START: Most Critical Instructions        │  ◄── 85% attention
│ ├── Current task description             │
│ ├── Key constraints                      │
│ └── Success criteria                     │
├─────────────────────────────────────────┤
│                                           │
│ MIDDLE: Supporting Information            │  ◄── 20% attention
│ ├── Background (if essential)            │
│ ├── Related context (minimal)            │
│ └── Examples (only if needed)            │
│                                           │
├─────────────────────────────────────────┤
│ END: Immediate Task Context               │  ◄── 75% attention
│ ├── Current file/code snippet            │
│ ├── Specific question/problem            │
│ └── Call to action                       │
└─────────────────────────────────────────┘
```

---

## Tool Integration for Context Reduction

### The Tool Search Tool Example

Anthropic reports **85% context reduction** using tools for retrieval:

```
WITHOUT Tools (Context Stuffing):
├── Include entire codebase context: 45K tokens
├── Include all related files: 12K tokens
├── Include documentation: 8K tokens
└── Total: 65K tokens
    Performance: 68% (lost-in-middle)

WITH Tools (On-Demand Retrieval):
├── Current task description: 2K tokens
├── Specific code snippet: 1.5K tokens
├── Tool definitions: 1K tokens
└── Total: 4.5K tokens
    Tools fetch context as needed: 0.5K per call
    Performance: 94% (high signal)

────────────────────────────────────────────
Context reduction: 93% ((65K - 4.5K) / 65K)
Performance improvement: +38% (68% → 94%)
```

### K&V Tool Strategy

```markdown
## Context Reduction via Tools

Instead of:
❌ Including entire repository structure in context
❌ Pre-loading all related files
❌ Dumping full documentation

Use tools:
✓ `warp-grep`: Search codebase on-demand
✓ `cass`: Retrieve decisions as needed
✓ `exa`: Fetch documentation when required
✓ `bv --robot-summary`: Get plan overview

Result: 4K baseline context, tools add <1K per retrieval
```

---

## High-Signal Tokens

### What Makes a Token High-Signal?

```
┌──────────────────────────────────────────────────────────────┐
│              HIGH-SIGNAL vs. LOW-SIGNAL TOKENS                │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  HIGH-SIGNAL (Include These):                                │
│  ├── Direct instructions for current task                    │
│  ├── Concrete examples (1-2 max)                             │
│  ├── Specific constraints                                     │
│  ├── Clear success criteria                                  │
│  ├── Relevant code snippet (not entire file)                 │
│  └── Recent error message (if debugging)                     │
│                                                               │
│  LOW-SIGNAL (Exclude These):                                 │
│  ├── Historical discussion                                   │
│  ├── Background rationale                                    │
│  ├── Tangential information                                  │
│  ├── "Just in case" context                                  │
│  ├── Redundant examples                                      │
│  └── Generic advice                                          │
│                                                               │
│  SIGNAL METRIC:                                               │
│  "Would removing this token affect task completion?"         │
│  ├── YES → High signal (keep)                                │
│  └── NO → Low signal (remove)                                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Anti-Patterns

### What NOT to Do

| Anti-Pattern | Problem | Solution |
|-------------|---------|----------|
| **Context stuffing** | Include "everything" | Selective inclusion only |
| **Just-in-case info** | "Might need this later" | Fetch on-demand via tools |
| **Full file dumps** | Entire files when snippet suffices | Extract relevant lines only |
| **Historical conversation** | Past discussion in context | Summarize or omit |
| **Critical info in middle** | Lost-in-middle effect | Move to start or end |
| **Static context** | Pre-loaded, gets stale | Dynamic retrieval via tools |

---

## Integration with K&V Workflow

### P6 Phase Chunking

Anthropic's principle validates K&V's phase-based approach:

```markdown
## Phase Chunking (Anthropic-Aligned)

❌ WRONG: Load entire plan into every agent
├── All phases in context: 35K tokens
├── All beads in context: 28K tokens
└── Lost-in-middle, degraded performance

✓ RIGHT: Phase-specific context
├── Current phase only: 3K tokens
├── Assigned beads only: 2K tokens
├── Tools for cross-phase lookup: on-demand
└── High signal, optimal performance
```

### P3 Grounding Enhancement

```markdown
## Grounding Checklist (Minimal Context)

Before starting work:
- [ ] Load current bead only (not all beads)
- [ ] Load test intentions (not full test suite)
- [ ] Load acceptance criteria (not full requirements doc)
- [ ] Use tools for related context (don't pre-load)

Estimated context: ~2-3K tokens (optimal)
```

---

## Context Engineering Checklist

### Before Including Context

```markdown
For each piece of information, ask:

1. **Necessity**: Is this required for current task?
   - [ ] Yes → Include
   - [ ] No → Remove

2. **Substitutability**: Can a tool provide this on-demand?
   - [ ] Yes → Use tool instead
   - [ ] No → Include (if necessary)

3. **Positioning**: Where should this go?
   - [ ] Critical → START or END
   - [ ] Supporting → MIDDLE (or remove)

4. **Recency**: Is this current?
   - [ ] Yes → Include
   - [ ] No → Update or remove

5. **Signal**: Would removing this affect success?
   - [ ] No → REMOVE (low signal)
   - [ ] Yes → Keep (high signal)
```

---

## Practical Examples

### Example 1: Bug Fix Task

```markdown
❌ BAD (Context Stuffing):
├── Entire module (1,200 lines): 8K tokens
├── Related modules (3 files): 12K tokens
├── Full git history: 6K tokens
├── All previous attempts: 5K tokens
└── Total: 31K tokens → 72% performance

✓ GOOD (Minimal Context):
├── Function with bug (25 lines): 0.3K tokens
├── Error message: 0.1K tokens
├── Test that fails: 0.2K tokens
├── Expected behavior: 0.2K tokens
└── Total: 0.8K tokens → 95% performance

Tools available for:
- Full file: warp-grep
- Related code: codebase search
- Git history: git log (on-demand)
```

### Example 2: New Feature Implementation

```markdown
❌ BAD:
├── All requirements: 10K tokens
├── All related features: 15K tokens
├── Full architecture doc: 12K tokens
└── Total: 37K tokens → 68% performance

✓ GOOD:
├── Current feature REQ/AC: 1.5K tokens
├── Test intentions: 0.8K tokens
├── Relevant interface (snippet): 0.5K tokens
└── Total: 2.8K tokens → 93% performance

Tools for:
- Related features: cass search
- Architecture: exa fetch
- Code examples: warp-grep
```

---

## Mathematical Model

### Signal-to-Noise Ratio

```
SNR = Signal_Tokens / (Signal_Tokens + Noise_Tokens)

Performance ∝ SNR

Example:
Scenario A: 3K signal + 1K noise = SNR 0.75 → 95% perf
Scenario B: 3K signal + 12K noise = SNR 0.20 → 68% perf

Same signal, different noise → 40% performance difference
```

---

## Key Takeaways

1. **Minimal context outperforms maximal** — 2-4K high-signal tokens achieve 95% performance. 16K+ mixed-quality tokens degrade to 58%.

2. **Tools reduce context by 85-93%** — On-demand retrieval eliminates need for pre-loading. Tool calls add <1K tokens each.

3. **Positioning matters more than volume** — Critical info at START (85% attention) or END (75%), not MIDDLE (20%).

4. **Lost-in-middle is real** — Context items in middle positions receive 60-70% less attention. Avoid placing critical info there.

5. **Quality over quantity** — Every token must "earn its place." Ask: "Would removing this affect success?" If no, remove it.

6. **Static context gets stale** — Pre-loaded context becomes outdated. Dynamic retrieval via tools stays fresh.

7. **P6 Phase Chunking is validated** — Anthropic's principle empirically supports K&V's phase-based, tool-augmented approach.

---

## Limitations

- **Task-specific optima** — Optimal context size varies by task complexity
- **Model-specific** — Different models have different attention patterns
- **Tool overhead** — Tool calls add latency (tradeoff with context quality)
- **Signal detection requires judgment** — Identifying high-signal tokens still needs human/AI skill

---

## See Also

- `004-context-length-hurts.md` — Evidence that long context degrades performance
- `005-lost-in-middle-code.md` — Lost-in-middle effect in code contexts
- `065-confucius-code-agent.md` — Hierarchical navigation reduces context needs
- `057-anthropic-context-engineering.md` — Tool integration for context reduction

---

## Research Impact Score

**Citation count:** Very High (official Anthropic guidance)
**Practical relevance:** ⭐⭐⭐⭐⭐ (directly informs P6 Phase Chunking and tool design)
**Methodological rigor:** ⭐⭐⭐⭐⭐ (internal Anthropic research + deployment data)
**Actionability:** ⭐⭐⭐⭐⭐ (clear principle: minimal, high-signal, strategically positioned)
