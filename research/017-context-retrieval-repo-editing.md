---
title: Context Retrieval for Repository Editing
tags: [context, retrieval, reasoning, sufficiency, localization, precision]
protocols: [P2, P5]
---

# Reasoning for Context Retrieval in Repository-Level Code Editing

**Paper:** On The Importance of Reasoning for Context Retrieval in Repository-Level Code Editing
**URL:** https://arxiv.org/abs/2406.04464
**Date:** June 2024
**Venue:** arXiv preprint

---

## Summary

Critical research isolating **context retrieval** as the primary bottleneck in repository-level editing. Demonstrates that LLMs are improving at finding relevant code ("precision") but still struggle to know when they have enough context ("sufficiency detection").

**Key insight:** You must retrieve **both precisely and completely**. Finding needles isn't enough if you don't know when the haystack is complete.

---

## The Context Retrieval Problem

### The Two Challenges

```
┌─────────────────────────────────────────────────────────────────┐
│            CONTEXT RETRIEVAL: TWO DIMENSIONS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PRECISION (finding relevant code):                             │
│  ├─ Challenge: Avoid irrelevant files/snippets                  │
│  ├─ Current status: LLMs improving rapidly                      │
│  └─ Success rate: ~70-80% for modern models                     │
│                                                                  │
│  SUFFICIENCY (knowing when you have enough):                    │
│  ├─ Challenge: Detect missing critical context                  │
│  ├─ Current status: LLMs still struggling                       │
│  └─ Success rate: ~40-50% for modern models                     │
│                                                                  │
│  Outcome:                                                        │
│  ├─ Good precision × poor sufficiency = incomplete solutions    │
│  ├─ Models find some relevant code but miss key invariants      │
│  └─ Edits work locally but break global constraints             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example: The Missing Invariant

```python
Task: "Add timeout parameter to connect() function"

Retrieved Context (High Precision):
├─ database/connection.py::connect()  ✓ (target function)
├─ database/pool.py (calls connect)   ✓ (call site)
└─ tests/test_connection.py           ✓ (tests)

Missing Context (Failed Sufficiency):
├─ database/invariants.py              ✗ (defines timeout constraints)
│  """
│  CRITICAL: All timeouts must be:
│  - Positive integers
│  - Between 1 and 3600 seconds
│  - Logged for monitoring
│  """
└─ database/monitoring.py              ✗ (timeout logging requirement)

Result:
├─ Implementation adds timeout parameter ✓
├─ Updates call sites correctly ✓
├─ BUT violates invariants (accepts any value) ✗
└─ AND misses logging requirement ✗

Root Cause: Model couldn't detect it was missing critical context
```

---

## The Long Code Arena Benchmark

### Novel Contributions

| Task Type | What It Tests | Example |
|-----------|---------------|---------|
| **Library-based generation** | Using project-specific APIs | "Use our custom ORM to query users" |
| **CI build repair** | Fixing build failures | "Resolve import errors after refactor" |
| **Project-level completion** | Multi-file consistency | "Complete API endpoint with middleware" |
| **Commit message generation** | Understanding changes | "Describe what this diff accomplishes" |
| **Bug localization** | Finding error sources | "Where is the authentication bug?" |
| **Module summarization** | Code comprehension | "Explain what this package does" |

### Why These Tasks Matter

```
Traditional Benchmarks:
├─ HumanEval: Self-contained functions
├─ MBPP: Single-file problems
└─ Issue: No project-wide context needed

Long Code Arena:
├─ Tasks require understanding project structure
├─ Must navigate imports, dependencies, conventions
└─ Tests context retrieval, not just generation
```

---

## Reasoning and Context Retrieval

### The Hypothesis

```
Question: Does "reasoning" (chain-of-thought, planning) improve context retrieval?

