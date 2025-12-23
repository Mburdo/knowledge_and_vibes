# LLMs in Requirements Engineering: Systematic Guideline

**Paper:** Using Large Language Models for Natural Language Processing Tasks in Requirements Engineering: A Systematic Guideline
**URL:** https://arxiv.org/abs/2402.13823
**Date:** February 2024
**Venue:** arXiv preprint

---

## Summary

Systematic framework for selecting appropriate LLM techniques for requirements engineering (RE) tasks. Provides decision trees based on task type (understanding vs. generation), automation level, and available resources (training data, compute, domain knowledge).

**Key finding:** RE effectiveness depends on **matching the right LLM technique** (zero-shot, few-shot, fine-tuning, RAG) **to the specific RE task** (extraction, classification, generation, validation).

---

## The Core Problem

### RE as a Minefield of NLP Tasks

Requirements engineering involves many distinct natural language processing tasks:

```
┌─────────────────────────────────────────────────────────────────┐
│                  RE TASK LANDSCAPE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  UNDERSTANDING TASKS (Analysis)                                  │
│  ├─ Ambiguity detection                                          │
│  ├─ Completeness checking                                        │
│  ├─ Consistency verification                                     │
│  ├─ Requirement classification                                   │
│  ├─ Stakeholder identification                                   │
│  └─ Dependency extraction                                        │
│                                                                  │
│  GENERATION TASKS (Synthesis)                                    │
│  ├─ Requirement refinement                                       │
│  ├─ Test case generation                                         │
│  ├─ Documentation synthesis                                      │
│  ├─ User story generation                                        │
│  ├─ Acceptance criteria writing                                  │
│  └─ Requirement normalization                                    │
│                                                                  │
│  TRANSFORMATION TASKS (Translation)                              │
│  ├─ Natural language → formal spec                               │
│  ├─ User story → use case                                        │
│  ├─ Legacy doc → modern format                                   │
│  └─ Domain terminology mapping                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Problem:** No single LLM approach works well for all tasks.

---

## The Guideline: Decision Framework

### Primary Decision Tree

```
┌─────────────────────────────────────────────────────────────────┐
│             LLM TECHNIQUE SELECTION TREE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  START: What is the RE task?                                     │
│         │                                                        │
│         ├─► UNDERSTANDING (classify/extract/detect)             │
│         │   │                                                    │
│         │   ├─► Do you have labeled examples?                   │
│         │   │   │                                                │
│         │   │   ├─► Yes (100+) ──► FINE-TUNING                  │
│         │   │   │                   Best accuracy                │
│         │   │   │                                                │
│         │   │   ├─► Some (5-50) ──► FEW-SHOT                    │
│         │   │   │                   Good balance                 │
│         │   │   │                                                │
│         │   │   └─► No (<5) ──────► ZERO-SHOT + RAG             │
│         │   │                       Use domain docs              │
│         │   │                                                    │
│         │   └─► Is ground truth available?                      │
│         │       │                                                │
│         │       ├─► Yes ──────────► RAG (retrieve facts)        │
│         │       └─► No ───────────► Ensemble voting             │
│         │                                                        │
│         └─► GENERATION (write/refine/synthesize)                │
│             │                                                    │
│             ├─► Is factual accuracy critical?                   │
│             │   │                                                │
│             │   ├─► Yes ─────────► RAG (ground in docs)         │
│             │   │                                                │
│             │   └─► No ──────────► Direct generation            │
│             │                                                    │
│             └─► Human review required?                          │
│                 │                                                │
│                 ├─► Yes ────────► Generate + validate            │
│                 │                                                │
│                 └─► No ─────────► Generate + auto-check         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Technique Details

### Zero-Shot Prompting

**When to use:**
- No training data available
- Quick experimentation
- General-purpose tasks

**Strengths:**
- No data preparation needed
- Fast to deploy
- Works out-of-the-box

**Weaknesses:**
- Lower accuracy than fine-tuned
- Inconsistent outputs
- Domain-specific terms may confuse

