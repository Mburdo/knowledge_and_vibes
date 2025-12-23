---
title: Live-SWE-agent - Self-Evolving Agents
tags: [self-evolution, adaptation, tool-creation, runtime, learning, guardrails]
protocols: [P8]
---

# Live-SWE-agent: Self-Evolving Agents

**Paper:** Live-SWE-agent: Can Software Engineering Agents Self-Evolve on the Fly?
**URL:** https://arxiv.org/abs/2511.13646
**Date:** November 2025

---

## Summary

Research introducing **Live-SWE-agent**, the first software agent that can **autonomously and continuously evolve its own scaffold implementation** during runtime while solving real-world problems.

**Key innovation:** Starts with a basic scaffold and improves itself on the fly, unlike static agent architectures.

---

## The Evolution Problem

### Static vs Self-Evolving Agents

| Aspect | Static Agent | Live-SWE-agent |
|--------|--------------|----------------|
| Scaffold | Fixed at design time | Evolves during use |
| Tools | Predefined set | Creates new tools |
| Strategies | Hard-coded | Learns from failures |
| Adaptation | None | Continuous |

### Why Evolution Matters

```
Static Agent Limitation:
├── Encounters novel problem
├── Tries existing strategies
├── All fail
└── Agent stuck (no adaptation)

Live-SWE-agent:
├── Encounters novel problem
├── Tries existing strategies
├── Fails
├── ANALYZES failure
├── CREATES new tool/strategy
├── RETRIES with improvement
└── PERSISTS successful adaptations
```

---

## Architecture

### Core Components

```
┌─────────────────────────────────────────────────────────────────┐
│                     LIVE-SWE-AGENT                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   BASE SCAFFOLD                          │    │
│  │  (minimal: file ops, search, execute)                    │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│  ┌────────────────────────┴────────────────────────────────┐    │
│  │                 EVOLUTION ENGINE                         │    │
│  │  ├── Failure Analyzer                                    │    │
│  │  ├── Strategy Generator                                  │    │
│  │  ├── Tool Creator                                        │    │
│  │  └── Scaffold Updater                                    │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│  ┌────────────────────────┴────────────────────────────────┐    │
│  │                  EVOLVED SCAFFOLD                        │    │
│  │  (base + learned tools + adapted strategies)             │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Evolution Loop

```
1. ATTEMPT: Try to solve problem with current scaffold
2. EVALUATE: Did it work?
   ├── Yes → Move on, scaffold unchanged
   └── No → Continue to step 3
3. ANALYZE: What caused the failure?
   ├── Missing capability?
   ├── Wrong strategy?
   └── Insufficient context?
4. EVOLVE: Generate improvement
   ├── Create new tool
   ├── Modify strategy
   └── Add context retrieval
5. UPDATE: Integrate into scaffold
6. RETRY: Go back to step 1
```

---

## Types of Self-Evolution

### 1. Tool Creation

Agent creates new tools when existing ones insufficient:

```python
# Agent encounters: Need to parse specific log format
# Creates:
def parse_application_logs(log_path: str) -> List[LogEntry]:
    """Parse application-specific log format."""
    entries = []
    with open(log_path) as f:
        for line in f:
            match = re.match(r'\[(\d+)\] (\w+): (.+)', line)
            if match:
                entries.append(LogEntry(
                    timestamp=int(match.group(1)),
                    level=match.group(2),
                    message=match.group(3)
                ))
    return entries

# Tool added to scaffold for future use
```

### 2. Strategy Adaptation

Agent learns better approaches:

```markdown
## Before (Failed Strategy)
1. Search for function by name
2. Read entire file
3. Make changes

## After (Evolved Strategy)
1. Search for function by name
2. Get function boundaries (AST)
3. Read only relevant context (callers, callees)
4. Make targeted changes
5. Verify with type checker
```

### 3. Context Enhancement

Agent learns what context is needed:

```
Initial: Load file containing error
Evolved: Load file + imports + callers + tests + related configs
```

---

## Performance Results

| Benchmark | Base Scaffold | After Evolution | Improvement |
|-----------|---------------|-----------------|-------------|
| SWE-bench | 32% | 47% | +15% |
| Multi-file tasks | 21% | 38% | +17% |
| Novel problems | 15% | 31% | +16% |

### Evolution Statistics

| Metric | Value |
|--------|-------|
| Avg tools created per session | 2.3 |
| Strategy adaptations | 4.1 |
| Evolution overhead | 12% of time |
| Successful evolutions | 67% |

---

## Constraints and Guardrails

### Safe Evolution

Not all evolution is beneficial:

| Constraint | Purpose |
|------------|---------|
| Sandbox execution | New tools can't harm system |
| Rollback capability | Undo harmful evolutions |
| Validation gates | Test new tools before adding |
| Human approval (optional) | Review significant changes |

### Evolution Limits

```
Guardrails:
├── Max tools per session: 10
├── Max strategy changes: 5
├── Evolution attempts per problem: 3
├── Rollback on failure: Automatic
└── Persistence: Only validated evolutions
```

---

## Practical Implications

### For Knowledge & Vibes

Live-SWE-agent's evolution aligns with ADaPT:

| Live-SWE Pattern | K&V Equivalent |
|------------------|----------------|
| Fail → Analyze → Evolve | Fail → ADaPT sub-bead |
| Create tool | Add to tooling manifest |
| Persist learning | CASS memory |
| Rollback on failure | Close without committing |

### Evolution vs ADaPT

```
Live-SWE Evolution (within session):
├── Failure
├── Create new tool/strategy
└── Retry with improvement

ADaPT Decomposition (across sessions):
├── Failure after 3 tries
├── Create sub-bead (new context)
└── Fresh attempt with isolated problem
```

**Complementary:** Evolution for tool/strategy; ADaPT for problem decomposition.

---

## Key Takeaways

1. **Static scaffolds limit agents** — Can't handle novel problems
2. **Evolution enables adaptation** — Learn from failures in real-time
3. **Guardrails essential** — Not all evolution is beneficial
4. **Tool creation is powerful** — Domain-specific tools emerge
5. **Persistence matters** — Successful evolutions should be retained
6. **Overhead is acceptable** — 12% time for 15%+ improvement

---

## Limitations

- Evolution quality depends on base model capability
- Novel tools may have bugs
- Strategy drift possible without validation
- Computational overhead for evolution

---

## See Also

- `038-adapt.md` — Adaptive decomposition pattern
- `001-live-swe-agent.md` — Original Live-SWE research
- `010-swe-agent.md` — Base SWE-agent architecture
