# Debate or Vote: Multi-Agent Decision Making

**Paper:** Debate or Vote: Which Yields Better Decisions in Multi-Agent Large Language Models?
**URL:** https://arxiv.org/abs/2508.17536
**Date:** August 2025

---

## Summary

Research separating multi-agent debate (MAD) into two components: majority voting and inter-agent debate.

**Critical finding:** Voting alone accounts for most performance gains. In many cases, pure voting outperforms debate.

---

## The Core Finding

### Debate is a Martingale

The paper proves mathematically that standard debate forms a **martingale**: a process where the expected value doesn't change. Agents randomly influence each other; on average, the effect is neutral.

```
What happens in unstructured debate:
────────────────────────────────────────────────────
Round 1: Agent A proposes X, Agent B proposes Y
Round 2: A argues for X, B argues for Y
Round 3: A influenced by B, changes to Y
Round 4: B influenced by A's original point, changes to X
...
Net effect: Random walk, no convergence to truth
────────────────────────────────────────────────────
```

---

## Performance Results

### Qwen2.5-7B Across 7 Benchmarks

| Method | Average Accuracy |
|--------|------------------|
| Single agent | 0.7205 |
| **Majority Voting** | **0.7691** |
| Debate T=2 | 0.7377 |
| Debate T=3 | 0.7112 |

```
Performance Comparison:
────────────────────────────────────────────────────────────
Single Agent     ████████████████████████████████████  72.05%
Majority Voting  ██████████████████████████████████████████  76.91%
Debate (2 turns) ████████████████████████████████████████  73.77%
Debate (3 turns) ████████████████████████████████████  71.12%
────────────────────────────────────────────────────────────
```

**Key observation:** More debate rounds made performance *worse*.

---

## Why Debate Fails

### The Random Walk Problem

```
┌─────────────────────────────────────────────────────────────────┐
│                  UNSTRUCTURED DEBATE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Agent A: "The answer is X because..."                           │
│  Agent B: "I disagree, Y because..."                             │
│  Agent A: "Hmm, B makes a point, maybe Y..."                     │
│  Agent B: "Actually, A's original reasoning..."                  │
│                                                                  │
│  Result: Agents trade positions, no net improvement              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Root Causes

| Problem | Effect |
|---------|--------|
| No evidence grounding | Arguments based on priors |
| Identical agents | Same biases, same errors |
| No anchor | Nothing prevents drift |
| Symmetric influence | Each agent equally likely to change |

---

## What Does Help

### Biased Debate Strategies

Breaking the random-walk property requires asymmetric influence:

| Strategy | How It Works | Result |
|----------|--------------|--------|
| MAD-Conformist | Agents keep answers matching majority | Breaks symmetry |
| MAD-Follower | 30% chance to adopt majority | Soft bias toward consensus |

### Why Bias Helps

```
Standard Debate:
Agent A: 50% chance to change
Agent B: 50% chance to change
Net: Random

Biased Debate (Conformist):
Agent A: If with majority, stay
Agent B: If with majority, stay
Minority agent: More likely to change
Net: Convergence toward majority
```

---

## Important Context

### What the Paper Tested

| Condition | Description |
|-----------|-------------|
| Tasks | QA benchmarks (short answers) |
| Agents | Identical LLMs |
| Roles | None (all equivalent) |
| Evidence | None (arguments from priors) |
| Structure | Freeform debate |

### What It Didn't Test

- Specialized agents with distinct roles
- Evidence-grounded arguments
- Structured challenge/response protocols
- Shared context documents

---

## Practical Implications

### For Knowledge & Vibes Calibration

The martingale result may not apply because K&V calibration is fundamentally different:

| Paper's Debate | K&V Calibration |
|----------------|-----------------|
| Identical agents | Specialized roles |
| No evidence | Evidence required |
| Freeform argument | Structured protocol |
| No shared context | Shared plan/tests |
| Rhetoric-based | Test-based adjudication |

### What to Take Away

1. **Voting is a strong baseline** - Multiple agents voting often beats debate
2. **Unstructured debate is neutral** - Random back-and-forth doesn't converge
3. **Structure and evidence matter** - Bias or anchor the debate
4. **Tests beat rhetoric** - Use test results, not arguments

### Applying to Calibration

```markdown
## How K&V Calibration Differs

1. Agents have distinct roles (coordinator, executor, etc.)
2. Claims must cite evidence (code, tests, docs)
3. Structured format (challenge → evidence → resolution)
4. Shared plan provides anchor
5. Disagreements resolved by running tests, not arguing

The paper's findings argue FOR this design:
- Don't rely on freeform debate
- Require evidence for claims
- Use tests to adjudicate
```

---

## Key Takeaways

1. **Voting often beats debate** - Simple majority vote is effective
2. **More debate can hurt** - 3 rounds worse than 2 rounds
3. **Debate is mathematically neutral** - Martingale property
4. **Bias helps** - Lock in verified claims, weight toward evidence
5. **Evidence grounding is critical** - Without it, debate is random walk

---

## See Also

- `038-adapt.md` - Adaptive decomposition pattern
- `059-multi-agent-orchestrator-2025.md` - Multi-agent coordination
- `006-dark-side-self-correction.md` - Limits of self-correction