**Example:**
```
Task: Detect ambiguous requirements

Prompt:
"Is the following requirement ambiguous? Answer YES or NO.
Requirement: 'The system should be fast and reliable.'"

Output: "YES - 'fast' and 'reliable' are vague terms without measurable criteria."
```

### Few-Shot Prompting

**When to use:**
- Have 5-50 labeled examples
- Domain-specific terminology
- Need better accuracy than zero-shot

**Strengths:**
- Significant accuracy boost
- Adapts to domain patterns
- No training infrastructure needed

**Weaknesses:**
- Requires curating good examples
- Example selection matters
- Context window limitations

**Example:**
```
Task: Classify requirement type

Examples:
"Users must be able to log in." → Functional
"System uptime must exceed 99.9%." → Non-functional
"Password must be 8+ characters." → Functional
"Response time under 200ms." → Non-functional

New requirement:
"The system shall encrypt all data at rest."
Classification: Non-functional (security requirement)
```

### Fine-Tuning

**When to use:**
- Have 100+ labeled examples
- High accuracy required
- Repetitive task at scale

**Strengths:**
- Highest accuracy
- Consistent outputs
- Cost-effective at scale

**Weaknesses:**
- Requires training infrastructure
- Upfront data preparation cost
- Model updates lag behind new patterns

**Example:**
```python
# Fine-tune for requirement completeness checking
training_data = [
    {
        "requirement": "Users can view their profile",
        "label": "INCOMPLETE",
        "missing": ["authentication", "authorization", "which fields"]
    },
    {
        "requirement": "Authenticated users with 'view_profile' permission can view their own profile showing name, email, and registration date",
        "label": "COMPLETE",
        "missing": []
    },
    # ... 100+ more examples
]

model = fine_tune(base_model="gpt-3.5", data=training_data)
```

### RAG (Retrieval-Augmented Generation)

**When to use:**
- Large knowledge base (codebase, docs, history)
- Factual accuracy critical
- Context exceeds model window

**Strengths:**
- Grounded in source material
- Handles massive knowledge bases
- Reduces hallucination

**Weaknesses:**
- Requires good retrieval system
- Slower (retrieve + generate)
- Quality depends on retrieval accuracy

**Example:**
```python
# RAG for requirement validation against existing system

def validate_requirement(new_req):
    # Retrieve relevant existing requirements
    similar_reqs = vector_search(
        query=new_req,
        corpus=existing_requirements,
        top_k=5
    )

    # Generate validation using retrieved context
    prompt = f"""
    New requirement: {new_req}

    Related existing requirements:
    {similar_reqs}

    Check for:
    1. Conflicts with existing requirements
    2. Duplicates
    3. Dependencies
    4. Consistency issues
    """

    return llm(prompt)
```

---

## Task-Specific Recommendations

### Ambiguity Detection

| Approach | Accuracy | Cost | Recommended |
|----------|----------|------|-------------|
| Zero-shot | 65-75% | Low | Starting point |
| Few-shot (ISO 29148 examples) | 80-85% | Low | **Best for most** |
| Fine-tuned | 85-90% | High | High-volume only |
| RAG (domain glossary) | 75-80% | Medium | Domain-specific |

**Recommended:** Few-shot with ISO 29148 quality criteria examples.

### Completeness Checking

| Approach | Accuracy | Cost | Recommended |
|----------|----------|------|-------------|
| Zero-shot | 50-60% | Low | Not recommended |
| Few-shot | 65-75% | Low | Acceptable |
| Fine-tuned | 80-85% | High | **Best** |
| RAG (template library) | 70-80% | Medium | Good alternative |

**Recommended:** Fine-tune on historical complete/incomplete pairs, or RAG with requirement templates.

### Requirement Generation

| Approach | Quality | Cost | Recommended |
|----------|---------|------|-------------|
| Zero-shot | Variable | Low | Not recommended |
| Few-shot (similar domains) | Good | Low | **Best for prototyping** |
| RAG (existing requirements) | High consistency | Medium | **Best for production** |
| Fine-tuned | High quality | High | Overkill |