Experiment Design:
├─ Baseline: Direct retrieval (query → results)
├─ Reasoning: Multi-step with explicit hypotheses
│  ├─ Step 1: What do we need to know?
│  ├─ Step 2: Where might that information be?
│  ├─ Step 3: Retrieve and validate
│  └─ Step 4: Assess sufficiency
└─ Measure: Precision, recall, sufficiency detection
```

### Results Summary

```
┌──────────────────────────────────────────────────────────────┐
│         REASONING IMPACT ON RETRIEVAL                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Metric: Precision (% retrieved items that are relevant)    │
│  ├─ Direct retrieval:    68%                                │
│  └─ With reasoning:      79%  (+11% improvement)            │
│                                                              │
│  Metric: Recall (% relevant items that were retrieved)      │
│  ├─ Direct retrieval:    62%                                │
│  └─ With reasoning:      71%  (+9% improvement)             │
│                                                              │
│  Metric: Sufficiency (% tasks where all needed context found)│
│  ├─ Direct retrieval:    41%                                │
│  └─ With reasoning:      52%  (+11% improvement)            │
│                                                              │
│  Conclusion:                                                 │
│  ├─ Reasoning helps significantly                           │
│  ├─ But sufficiency is still weak (52% is not good enough)  │
│  └─ Need better sufficiency detection methods               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## The Sufficiency Problem

### Why Sufficiency is Hard

```
Detecting Missing Context Requires:

1. Domain Knowledge
   ├─ "What constraints exist in this codebase?"
   ├─ "What invariants must be preserved?"
   └─ Challenge: Not explicitly documented

2. Negative Reasoning
   ├─ "What am I NOT seeing?"
   ├─ "What questions remain unanswered?"
   └─ Challenge: Absence is invisible

3. Global Understanding
   ├─ "How do all pieces fit together?"
   ├─ "What dependencies am I missing?"
   └─ Challenge: Can't load entire repo

4. Experience/Intuition
   ├─ "This feels incomplete..."
   ├─ "I usually see X with Y..."
   └─ Challenge: LLMs lack developer intuition
```

### Sufficiency Detection Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│           SUFFICIENCY DETECTION APPROACHES                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Approach 1: Checklist-Based                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ For each task type, ask:                                   │ │
│  │ ☐ Do I have the target code?                               │ │
│  │ ☐ Do I have all callers?                                   │ │
│  │ ☐ Do I have type definitions?                              │ │
│  │ ☐ Do I have error handling context?                        │ │
│  │ ☐ Do I have tests?                                         │ │
│  │ ☐ Do I have documentation?                                 │ │
│  │                                                             │ │
│  │ If any ☐ unchecked → insufficient context                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Approach 2: Dependency-Based                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Build dependency graph:                                    │ │
│  │ ├─ What imports does target need?                          │ │
│  │ ├─ What functions does it call?                            │ │
│  │ ├─ What types does it use?                                 │ │
│  │ └─ For each dependency, recursively check sufficiency      │ │
│  │                                                             │ │
│  │ If dependency chain incomplete → insufficient              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Approach 3: Question-Based                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Generate questions about the task:                         │ │
│  │ ├─ "What invariants must be preserved?"                    │ │
│  │ ├─ "What error cases exist?"                               │ │
│  │ ├─ "What patterns does this code follow?"                  │ │
│  │ └─ "What constraints apply?"                               │ │
│  │                                                             │ │
│  │ For each question, search for answers                      │ │
│  │ If questions remain unanswered → insufficient              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Approach 4: Test-Based                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Generate discriminating tests:                             │ │
│  │ ├─ Create tests that should pass                           │ │
│  │ ├─ Create tests for edge cases                             │ │
│  │ ├─ Create tests for common mistakes                        │ │
│  │ └─ If can't generate tests → missing context               │ │
│  │                                                             │ │
│  │ Confidence in tests correlates with context sufficiency    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation Guide

### Reasoning-Based Retrieval

