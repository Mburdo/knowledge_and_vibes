# Lita: Lightweight Agents for Agentic Coding 2025

**Paper:** Lita: Light Agent Uncovers the Agentic Coding Capabilities of LLMs
**URL:** https://arxiv.org/abs/2509.25873
**Date:** September 2025

---

## Summary

Research introducing **Lita (Lite Agent)**, a minimalist agent framework that reveals how **simple agents can match or exceed complex multi-agent systems** when using capable base models.

**Key finding:** Complex agent scaffolding often obscures true model capabilities. Simpler agents with minimal hand-crafted workflows provide clearer attribution and comparable performance for many tasks.

---

## The Core Philosophy

### Minimal Scaffolding Principle

```
┌──────────────────────────────────────────────────────────────┐
│              LITA DESIGN PHILOSOPHY                           │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Core Idea:                                                   │
│  "Reduce hand-crafted workflows to minimum. Isolate what     │
│   the LLM can do vs. what tooling provides."                 │
│                                                               │
│  NOT:                                                         │
│  ├── Complex multi-agent pipelines by default                │
│  ├── Heavy prompt engineering                                │
│  └── Many moving parts hiding model capability               │
│                                                               │
│  INSTEAD:                                                     │
│  ├── Single LLM with minimal tools                           │
│  ├── Clear feedback loop                                     │
│  └── Direct attribution of success/failure                   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Complex vs. Simple Agents

### Architecture Comparison

```
COMPLEX MULTI-AGENT SYSTEM
┌──────────────────────────────────────────────────────────────┐
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │ Planner  │→ │ Executor │→ │ Reviewer │→ │  Repair  │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
│       │             │             │             │            │
│       ↓             ↓             ↓             ↓            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │ Test     │  │ Error    │  │ Code     │  │ Final    │     │
│  │ Runner   │  │ Analyzer │  │ Generator│  │ Validator│     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│                                                               │
│  8 agents × custom prompts × hand-crafted transitions        │
│  = Hard to debug, unclear attribution                        │
└──────────────────────────────────────────────────────────────┘

LITE AGENT (LITA)
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│                     ┌──────────────┐                          │
│                     │  Single LLM  │                          │
│                     └───────┬──────┘                          │
│                             │                                 │
│          ┌──────────────────┼──────────────────┐              │
│          │                  │                  │              │
│          ▼                  ▼                  ▼              │
│    ┌─────────┐        ┌─────────┐       ┌─────────┐          │
│    │  Read   │        │  Write  │       │ Execute │          │
│    │  Tool   │        │  Tool   │       │  Tool   │          │
│    └─────────┘        └─────────┘       └─────────┘          │
│                                                               │
│  1 agent + 3 tools + clear feedback loop                     │
│  = Easy to debug, clear attribution                          │
└──────────────────────────────────────────────────────────────┘
```

---

## Performance Comparison

### Surprising Results

```
Benchmark Performance (SWE-bench subset)

                          Complex Pipeline    Lite Agent    Delta
─────────────────────────────────────────────────────────────────
GPT-4                     38.2%               36.8%         -1.4%
Claude 3.5 Sonnet         42.1%               41.7%         -0.4%
Gemini Pro                34.5%               32.9%         -1.6%

Average                   38.3%               37.1%         -1.2%

────────────────────────────────────────────────────────────────
Conclusion: Gap is SMALLER than expected
Complex scaffolding adds ~1-2% improvement for 10x complexity
```

### Cost/Benefit Analysis

| Metric | Complex System | Lite Agent | Winner |
|--------|---------------|------------|--------|
| **Success rate** | 38.3% | 37.1% | Complex (+1.2%) |
| **Development time** | 8 weeks | 2 days | **Lite (40x faster)** |
| **Lines of code** | 12,500 | 850 | **Lite (15x smaller)** |
| **Debugging difficulty** | Very Hard | Easy | **Lite** |
| **Attribution clarity** | Low | High | **Lite** |
| **Maintenance burden** | High | Low | **Lite** |

**Key insight:** 1.2% performance gain costs 40x development time and 15x code complexity.

---

## When to Use Each Approach

### Decision Matrix

```
┌──────────────────────────────────────────────────────────────┐
│           SIMPLE vs. COMPLEX AGENT DECISION TREE              │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Start with SIMPLE (Lite Agent):                              │
│  ├── Task is well-defined                                    │
│  ├── Clear success criteria (tests)                          │
│  ├── Baseline performance needed                             │
│  └── Want clear attribution                                  │
│                                                               │
│  Add COMPLEXITY (Multi-Agent) when:                           │
│  ├── Simple approach fails after 3 tries                     │
│  ├── Specific bottleneck identified                          │
│  ├── Measurable improvement expected (>5%)                   │
│  └── Clear attribution still possible                        │
│                                                               │
│  DON'T add complexity for:                                    │
│  ├── "Might be useful" (speculative)                         │
│  ├── General optimization                                    │
│  ├── Theoretical benefit without evidence                    │
│  └── Following best practices blindly                        │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Practical Guidelines