**Recommended:** RAG to ensure consistency with existing requirements and domain patterns.

### Dependency Extraction

| Approach | Precision | Recall | Recommended |
|----------|-----------|--------|-------------|
| Zero-shot | 40-50% | 30-40% | Not recommended |
| Few-shot | 60-70% | 50-60% | Acceptable |
| Fine-tuned | 80-85% | 75-80% | **Best** |
| RAG (architecture docs) | 70-75% | 65-70% | Good |

**Recommended:** Fine-tune if have labeled dependency data, otherwise RAG with system architecture documentation.

---

## Practical Implications

### For Knowledge & Vibes

Map RE tasks to LLM techniques:

| K&V RE Task | Best Technique | Rationale |
|-------------|----------------|-----------|
| Extract REQ from goals | Few-shot + RAG | Use similar projects + domain docs |
| Validate REQ quality | Few-shot (ISO 29148) | Standard quality criteria |
| Generate AC from REQ | RAG (test patterns) | Ground in existing test suite |
| Detect ambiguities | Few-shot | Quality criteria are learnable |
| Check completeness | RAG (templates) | Compare to requirement templates |
| Identify dependencies | RAG (codebase) | Extract from actual code/arch |

### RE Pipeline Design

```markdown
## K&V RE Sub-Pipeline

1. **Requirement Extraction** (Few-shot)
   - Input: Natural language goals
   - Examples: 5-10 similar projects
   - Output: Structured REQ-* statements

2. **Quality Validation** (Few-shot)
   - Input: REQ-* statements
   - Examples: ISO 29148 quality criteria
   - Output: Issues + suggested fixes

3. **Completeness Check** (RAG)
   - Input: REQ-* statements
   - Retrieval: Requirement templates for domain
   - Output: Missing elements

4. **AC Generation** (RAG)
   - Input: REQ-* statements
   - Retrieval: Similar REQs with AC from history
   - Output: AC-* test specifications

5. **Dependency Analysis** (RAG)
   - Input: All REQ-*
   - Retrieval: Architecture docs + codebase
   - Output: Dependency graph
```

---

## Implementation Checklist

### For Requirements Quality Gate
- [ ] Few-shot prompt for ambiguity detection (use ISO 29148 examples)
- [ ] RAG system for completeness (retrieve requirement templates)
- [ ] Few-shot prompt for consistency checking
- [ ] Ensemble voting for critical validations

### For Requirements Generation
- [ ] RAG retrieval over existing requirements (find similar)
- [ ] RAG retrieval over domain documentation (ground in facts)
- [ ] Few-shot prompt with domain-specific examples
- [ ] Human review checklist (operator validates)

### For Test Generation
- [ ] RAG retrieval over existing test suites (find patterns)
- [ ] Few-shot prompt with AC → test examples
- [ ] Template library for common test scenarios

---

## Key Takeaways

1. **No universal solution** — Different RE tasks need different LLM techniques
2. **RAG for facts** — When accuracy matters, ground generation in source material
3. **Few-shot for patterns** — When you have 5-50 examples, few-shot is sweet spot
4. **Fine-tune for scale** — Only when you have 100+ examples and high volume
5. **Always validate** — LLM outputs need human or automated validation checkpoints

---

## Limitations

### Research Scope
- **Guideline, not benchmark** — Doesn't provide controlled comparisons
- **General RE tasks** — Specific domains may differ
- **Technique availability** — Based on 2024 LLM capabilities

### Practical Constraints
- **Data requirements vary** — Guidelines assume certain data availability
- **Domain expertise needed** — Selecting right examples requires RE knowledge
- **Continuous evolution** — LLM capabilities change, guidelines must update

---

## See Also

- `033-requirements-to-code.md` — Progressive prompting pipeline for RE
- `035-llm-vs-human-re.md` — LLM vs. human performance in RE tasks
- `036-requirements-qa-iso-29148.md` — ISO 29148 quality criteria for requirements
- `037-requirements-to-code-practices.md` — How practitioners use LLMs in RE
- `062-rag-repository-code.md` — RAG techniques for code repositories
