---
title: Live-SWE-agent - Dynamic Tool Creation
tags: [agents, tool-creation, reflection, swe-bench, self-evolution]
protocols: [P8]
---

# Live-SWE-agent: Dynamic Tool Creation

**Paper:** Live-SWE-agent: Can Software Engineering Agents Self-Evolve on the Fly?
**URL:** https://arxiv.org/abs/2511.13646
**Date:** November 2025

---

## Summary

Live-SWE-agent achieves **77.4% on SWE-bench Verified** by allowing agents to create custom tools during problem-solving. The agent starts with minimal capabilities (bash only) and writes helper scripts as needed.

**Critical insight:** The "self-evolution" is per-task tool creation, not accumulated learning across tasks.

---

## What "Self-Evolution" Actually Means

### Not What You Might Think

```
What it sounds like:           What it actually is:
─────────────────────          ──────────────────────
Agent improves over time       Agent creates scripts per-task
Knowledge persists             Tools are NOT carried forward
Core agent learns              Core agent loop stays fixed
```

### The Real Mechanism

1. Agent encounters a problem
2. Realizes existing tools are insufficient
3. Writes a Python/bash helper script
4. Uses the script for this task only
5. Next task: starts fresh, may recreate similar tools

---

## Performance Results

### SWE-Bench Benchmarks

| Benchmark | Score | Context |
|-----------|-------|---------|
| SWE-bench Verified | 77.4% | Standard benchmark |
| SWE-Bench Pro | 45.8% | Harder, more realistic tasks |

### Ablation Study (50-problem subset)

| Condition | Solve Rate | Delta |
|-----------|------------|-------|
| Full system | 76.0% | baseline |
| Without tool creation | 62.0% | -14% |
| Without reflection | 64.0% | -12% |

```
Contribution to Success:
─────────────────────────────────────────────
Tool Creation  ████████████████████  +14%
Reflection     ████████████████      +12%
─────────────────────────────────────────────
```

---

## How Tool Creation Works

### The Reflection Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                    LIVE-SWE-AGENT LOOP                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Attempt task with existing tools                            │
│     │                                                           │
│     ├── Success? → Done                                         │
│     └── Struggle? → Continue                                    │
│                                                                  │
│  2. Reflection: "Would creating a tool help?"                   │
│     │                                                           │
│     ├── No → Try different approach                             │
│     └── Yes → Create helper script                              │
│                                                                  │
│  3. Write and use custom tool                                   │
│     │                                                           │
│     └── Retry task with new capability                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Types of Tools Created

| Tool Type | Example | Purpose |
|-----------|---------|---------|
| File editors | Custom search-replace | Handle complex edits |
| Search utilities | Semantic code search | Find relevant code |
| Language analyzers | AST parsers | Understand code structure |
| Test runners | Custom test harness | Verify changes |

---

## Why Tool Creation Helps

### The Capability Gap

Standard agents have fixed tool sets. When a problem requires a capability they lack, they improvise poorly:

```
Without Tool Creation:
─────────────────────
Problem: "Find all usages of deprecated API"
Agent: Uses grep, misses dynamic calls
Result: Incomplete fix

With Tool Creation:
─────────────────────
Problem: "Find all usages of deprecated API"
Agent: Writes AST-based analyzer
Result: Finds all usages including dynamic
```

### Model Requirements

| Model Class | Performance | Notes |
|-------------|-------------|-------|
| GPT-5 class | Works well | Full benefit |
| GPT-4 class | Works | Reduced benefit |
| GPT-5-Nano | Fails | Infinite loops, worse than baseline |

**Key finding:** Weaker models entered infinite loops trying to create tools, performing worse than no tool creation at all.

---

## Practical Implications

### For Knowledge & Vibes

| Finding | Implication |
|---------|-------------|
| Tool creation adds 14% | Allow agents to create helper scripts |
| Core loop stays fixed | Phase templates and planning structure unaffected |
| Reflection prompts help | Ask "would a tool help?" when stuck |
| Weak models fail | Only use with capable models |

### Integration Approach

```markdown
## When Agents Get Stuck

1. Standard approach: Try different strategy
2. If pattern repeats: Consider tool creation
3. Reflection prompt: "Would writing a helper script solve this?"
4. If yes: Create targeted script for this task
5. Important: Don't expect script to persist
```

### What This Doesn't Change

- Phase templates and planning structure
- Bead decomposition patterns
- Multi-agent coordination
- Verification requirements

The paper is about runtime tool creation during execution, not planning artifacts or workflow structure.

---

## Key Takeaways

1. **Dynamic tool creation works** - 14% improvement on SWE-bench
2. **Reflection is key** - "Would a tool help?" prompt matters
3. **Tools are ephemeral** - Don't persist between tasks
4. **Model capability matters** - Weak models do worse with this
5. **Core workflow unchanged** - Only execution strategy affected

---

## See Also

- `067-live-swe-agent-evolution.md` - Later research on this approach
- `007-swe-gym.md` - SWE-bench training approaches
- `010-swe-agent.md` - Original SWE-agent