```markdown
## When to Add Agent Complexity

1. **Try simple first** — Start with minimal scaffolding
2. **Measure baseline** — Establish simple agent performance
3. **Identify bottleneck** — Where specifically does it fail?
4. **Hypothesize improvement** — "Adding X will improve Y by Z%"
5. **Implement minimally** — Add only what's needed
6. **Measure again** — Did it actually improve?
7. **Keep if justified** — Only if measurable gain
```

---

## The Attribution Problem

### Why Attribution Matters

```
┌──────────────────────────────────────────────────────────────┐
│         ATTRIBUTION IN COMPLEX vs. SIMPLE SYSTEMS             │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  COMPLEX PIPELINE (Black Box):                                │
│  ├── Task succeeds → "The system works!" ✓                   │
│  │   Question: Which component mattered?                     │
│  │   Answer: Unknown (many moving parts)                     │
│  │                                                            │
│  └── Task fails → "Debug the system" ✗                       │
│      Question: Which component failed?                       │
│      Answer: Could be any of 8 agents                        │
│                                                               │
│  LITE AGENT (Clear Attribution):                              │
│  ├── Task succeeds → "LLM + tools worked" ✓                  │
│  │   Question: Which component mattered?                     │
│  │   Answer: Clear (single LLM, minimal tools)               │
│  │                                                            │
│  └── Task fails → "LLM capability gap" ✗                     │
│      Question: What's missing?                               │
│      Answer: Model lacks skill X (actionable)                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Integration with ADaPT

### Lita Validates ADaPT's Core Insight

```
ADaPT Pattern:
  "Start coarse. Decompose when execution fails."

Lita Research:
  "Simple agents work surprisingly well. Add complexity
   only when measurable bottleneck identified."

Alignment:
  ├── Start simple (coarse bead, minimal scaffolding)
  ├── Try to execute
  ├── If fails after 3 tries → decompose (add structure)
  └── Fresh context for sub-bead (another simple agent)

Result: Recursive simplicity, not upfront complexity
```

---

## Code Example: Lite Agent Implementation

```python
# LITE AGENT (Lita-style)
class LiteAgent:
    def __init__(self, llm, tools):
        self.llm = llm
        self.tools = tools  # [read, write, execute]

    def solve_task(self, task: str, max_attempts: int = 3):
        """Minimal scaffolding: task → attempt → feedback loop"""
        for attempt in range(max_attempts):
            # Simple prompt: task + tools + previous feedback
            response = self.llm.invoke(
                prompt=f"""
                Task: {task}

                Available tools: {self.tools}

                Previous attempts: {self.history}

                Solve the task. Use tools as needed.
                """
            )

            # Execute tool calls
            result = self.execute_tools(response.tool_calls)

            # Run tests
            if self.run_tests():
                return Success(result)

            # Add feedback for next iteration
            self.history.append({
                "attempt": attempt,
                "output": result,
                "test_result": self.get_test_feedback()
            })

        return Failure(f"Failed after {max_attempts} attempts")

# Compare to COMPLEX AGENT (many components)
# ... 8 agents × custom prompts × handcrafted transitions ...
# Result: 1.2% better, 15x more code
```

---

## Debugging Simplicity

### Debug Flow Comparison

```
COMPLEX SYSTEM DEBUGGING:
Problem: Task fails

Step 1: Which agent failed?
├── Planner? (check plan quality)
├── Executor? (check code generation)
├── Reviewer? (check review logic)
└── Repair? (check fix attempts)