```python
class ReasoningRetriever:
    """
    Multi-step context retrieval with reasoning and sufficiency checks.
    """

    def __init__(self, repo_path):
        self.repo = Repository(repo_path)
        self.retrieved = set()
        self.reasoning_trace = []

    def retrieve_with_reasoning(self, task_description):
        """Retrieve context using explicit reasoning steps."""

        context = {
            'code': [],
            'dependencies': [],
            'constraints': [],
            'tests': []
        }

        # Step 1: Understand what's needed
        requirements = self._analyze_requirements(task_description)
        self.reasoning_trace.append({
            'step': 'analyze_requirements',
            'output': requirements
        })

        # Step 2: Initial retrieval
        initial_context = self._initial_retrieval(requirements)
        context['code'].extend(initial_context)
        self.reasoning_trace.append({
            'step': 'initial_retrieval',
            'retrieved': [c['file'] for c in initial_context]
        })

        # Step 3: Iterative expansion
        iteration = 0
        while not self._is_sufficient(context, requirements):
            iteration += 1
            if iteration > 5:  # Prevent infinite loops
                break

            # Identify gaps
            gaps = self._identify_gaps(context, requirements)
            self.reasoning_trace.append({
                'step': f'identify_gaps_{iteration}',
                'gaps': gaps
            })

            # Fill gaps
            additional = self._fill_gaps(gaps)
            for category, items in additional.items():
                context[category].extend(items)

            self.reasoning_trace.append({
                'step': f'fill_gaps_{iteration}',
                'added': {k: len(v) for k, v in additional.items()}
            })

        # Step 4: Final sufficiency check
        sufficiency = self._check_sufficiency(context, requirements)
        self.reasoning_trace.append({
            'step': 'final_sufficiency_check',
            'result': sufficiency
        })

        return context, sufficiency

    def _analyze_requirements(self, task_description):
        """Use LLM to analyze what context is needed."""

        prompt = f"""
        Task: {task_description}

        What information do we need to complete this task?

        Consider:
        1. Target code location
        2. Dependencies (imports, calls)
        3. Type definitions
        4. Constraints/invariants
        5. Error handling patterns
        6. Related tests
        7. Documentation

        Return structured list of requirements.
        """

        requirements = llm.call(prompt)
        return requirements

    def _identify_gaps(self, context, requirements):
        """Identify what's missing from current context."""

        prompt = f"""
        Task requirements:
        {requirements}

        Current context:
        - Code files: {[c['file'] for c in context['code']]}
        - Dependencies: {[d['name'] for d in context['dependencies']]}
        - Constraints: {[c['description'] for c in context['constraints']]}
        - Tests: {[t['file'] for t in context['tests']]}

        What critical information is still missing?

        For each gap, specify:
        1. What is missing
        2. Why it's needed
        3. Where it might be found

        Return list of gaps with search strategies.
        """

        gaps = llm.call(prompt)
        return gaps

    def _is_sufficient(self, context, requirements):
        """Check if we have sufficient context."""

        # Checklist approach
        checklist = self._generate_checklist(requirements)
        checklist_score = self._evaluate_checklist(checklist, context)

        # Dependency approach
        dependency_complete = self._check_dependencies(context)

        # Question approach
        questions = self._generate_questions(requirements)
        questions_answered = self._check_questions(questions, context)

        # Combine signals
        sufficiency_score = (
            checklist_score * 0.4 +
            dependency_complete * 0.3 +
            questions_answered * 0.3
        )

        return sufficiency_score > 0.8

    def _generate_checklist(self, requirements):
        """Generate checklist of required context."""

        prompt = f"""
        Requirements: {requirements}

        Generate a checklist of context that MUST be retrieved.

        Format:
        - [ ] Item description (how to verify)

        Be comprehensive. Include:
        - Target code
        - All dependencies
        - Type definitions
        - Constraints/invariants
        - Error handling
        - Tests
        - Documentation
        """

        checklist = llm.call(prompt)
        return checklist

    def _evaluate_checklist(self, checklist, context):
        """Score how much of checklist is satisfied."""

        prompt = f"""
        Checklist:
        {checklist}

        Retrieved context:
        {context}

        For each checklist item, determine if it's satisfied by the context.

        Return:
        - Total items: N
        - Satisfied items: M
        - Score: M/N
        - Missing items: [list]
        """

        evaluation = llm.call(prompt)
        return evaluation['score']

    def _check_dependencies(self, context):
        """Verify all dependencies are resolved."""

        unresolved = []

        for code_item in context['code']:
            # Extract imports
            imports = extract_imports(code_item['content'])

            for imp in imports:
                # Check if imported module is in context
                if not self._has_import(imp, context):
                    unresolved.append(imp)

            # Extract function calls
            calls = extract_calls(code_item['content'])

            for call in calls:
                # Check if called function is in context
                if not self._has_function(call, context):
                    unresolved.append(call)

        # Score: 1.0 if all resolved, 0.0 if many unresolved
        if len(unresolved) == 0:
            return 1.0
        else:
            # Partial score based on resolution rate
            total = len(imports) + len(calls)
            return 1.0 - (len(unresolved) / total) if total > 0 else 0.0

    def _generate_questions(self, requirements):
        """Generate questions that must be answered."""

        prompt = f"""
        Requirements: {requirements}

        Generate critical questions that must be answered to complete this task.

        Focus on:
        - Invariants: "What must always be true?"
        - Constraints: "What are the limits?"
        - Patterns: "How is similar code structured?"
        - Edge cases: "What can go wrong?"
        - Dependencies: "What does this depend on?"

        Return list of specific, answerable questions.
        """

        questions = llm.call(prompt)
        return questions

    def _check_questions(self, questions, context):
        """Determine if questions are answered by context."""

        answered = 0

        for question in questions:
            prompt = f"""
            Question: {question}

            Context:
            {context}

            Can this question be answered using the context?

            Return:
            - answered: true/false
            - answer: (if answered)
            - confidence: 0-1
            """

            result = llm.call(prompt)
            if result['answered'] and result['confidence'] > 0.7:
                answered += 1

        return answered / len(questions) if questions else 0.0
```

