# TDAG: Dynamic Task Decomposition and Agent Generation

**Paper:** TDAG: A Multi-Agent Framework based on Dynamic Task Decomposition and Agent Generation
**URL:** https://arxiv.org/abs/2402.10178
**Date:** February 2024
**Venue:** arXiv preprint (later journal version)

---

## Summary

Multi-agent framework where the system dynamically decomposes complex tasks into subtasks and generates specialized sub-agents for each subtask on demand. Task decomposition can be revised based on intermediate outcomes to reduce error propagation. Includes skill accumulation library for reuse.

**Key finding:** Dynamic decomposition (adjust task graph during execution) outperforms static decomposition by **15-22%** on complex multi-step tasks, as it adapts to **actual execution feedback** rather than upfront guesses.

---

## The Core Problem

### Static Decomposition Fails

```
┌─────────────────────────────────────────────────────────────────┐
│             STATIC PLAN-THEN-EXECUTE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Complex Task: "Analyze sales data and generate report"          │
│         │                                                        │
│         ▼                                                        │
│  PLAN (upfront, no execution feedback):                          │
│  ├─ Subtask 1: Download data                                     │
│  ├─ Subtask 2: Clean data                                        │
│  ├─ Subtask 3: Analyze trends                                    │
│  ├─ Subtask 4: Generate visualizations                           │
│  └─ Subtask 5: Write report                                      │
│         │                                                        │
│         ▼                                                        │
│  EXECUTE (rigidly follow plan):                                  │
│  ├─ Subtask 1: ✓ Downloaded                                      │
│  ├─ Subtask 2: ✗ FAIL - Data format unexpected                   │
│  │                                                               │
│  │  PROBLEM: Plan assumed clean CSV                              │
│  │  REALITY: Data is nested JSON with inconsistent schema        │
│  │                                                               │
│  │  Static plan can't adapt →                                    │
│  │  Either fail entirely OR                                      │
│  │  Force subtask 2 to handle (causing downstream errors)        │
│  │                                                               │
│  └─ Plan is now invalid but can't revise                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Core issue:** The correct decomposition is often **unknowable** until execution reveals constraints.

---

## The TDAG Solution

### Dynamic Task Graph

```
┌─────────────────────────────────────────────────────────────────┐
│                  TDAG ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DECOMPOSITION PHASE                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Task: "Analyze sales data and generate report"            │ │
│  │        │                                                    │ │
│  │        ▼                                                    │ │
│  │  Initial Decomposition (coarse):                            │ │
│  │  ├─ T1: Acquire data                                        │ │
│  │  ├─ T2: Process data                                        │ │
│  │  ├─ T3: Analyze                                             │ │
│  │  └─ T4: Report                                              │ │
│  │                                                             │ │
│  └──────────────────────┬──────────────────────────────────────┘ │
│                         │                                        │
│  AGENT GENERATION                                                │
│  ┌──────────────────────▼─────────────────────────────────────┐ │
│  │                                                             │ │
│  │  For T1: Generate "DataAcquisitionAgent"                    │ │
│  │    Specialized for: API calls, file I/O, authentication     │ │
│  │    Tools: requests, boto3, pandas                           │ │
│  │                                                             │ │
│  └──────────────────────┬──────────────────────────────────────┘ │
│                         │                                        │
│  EXECUTION + FEEDBACK                                            │
│  ┌──────────────────────▼─────────────────────────────────────┐ │
│  │                                                             │ │
│  │  T1 executes: ✓ Data acquired                               │ │
│  │    Outcome: Data is nested JSON (unexpected)                │ │
│  │    Metadata: {format: "json", nested: true, size: "2GB"}    │ │
│  │        │                                                    │ │
│  │        ▼                                                    │ │
│  │  DYNAMIC RE-DECOMPOSITION:                                  │ │
│  │                                                             │ │
│  │  T2 (Process data) NOW decomposes into:                     │ │
│  │  ├─ T2.1: Flatten JSON structure                            │ │
│  │  ├─ T2.2: Handle missing fields                             │ │
│  │  ├─ T2.3: Chunk large file (2GB)                            │ │
│  │  └─ T2.4: Validate schema                                   │ │
│  │        │                                                    │ │
│  │        ▼                                                    │ │
│  │  Generate specialized agents for each:                      │ │
│  │  - JSONFlattenerAgent                                       │ │
│  │  - DataCleaningAgent                                        │ │
│  │  - ChunkProcessorAgent                                      │ │
│  │  - SchemaValidatorAgent                                     │ │
│  │                                                             │ │
│  └──────────────────────┬──────────────────────────────────────┘ │
│                         │                                        │
│  SKILL LIBRARY                                                   │
│  ┌──────────────────────▼─────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Save successful sub-agent solutions:                       │ │
│  │  "JSONFlattenerAgent" → skill library                       │ │
│  │                                                             │ │
│  │  Next time: Reuse instead of regenerating                   │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Innovations

### 1. Dynamic Decomposition

**Static approach:**
```
Plan all subtasks upfront → Execute rigidly
```