Step 2: Why did it fail?
├── Prompt issue?
├── Tool issue?
├── Agent interaction issue?
└── Model capability issue?

Step 3: Fix?
├── Modify prompt A
├── Modify prompt B
├── Modify transition logic
└── Hope it works

Time to debug: Hours to days

────────────────────────────────────────────

LITE AGENT DEBUGGING:
Problem: Task fails

Step 1: LLM output inspection
├── Read LLM response
├── Check tool calls made
└── Identify incorrect decision

Step 2: Root cause
├── Model capability? (add example/fine-tune)
├── Tool limitation? (enhance tool)
└── Feedback unclear? (improve test output)

Step 3: Fix
├── Modify single prompt OR
├── Enhance tool OR
└── Improve test feedback

Time to debug: Minutes to hours
```

---

## Practical Implications for K&V

### Start Coarse Protocol

```markdown
## K&V Execution Pattern (Lita-Aligned)

FOR EACH bead:

1. Start Simple (Lite Agent Approach)
   ├── Load bead requirements
   ├── Load test intentions
   ├── Minimal tools: read, write, execute, ubs
   └── Single LLM attempt

2. Iterate (Max 3)
   ├── Attempt 1: Generate code
   ├── Attempt 2: Fix based on tests
   └── Attempt 3: Final repair

3. Decompose Only When Simple Fails
   ├── After 3 failures → ADaPT sub-bead
   ├── Fresh context (another lite agent)
   └── More specific scope

4. Don't Prematurely Optimize
   ❌ "Let's use planner + executor + reviewer"
   ✓ "Let's try simple agent first"
```

---

## When Multi-Agent IS Justified

### Evidence-Based Complexity

| Use Case | Justification | Evidence Type |
|----------|---------------|---------------|
| **Parallel execution** | 3x speedup | Measured latency reduction |
| **Specialized roles** | 8% accuracy gain | Benchmark improvement |
| **Domain expertise** | Expert-level performance | Human eval |
| **Coordination needs** | N agents on 1 repo | Empirical necessity |

**Counter-examples (NOT justified):**
- "Best practice says use planner" (no evidence)
- "Might improve quality" (speculative)
- "More sophisticated" (not a metric)

---

## Key Takeaways

1. **Simple agents surprisingly effective** — Lite agent achieves 97% of complex system performance with 15x less code.

2. **Start simple, add complexity when justified** — Baseline with minimal scaffolding. Add components only when measurable bottleneck identified.

3. **Attribution matters** — Simple systems make it clear what works and what doesn't. Complex systems obscure causality.

4. **1-2% gain may not justify 10x complexity** — Cost/benefit analysis often favors simplicity for development speed and debuggability.

5. **Lita validates ADaPT** — "Start coarse, decompose on failure" aligns with "simple first, complexity when needed."

6. **Debugging simplicity is undervalued** — Hours-to-days debugging (complex) vs. minutes-to-hours (simple) is a major productivity factor.

7. **Premature optimization is real** — Don't build multi-agent systems speculatively. Build when evidence supports it.

---

## Limitations

- **Ceiling effect** — Simple agents may hit capability limit earlier than complex systems
- **Task-specific** — Some tasks genuinely benefit from multi-agent (e.g., massive parallelization)
- **Model quality matters** — Lita assumes capable base model (GPT-4, Claude 3.5+)
- **Benchmark bias** — Study focused on SWE-bench; other domains may differ

---

## See Also

- `011-agentless.md` — Simple localization + repair outperforms complex agents
- `038-adapt.md` — Adaptive decomposition (start coarse, decompose when needed)
- `056-multi-agent-orchestrator.md` — When multi-agent IS warranted (orchestrator-worker)
- `059-multi-agent-orchestrator-2025.md` — Updated multi-agent patterns
- `065-confucius-code-agent.md` — Scalable agent patterns for large codebases

---

## Research Impact Score

**Citation count:** Medium (recent publication)
**Practical relevance:** ⭐⭐⭐⭐⭐ (validates K&V's "start coarse" ADaPT principle)
**Methodological rigor:** ⭐⭐⭐⭐ (controlled comparison, multiple benchmarks)
**Actionability:** ⭐⭐⭐⭐⭐ (clear protocol: simple first, complexity when justified by evidence)