---

## Integration with K&V Workflow

### Sufficiency Checks in Beads

```yaml
# Investigation bead with sufficiency validation
- id: bd-701
  title: "Retrieve context for authentication fix"
  phase: investigation
  deliverable: "Complete context for fix"
  validation:
    sufficiency_checklist:
      - target_code: true
      - all_callers: true
      - type_definitions: true
      - error_handling: true
      - tests: true
      - constraints: true
    minimum_score: 0.8

# If sufficiency fails, create sub-bead
- id: bd-702
  title: "Fill context gaps"
  phase: investigation
  condition: "bd-701 sufficiency_score < 0.8"
  deliverable: "Missing context items"
```

### Calibration with Sufficiency Assessment

```python
# In calibration:

def validate_phase_completeness(completed_beads):
    """Check if we have sufficient context to proceed."""

    # Collect all retrieved context
    all_context = collect_context(completed_beads)

    # Generate requirements from beads
    requirements = generate_requirements(completed_beads)

    # Assess sufficiency
    sufficiency = assess_sufficiency(all_context, requirements)

    if sufficiency['score'] < 0.8:
        # Create beads to fill gaps
        gap_beads = create_gap_filling_beads(sufficiency['gaps'])

        send_message(
            subject="[INSUFFICIENT CONTEXT] Gaps identified",
            body=f"Sufficiency score: {sufficiency['score']}\n"
                 f"Gaps: {sufficiency['gaps']}\n"
                 f"Created {len(gap_beads)} beads to fill gaps"
        )

        return False  # Phase not complete

    return True  # Sufficient context
```

---

## Key Takeaways

1. **Context retrieval is the bottleneck** — Not generation, but finding what to generate from
2. **Precision is improving, sufficiency is not** — Models find relevant code but miss critical pieces
3. **Reasoning helps but isn't enough** — Multi-step retrieval improves but doesn't solve sufficiency
4. **Checklists are valuable** — Explicit requirements help detect gaps
5. **Dependency analysis matters** — Unresolved dependencies indicate insufficient context
6. **Questions reveal gaps** — Unanswerable questions point to missing context
7. **Tooling can help** — Structured retrieval beats unstructured search

---

## See Also

- `012-autocoderover.md` — Structure-aware navigation and retrieval
- `011-agentless.md` — Hierarchical localization approach
- `016-res-q.md` — Repository-scale evaluation
- `065-confucius-code-agent.md` — Progressive loading and navigation
- `057-anthropic-context-engineering.md` — Minimal sufficient context