**TDAG approach:**
```
Execute → Observe outcome → Decompose if needed → Repeat
```

**Example:**
```python
class TaskGraph:
    def decompose(self, task, execution_context=None):
        """
        Decompose task based on both:
        1. Task description (what we know upfront)
        2. Execution context (what we learned from attempts)
        """
        if execution_context is None:
            # First decomposition: coarse-grained
            return self.initial_decomposition(task)
        else:
            # Re-decomposition: informed by actual execution
            return self.adaptive_decomposition(task, execution_context)

    def adaptive_decomposition(self, task, context):
        """
        Use execution outcomes to refine decomposition
        """
        prompt = f"""
        Task: {task.description}

        Execution attempted with outcome:
        {context.outcome}

        Issues encountered:
        {context.issues}

        Decompose this task into subtasks that specifically address
        the issues encountered. Consider:
        - Data format: {context.metadata.format}
        - Data size: {context.metadata.size}
        - Failure points: {context.failure_points}
        """

        return llm_decompose(prompt)
```

### 2. Agent Generation (Not Reuse)

**Traditional multi-agent:**
```
Pre-defined agents: [Planner, Coder, Tester, Reviewer]
→ Map task to existing agent
```

**TDAG:**
```
For each subtask → Generate specialized agent
→ Agent has tools, prompts, constraints specific to subtask
```

**Why this works:**
- Specialized agents perform better on narrow tasks
- No need to anticipate all agent types upfront
- Agents are "components" not "roles"

**Example:**
```python
def generate_agent(subtask, context):
    """
    Create agent specialized for specific subtask
    """
    return Agent(
        name=f"{subtask.type}Agent",
        tools=select_tools(subtask.requirements),
        prompt_template=craft_prompt(subtask),
        constraints=extract_constraints(context),
        examples=find_similar_from_library(subtask)
    )

# Example outputs:
generate_agent("Flatten nested JSON", context)
→ JSONFlattenerAgent
  Tools: [jq, pandas, json_normalize]
  Prompt: "Flatten nested JSON preserving all fields..."

generate_agent("Validate data schema", context)
→ SchemaValidatorAgent
  Tools: [jsonschema, pandas_dtype_check]
  Prompt: "Validate data against schema, report violations..."
```

### 3. Skill Accumulation Library

**Problem:** Regenerating solutions is wasteful.

**TDAG solution:** Save successful agent solutions to library.

```python
class SkillLibrary:
    """
    Store and retrieve agent solutions
    """
    def save_skill(self, task_signature, agent_config, solution):
        """
        task_signature: hash of (task type, inputs, constraints)
        agent_config: agent that solved it
        solution: code/approach that worked
        """
        self.library[task_signature] = {
            "agent": agent_config,
            "solution": solution,
            "success_count": 1,
            "last_used": now()
        }

    def find_similar(self, new_task):
        """
        Semantic search for similar solved tasks
        """
        candidates = vector_search(
            query=new_task.embedding,
            corpus=self.library,
            top_k=3
        )

        return [self.library[c] for c in candidates]

    def reuse_or_generate(self, task):
        """
        Try to reuse existing solution, generate if no match
        """
        similar = self.find_similar(task)

        if similar and similarity_score(task, similar[0]) > 0.85:
            # Reuse with minor adaptation
            return adapt_solution(similar[0].solution, task)
        else:
            # Generate new agent
            new_agent = generate_agent(task)
            solution = new_agent.solve(task)
            self.save_skill(task.signature, new_agent, solution)
            return solution
```

---

## Performance Results

### Benchmark Performance

| Benchmark | Static Plan-Execute | TDAG | Improvement |
|-----------|---------------------|------|-------------|
| **WebShop** (e-commerce tasks) | 58% | 73% | **+15%** |
| **ALFWorld** (household tasks) | 62% | 79% | **+17%** |
| **Mind2Web** (web navigation) | 41% | 54% | **+13%** |
| **Average** | 53.7% | 68.7% | **+15%** |

### Why Dynamic Beats Static

