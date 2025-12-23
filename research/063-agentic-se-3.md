---
title: Agentic Software Engineering (SE 3.0) 2025
tags: [agentic-se, se-3.0, trust, autonomous, orchestration, framework]
protocols: [P10, P14]
---

# Agentic Software Engineering (SE 3.0) 2025

**Sources:**
- "Agentic Software Engineering: Foundational Pillars and a Research Roadmap" (September 2025)
- "The Rise of AI Teammates in Software Engineering (SE) 3.0" (July 2025)
- "Agentic AI Software Engineers: Programming with Trust" (September 2025)
- AIDev Dataset: Real-world operations of Codex, Devin, Copilot, Cursor, Claude Code

---

## Summary

2025 research defining **Agentic Software Engineering (SE 3.0)** — the transition from LLMs as code generators to LLMs as autonomous software engineering agents.

**Key insight:** The shift from "programming at scale" to "programming with trust."

---

## The Three Eras of Software Engineering

| Era | Paradigm | Human Role | AI Role |
|-----|----------|------------|---------|
| **SE 1.0** | Manual | All tasks | None |
| **SE 2.0** | Assisted | Decision + oversight | Completion, suggestions |
| **SE 3.0** | Agentic | Goals + trust | Autonomous execution |

### SE 3.0 Characteristics

```
SE 2.0 (Assistive):
├── Human writes code
├── AI suggests completions
├── Human reviews suggestions
└── Human makes decisions

SE 3.0 (Agentic):
├── Human sets goals
├── AI plans approach
├── AI implements solution
├── AI tests and iterates
├── Human validates outcomes
└── Trust determines autonomy level
```

---

## Structured Agentic Software Engineering (SASE)

From the research roadmap:

### The Duality Principle

```
┌─────────────────────────────────────────────────────────────────┐
│                    SASE FRAMEWORK                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SE for Humans          ←→          SE for Agents                │
│  ──────────────                     ────────────────             │
│  • Human-readable docs              • Machine-parseable specs    │
│  • IDE interfaces                   • API-first tools            │
│  • Manual workflows                 • Automated pipelines        │
│  • Intuitive conventions            • Explicit protocols         │
│                                                                  │
│  Must reimagine foundational pillars for agent compatibility     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Four Pillars of SASE

| Pillar | Human-Oriented | Agent-Oriented |
|--------|----------------|----------------|
| **Actors** | Developers, testers, PMs | Specialized agents, orchestrators |
| **Processes** | Agile, waterfall, kanban | ADaPT, TDD-loops, calibration |
| **Tools** | IDEs, debuggers, profilers | APIs, MCP, structured outputs |
| **Artifacts** | Docs, code, tests | Beads, traces, verification evidence |

---

## Trust as the Central Challenge

### The Trust Taxonomy

From "Programming with Trust":

```
Trust Levels:
├── L0: No Trust (human writes all code)
├── L1: Verify Everything (human reviews every line)
├── L2: Spot Check (human samples outputs)
├── L3: Exception-Based (human handles edge cases)
├── L4: Goal-Directed (human sets goals, validates outcomes)
└── L5: Full Autonomy (human approves releases only)

