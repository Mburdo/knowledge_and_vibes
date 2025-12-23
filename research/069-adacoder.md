---
title: AdaCoder - Adaptive Planning Multi-Agent Framework
tags: [adaptive, multi-agent, model-profiling, decomposition, generalizability, planning]
protocols: [P3, P10]
---

# AdaCoder: Adaptive Planning Multi-Agent Framework

**Paper:** AdaCoder: An Adaptive Planning and Multi-Agent Framework for Function-Level Code Generation
**URL:** https://arxiv.org/abs/2504.04220
**Date:** April 5, 2025

---

## Summary

Research introducing **AdaCoder**, a novel adaptive planning, multi-agent framework for function-level code generation. Empirically studies the generalizability of existing multi-agent frameworks across various open-source LLMs.

**Key finding:** Existing multi-agent frameworks often don't generalize across different LLMs. AdaCoder adapts its approach based on the underlying model.

---

## The Generalizability Problem

### Existing Frameworks Fail to Generalize

| Framework | Works Best With | Fails With |
|-----------|-----------------|------------|
| MapCoder | GPT-4 | Smaller models |
| AgentCoder | Claude | Open-source |
| Self-Refine | Strong models | Weak models |

**Root cause:** Frameworks are tuned for specific model capabilities.

### AdaCoder's Insight

Different models need different:
- **Decomposition granularity** — Weak models need smaller steps
- **Feedback formats** — Some models respond better to certain prompts
- **Iteration limits** — Model-specific decay rates
- **Agent roles** — Not all models can play all roles

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        ADACODER                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                 MODEL PROFILER                           │    │
│  │  ├── Capability assessment                               │    │
│  │  ├── Optimal decomposition level                         │    │
│  │  └── Best feedback format                                │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              ADAPTIVE PLANNER                            │    │
│  │  ├── If strong model → Coarse decomposition              │    │
│  │  ├── If weak model → Fine decomposition                  │    │
│  │  └── Adjusts based on task complexity                    │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              MULTI-AGENT EXECUTOR                        │    │
│  │  ├── Coder Agent (generates code)                        │    │
│  │  ├── Tester Agent (writes tests)                         │    │
│  │  ├── Reviewer Agent (validates)                          │    │
│  │  └── Coordinator (manages flow)                          │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              ADAPTIVE FEEDBACK                           │    │
│  │  ├── Error format tuned to model                         │    │
│  │  ├── Iteration limit based on model decay                │    │
│  │  └── Escalation when model-specific limit reached        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Model Profiling

### Capability Assessment

Before execution, AdaCoder profiles the model:

```python
def profile_model(model: LLM) -> ModelProfile:
    # Test on calibration set
    results = []
    for task in calibration_tasks:
        result = model.generate(task.prompt)
        results.append(evaluate(result, task.expected))

    return ModelProfile(
        code_quality=avg_code_quality(results),
        decomposition_capability=test_decomposition(model),
        self_repair_capability=test_repair(model),
        decay_rate=measure_decay(model),  # DDI-style
        optimal_feedback_format=test_feedback_formats(model)
    )
```

### Profile-Based Adaptation

| Model Capability | Decomposition | Iteration Limit | Agent Roles |
|------------------|---------------|-----------------|-------------|
| High (GPT-4 class) | Coarse (2-3 steps) | 3 | All agents |
| Medium (GPT-3.5 class) | Medium (4-6 steps) | 2 | Coder + Tester |
| Low (small open-source) | Fine (8-10 steps) | 1 | Coder only |

---

## Adaptive Planning

### Granularity Selection

```python
def plan_task(task: str, profile: ModelProfile) -> List[Step]:
    complexity = estimate_complexity(task)

    if profile.decomposition_capability == "high" and complexity < 3:
        # Strong model, simple task: minimal decomposition
        return [Step(task)]

    elif profile.decomposition_capability == "medium":
        # Medium model: standard decomposition
        return decompose_standard(task, max_steps=5)

    else:
        # Weak model: maximum decomposition
        return decompose_fine(task, max_steps=10)
```

### Complexity Estimation

| Complexity Level | Indicators | Decomposition |
|------------------|------------|---------------|
| Low (1-2) | Single function, clear spec | Minimal |
| Medium (3-5) | Multiple functions, some ambiguity | Standard |
| High (6+) | Multi-file, complex logic | Maximum |

---

## Adaptive Feedback

### Model-Specific Error Formatting

```python
def format_error(error: Exception, profile: ModelProfile) -> str:
    base_msg = f"Error: {error.type}\n{error.message}"

    if profile.optimal_feedback_format == "detailed":
        return f"""
{base_msg}

Stack trace:
{error.traceback}

Suggested fix areas:
{analyze_error_location(error)}

Similar successful fixes:
{retrieve_similar_fixes(error)}
"""

    elif profile.optimal_feedback_format == "concise":
        return f"{base_msg}\nFix the {error.type} at line {error.line}"

    else:  # minimal
        return f"Fix: {error.message}"
```

### Adaptive Iteration Limits

Based on model-specific DDI:

```python
def should_continue(attempt: int, profile: ModelProfile) -> bool:
    model_limit = {
        "high": 3,    # Strong models: 3 attempts
        "medium": 2,  # Medium models: 2 attempts
        "low": 1      # Weak models: 1 attempt (then decompose)
    }
    return attempt < model_limit[profile.self_repair_capability]
```

---

## Performance Results

### Cross-Model Evaluation

| Model | MapCoder | AgentCoder | AdaCoder |
|-------|----------|------------|----------|
| GPT-4 | 71% | 68% | 72% |
| Claude 3 | 65% | 69% | 70% |
| Llama 70B | 48% | 45% | 58% |
| Mistral 7B | 31% | 28% | 42% |
| CodeLlama 7B | 27% | 24% | 38% |

**Key insight:** Biggest gains on smaller models (+10-14%).

### Why AdaCoder Helps Weak Models

| Adaptation | Impact on Weak Models |
|------------|----------------------|
| Finer decomposition | Simpler sub-tasks |
| Single attempt | Avoids harmful iteration |
| Concise feedback | Less context pollution |
| Fewer agents | Simpler coordination |

---

## Practical Implications

### For Knowledge & Vibes

AdaCoder validates adaptive approaches:

| AdaCoder Pattern | K&V Application |
|------------------|-----------------|
| Model profiling | Select agent type based on task |
| Adaptive decomposition | ADaPT pattern |
| Iteration limits | 3-try rule, model-adjusted |
| Feedback formatting | Structured error messages |

### Model-Aware Agent Design

```markdown
## Agent Assignment Based on Task + Model

| Task Complexity | Strong Model | Weak Model |
|-----------------|--------------|------------|
| Simple fix | Single agent | Single agent |
| Feature add | Multi-agent | More decomposition |
| Refactor | Multi-agent | Human oversight |
```

---

## Key Takeaways

1. **One size doesn't fit all** — Frameworks must adapt to model capability
2. **Profile before execution** — Know your model's strengths
3. **Decomposition is model-dependent** — Weak models need finer steps
4. **Iteration limits vary** — Model-specific decay rates
5. **Biggest gains on weak models** — Adaptation helps most where needed

---

## See Also

- `040-mapcoder.md` — Fixed multi-agent approach
- `038-adapt.md` — Adaptive decomposition
- `060-debugging-decay-index.md` — Model-specific decay