| Failure Mode | Static Decomposition | TDAG |
|--------------|---------------------|------|
| **Unexpected data format** | Fails (can't adapt) | Re-decomposes with format-specific tasks |
| **Resource constraints** | Fails or inefficient | Decomposes into smaller chunks |
| **Missing dependencies** | Blocks downstream tasks | Inserts new dependency resolution task |
| **Error propagation** | Compounds across subtasks | Isolates and re-decomposes failing subtask |

---

## Practical Implications

### For Knowledge & Vibes

TDAG validates core ADaPT principles:

| TDAG Insight | K&V Application |
|--------------|-----------------|
| **Dynamic decomposition** | Don't decompose all beads upfront; decompose on failure |
| **Agent generation** | Spawn sub-beads with specific context, not generic beads |
| **Skill accumulation** | CASS stores successful bead solutions for reuse |
| **Execution-informed** | Use test failures to guide sub-bead creation |

### Bead Decomposition Strategy

```markdown
## TDAG-Inspired Bead Workflow

1. **Initial Bead Creation** (Coarse)
   - Create beads at feature/component level
   - Don't decompose further yet

2. **Attempt Execution**
   - Agent implements bead
   - Runs tests
   - Observes: passes, failures, unexpected issues

3. **Outcome-Based Decision**

   IF all tests pass:
     → Close bead, move on

   IF tests fail after 1-2 iterations:
     → Continue iterating (simple bugs)

   IF tests fail after 3 iterations:
     → STOP: Analyze failure
     → Dynamic decomposition

4. **Dynamic Decomposition** (Execution-Informed)

   Analyze failure:
   - What specific aspect failed?
   - What constraints were discovered?
   - What dependencies are missing?

   Create specialized sub-beads:
   - Each targets ONE failure aspect
   - Include execution context (what was tried, what failed)
   - Narrow scope (easier for agent)

5. **Skill Reuse**
   - Check CASS for similar failed beads
   - Reuse successful sub-bead patterns
   - Don't regenerate solutions
```

### Example: TDAG-Style Decomposition

```markdown
## Initial Bead

BEAD-100: Implement data import pipeline
  REQ: Import customer data from various sources
  AC: All source formats supported, data validated

## First Execution

Attempt 1: ✗ Fails (unexpected JSON nesting)
Attempt 2: ✗ Fails (JSON too large for memory)
Attempt 3: ✗ Fails (validation too slow)

→ TRIGGER: Dynamic decomposition (3 failures)

## Execution-Informed Analysis

Discovered constraints:
- Data is nested JSON (not flat CSV as assumed)
- Files are 2-5GB (exceeds memory)
- Validation schema complex (performance issue)

Failure aspects:
1. JSON structure handling
2. Memory management
3. Validation performance

## Dynamic Sub-Beads (TDAG-Generated)

BEAD-100.1: JSON structure flattening
  Context: Data is nested 3-4 levels deep
  Constraint: Must preserve all fields
  AC: Flat structure, no data loss
  Tools: jq, pandas.json_normalize
  Example: See BEAD-085 (similar JSON flattening)

BEAD-100.2: Streaming large file processor
  Context: Files 2-5GB, can't load into memory
  Constraint: Must process in chunks
  AC: Full file processed, constant memory
  Tools: pandas.read_json(chunksize=...), itertools
  Example: See BEAD-092 (streaming CSV processor)

BEAD-100.3: Optimized schema validation
  Context: Validation taking 10+ minutes per file
  Constraint: Must complete in < 60 seconds
  AC: Same validation coverage, 10x faster
  Tools: jsonschema with caching, vectorized ops
  Example: See BEAD-078 (fast validation pattern)

→ Each sub-bead is executable independently
→ Each has narrower scope (higher success rate)
→ Examples retrieved from CASS (skill reuse)
```

---

## Implementation Checklist

### For Dynamic Decomposition
- [ ] Track iteration count per bead
- [ ] At 3 failures: Stop and analyze
- [ ] Extract execution context (what was tried, what failed, constraints discovered)
- [ ] Use context to decompose (not just requirement)
- [ ] Create sub-beads with specific failure aspects

### For Agent Generation
- [ ] Don't assign generic "coder" agent to sub-beads
- [ ] Generate context-specific sub-bead description
- [ ] Include tools/libraries relevant to specific subtask
- [ ] Provide examples from similar successful beads (CASS search)

### For Skill Library (CASS Integration)
- [ ] Store successful bead solutions with metadata
- [ ] Semantic search for similar failed beads
- [ ] Reuse patterns from similar successes
- [ ] Track reuse success rate (does it help?)

---

## Key Takeaways

1. **Dynamic beats static** — Decompose based on execution feedback, not upfront guesses (+15-22% success)
2. **Agents as components** — Generate specialized sub-agents for subtasks, not predefined roles
3. **Execution informs decomposition** — Use what you learned from failures to guide sub-task creation
4. **Skill accumulation matters** — Reuse successful solutions instead of regenerating
5. **Adaptive task graphs** — Task decomposition is not static; revise based on outcomes

---

## Limitations

### Research Scope
- **Interactive tasks** — Benchmarks are web navigation, household tasks (not repo-scale SWE)
- **Single-agent baseline** — Compared to static decomposition, not other dynamic approaches
- **Success metrics** — Task completion, not code quality or maintainability

### Practical Constraints
- **Decomposition cost** — Analyzing failures and re-decomposing adds overhead
- **Skill library quality** — Reuse only helps if library has relevant examples
- **Infinite recursion risk** — Need stopping condition (max depth, token limit)

### Open Questions
- **Optimal stopping** — When to stop decomposing and accept failure?
- **Skill generalization** — How similar must tasks be for reuse to work?
- **Multi-agent coordination** — How to coordinate many specialized sub-agents?

---

## See Also

- `038-adapt.md` — As-needed decomposition and planning (similar adaptive principle)
- `060-debugging-decay-index.md` — Why iteration limits (3-attempt rule) matter
- `040-mapcoder.md` — Multi-agent pipeline for code generation
- `059-multi-agent-orchestrator-2025.md` — Multi-agent coordination patterns