Current State (2025): Most teams at L1-L2
Target State: L3-L4 for routine tasks
```

### Trust-Building Mechanisms

| Mechanism | How It Builds Trust |
|-----------|---------------------|
| **TDD** | Tests prove correctness |
| **Security scanning** | Catches vulnerabilities |
| **Traceability** | Links requirements to implementation |
| **Calibration** | Detects drift early |
| **Evidence artifacts** | Auditable decision trail |

---

## AIDev Dataset Findings

Real-world analysis of 5 leading agents across thousands of repositories:

### Agent Capabilities Compared

| Capability | Codex | Devin | Copilot | Cursor | Claude Code |
|------------|-------|-------|---------|--------|-------------|
| Code generation | High | High | High | High | High |
| Multi-file editing | Medium | High | Medium | High | High |
| Test writing | Medium | High | Medium | Medium | High |
| Debugging | Low | Medium | Low | Medium | High |
| Planning | Low | High | Low | Medium | High |

### Common Failure Modes

| Failure Mode | Frequency | Root Cause |
|--------------|-----------|------------|
| Context loss | High | Token limits, no memory |
| Hallucinated APIs | Medium | Training data gaps |
| Incomplete fixes | Medium | Debugging decay |
| Convention drift | Medium | Missing codebase context |
| Security issues | Medium | Training on insecure code |

---

## Six-Layer Vision Framework

From the 2025 research roadmap:

```
┌─────────────────────────────────────────────────────────────────┐
│                   SIX-LAYER FRAMEWORK                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Layer 1: INPUT                                                  │
│  ├── Requirements (REQ-*, AC-*)                                  │
│  ├── Context (codebase, history)                                 │
│  └── Constraints (security, performance)                         │
│                                                                  │
│  Layer 2: ORCHESTRATION                                          │
│  ├── Planning (phases, beads)                                    │
│  ├── Decomposition (ADaPT)                                       │
│  └── Coordination (Agent Mail)                                   │
│                                                                  │
│  Layer 3: DEVELOPMENT                                            │
│  ├── Code generation                                             │
│  ├── TDD cycles                                                  │
│  └── Security scanning                                           │
│                                                                  │
│  Layer 4: VALIDATION                                             │
│  ├── Tests (unit, integration)                                   │
│  ├── Verification (ubs, type checks)                             │
│  └── Review (calibration)                                        │
│                                                                  │
│  Layer 5: SYNTHESIS                                              │
│  ├── Integration (merge tracks)                                  │
│  ├── Traceability (REQ → code → test)                           │
│  └── Documentation                                               │
│                                                                  │
│  Layer 6: REFINEMENT & DEBUG                                     │
│  ├── ADaPT sub-beads                                             │
│  ├── Fresh context resets                                        │
│  └── Human escalation                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implications for Knowledge & Vibes

### Alignment with SE 3.0

| SE 3.0 Principle | K&V Implementation |
|------------------|-------------------|
| Goal-directed | North Star Card |
| Trust via verification | TDD + ubs + calibration |
| Agent coordination | Agent Mail + orchestrator |
| Structured artifacts | Beads + phases + traces |
| Adaptive execution | ADaPT pattern |
| Evidence-based | Research backing for all protocols |

### Full Pipeline as SE 3.0 Instance

```
Knowledge & Vibes Full Pipeline = SE 3.0 Implementation

Stage 0-2: INPUT LAYER
├── North Star → Goals
├── Requirements → REQ-*/AC-*
└── QA → Constraint refinement

Stage 3-5: ORCHESTRATION LAYER
├── Decision Search → ADRs
├── Risk Assessment → Constraints
└── Spikes → Uncertainty reduction

Stage 6-7: ORCHESTRATION + DEVELOPMENT PREP
├── Plan Assembly → Phase structure
└── Decomposition → Beads

Stage 8: DEVELOPMENT + VALIDATION LAYERS
├── Parallel execution → 10+ agents
├── TDD cycles → Tests first
├── Security → ubs scanning
└── Coordination → Agent Mail

Stage 9: SYNTHESIS LAYER
├── Calibration → Phase boundary check
└── Traceability → REQ → bead → test

Stage 10: OUTPUT
└── Release with evidence
```

---

## Future Directions (from 2025 research)

### Near-term (2025-2026)

- Agent-native IDEs
- Standardized agent protocols (MCP expansion)
- Trust certification frameworks
- Shared agent memory systems

### Medium-term (2026-2028)

- Cross-project agent learning
- Autonomous refactoring at scale
- Self-improving agent architectures
- Formal verification integration

---

## Key Takeaways

1. **SE 3.0 is happening now** — Agents are teammates, not just tools
2. **Trust is the bottleneck** — Technical capability exceeds human trust
3. **Verification builds trust** — TDD, security scans, calibration
4. **Duality is essential** — Tools must work for both humans and agents
5. **Evidence over rhetoric** — Tests adjudicate, not debates
6. **Structured processes help** — ADaPT, phases, beads enable agent success

---

## See Also

- `056-multi-agent-orchestrator.md` — Orchestrator patterns
- `038-adapt.md` — Adaptive decomposition
- `054-tdd-ai-code-gen.md` — TDD as trust mechanism
- `docs/workflow/PHILOSOPHY.md` — K&V's SE 3.0 philosophy
