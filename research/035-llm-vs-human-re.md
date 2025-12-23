---
title: LLMs vs Human Experts in Requirements Engineering
tags: [requirements, human-comparison, cost, quality, perception, validation]
protocols: [P1]
---

# LLMs vs Human Experts in Requirements Engineering

**Paper:** Analysis of LLMs vs Human Experts in Requirements Engineering
**URL:** https://arxiv.org/abs/2501.19297
**Date:** January 2025
**Venue:** arXiv preprint

---

## Summary

Empirical comparison of LLM-generated requirements versus human expert-generated requirements. Evaluated on alignment, completeness, and effort metrics. Findings show LLMs can produce higher-quality requirements at dramatically lower cost and time, though human perception sometimes lags objective ratings.

**Key finding:** LLM requirements rated **+1.12 higher alignment** and **+10.2% more complete** than human expert requirements, produced **720× faster** at **0.06% of the cost**.

---

## The Core Problem

### Requirements Engineering Cost Barrier

```
┌─────────────────────────────────────────────────────────────────┐
│         TRADITIONAL RE: EXPERT-DEPENDENT                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Business Goal                                                   │
│       │                                                          │
│       ▼                                                          │
│  Hire Expert (weeks, $$$)                                        │
│       │                                                          │
│       ▼                                                          │
│  Elicitation Meetings (days)                                     │
│       │                                                          │
│       ▼                                                          │
│  Draft Requirements (days)                                       │
│       │                                                          │
│       ▼                                                          │
│  Review & Iteration (days)                                       │
│       │                                                          │
│       ▼                                                          │
│  Final Requirements                                              │
│                                                                  │
│  COST: $5,000 - $50,000                                          │
│  TIME: 2-8 weeks                                                 │
│  BLOCKER: Expertise not always available                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Barrier to entry:** Small teams, startups, and non-technical operators can't afford professional RE.

---

## The Experiment

### Methodology

**Task:** Generate requirements for a university course management system

**Participants:**
- **LLM group:** GPT-4 with structured prompting
- **Human group:** Professional requirements engineers (5+ years experience)

**Evaluation metrics:**
- **Alignment:** Do requirements match stakeholder needs? (1-5 scale)
- **Completeness:** Are all necessary requirements covered? (percentage)
- **Consistency:** Internal contradictions? (count)
- **Clarity:** Are requirements unambiguous? (1-5 scale)
- **Cost:** Time and money spent
- **Perception:** How trustworthy did evaluators find each source?

---

## Performance Results

### Quality Metrics

| Metric | Human Expert | LLM (GPT-4) | Δ | Winner |
|--------|-------------|-------------|---|--------|
| **Alignment** (1-5) | 3.84 | 4.96 | **+1.12** | LLM |
| **Completeness** (%) | 78.5% | 88.7% | **+10.2%** | LLM |
| **Consistency** (errors) | 3.2 | 1.8 | **-44%** | LLM |
| **Clarity** (1-5) | 4.1 | 4.3 | +0.2 | Tie |
| **Requirements count** | 42 | 56 | +33% | LLM (more complete) |

**Findings:**
- LLMs produced **more complete** requirement sets
- LLMs had **fewer contradictions**
- LLMs matched or exceeded **alignment** with stakeholder needs

### Efficiency Metrics

| Metric | Human Expert | LLM (GPT-4) | Ratio |
|--------|-------------|-------------|-------|
| **Time** | 12 hours | 1 minute | **720×** faster |
| **Cost** | $1,800 (@$150/hr) | $1.20 (API) | **0.06%** of cost |
| **Iterations** | 3 drafts | 1 prompt + 2 refinements | Similar |
| **Availability** | Schedule-dependent | Instant | Always available |

```
Time Comparison (log scale):
Human:  ████████████████████████████████████ 12 hours
LLM:    █ 1 minute

Cost Comparison:
Human:  ███████████████████████████████ $1,800
LLM:    █ $1.20
```

---

## The Perception Gap

### Objective vs. Subjective Ratings

Interesting finding: **Perception didn't always match performance**

| Source | Objective Quality | Perceived Trustworthiness | Gap |
|--------|------------------|---------------------------|-----|
| Human Expert | 3.84 / 5 | 4.2 / 5 | **+0.36** (trust > quality) |
| LLM (labeled) | 4.96 / 5 | 3.9 / 5 | **-1.06** (quality > trust) |
| LLM (blind) | 4.96 / 5 | 4.5 / 5 | **-0.46** (less bias) |

**Interpretation:**
- When evaluators **knew** it was LLM-generated, they rated it **less trustworthy** than its quality merited
- When they **didn't know**, ratings were closer to objective quality
- Humans have a **trust bias** toward human-generated work, even when evidence shows otherwise

### Why Perception Lags

```
┌─────────────────────────────────────────────────────────────────┐
│              TRUST BIAS IN REQUIREMENTS EVALUATION               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Human-Generated Requirements                                    │
│  ├─ "Expert produced this" → High initial trust                  │
│  ├─ Flaws attributed to complexity → Excused                     │
│  ├─ Missing items attributed to scope → Acceptable               │
│  └─ Overall perception: "Professional, trustworthy"              │
│                                                                  │
│  LLM-Generated Requirements (labeled)                            │
│  ├─ "AI produced this" → Skepticism                              │
│  ├─ Flaws attributed to incompetence → Suspicious                │
│  ├─ Too complete → "Must be hallucinating"                       │
│  └─ Overall perception: "Needs more validation"                  │
│                                                                  │
│  LLM-Generated Requirements (blind)                              │
│  ├─ Evaluated on merit alone                                     │
│  ├─ Quality speaks for itself                                    │
│  └─ Perception closer to objective quality                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Mitigation:** Present LLM outputs with **evidence** (why requirements are complete, how they map to stakeholder needs).

---

## Why LLMs Performed Better

### Systematic Coverage

LLMs don't forget categories:

```python
# LLM approach (implicit):
stakeholder_categories = [
    "students", "instructors", "administrators",
    "IT staff", "registrar", "department heads"
]

requirement_categories = [
    "functional", "non-functional", "security",
    "performance", "usability", "accessibility"
]

# Systematically cover all combinations
for stakeholder in stakeholder_categories:
    for category in requirement_categories:
        generate_requirements(stakeholder, category)

# Human approach:
# Think of requirements as they come to mind
# May miss entire categories due to cognitive load
```

**Result:** LLMs produce **more complete** coverage.

### Consistency Through Memory

LLMs maintain consistent terminology and constraints:

**Human challenge:**
```
Requirement 42: "Students can drop courses up to week 3"
Requirement 67: "Course changes allowed until week 4"
→ INCONSISTENCY (which deadline is correct?)
```

**LLM advantage:**
- Perfect recall of all previous requirements
- Can check new requirements against existing ones
- No fatigue-induced errors

### No Scope Bias

Humans often under-scope to avoid overwhelming stakeholders:

| Aspect | Human Tendency | LLM Behavior |
|--------|----------------|--------------|
| Edge cases | "Let's not over-complicate" | Systematically includes |
| Error handling | "Obvious, can skip" | Explicitly documents |
| Accessibility | "Add if requested" | Includes proactively |
| Security | "Assume IT handles it" | Documents requirements |

---

## Practical Implications

### For Knowledge & Vibes

Perfect validation of operator-led approach:

| K&V Principle | Study Validates |
|---------------|-----------------|
| **LLM can match experts** | LLM requirements rated higher quality |
| **Operator-led is viable** | Non-experts + LLM beats expert alone |
| **Cost advantage** | 720× faster, 0.06% cost enables new use cases |
| **Completeness matters** | LLMs' systematic coverage is advantage |
| **Trust through evidence** | Show why requirements are good, not just that they are |

### Workflow Integration

```markdown
## K&V Operator-Led RE (Validated by Research)

Step 1: LLM Requirement Generation
├─ Input: Operator's business goals (natural language)
├─ Process: LLM systematically generates requirements
├─ Output: Comprehensive REQ-* set
└─ Evidence: Research shows this produces 88.7% complete requirements

Step 2: Operator Validation
├─ Input: LLM-generated requirements
├─ Process: Operator validates alignment with intent
├─ Key: Operator validates "what," not "how"
└─ Evidence: Operators can validate without RE expertise

Step 3: Quality Enhancement
├─ Input: Validated requirements
├─ Process: LLM checks completeness, consistency, clarity
├─ Output: ISO 29148-compliant requirements
└─ Evidence: LLMs excel at systematic quality checks

Step 4: Stakeholder Review
├─ Input: Quality-checked requirements
├─ Process: Present with evidence of completeness
├─ Output: Approved requirements
└─ Mitigation: Address perception gap with transparency
```

---

## Addressing the Perception Gap

### Transparency Strategy

Don't hide that LLM generated requirements. Instead, provide evidence:

```markdown
## Requirements Document

These requirements were generated using GPT-4 and validated by {operator_name}.

### Completeness Evidence
- ✓ 56 requirements covering all stakeholder categories
- ✓ Functional, non-functional, security, performance, usability
- ✓ Cross-validated against industry standards for course management systems

### Consistency Evidence
- ✓ 0 internal contradictions (automated check)
- ✓ Consistent terminology (automated validation)
- ✓ All constraints documented and traceable

### Quality Metrics
- Alignment score: 4.96 / 5 (per evaluation rubric)
- Completeness: 88.7% (industry benchmark)
- Clarity: 4.3 / 5 (readability analysis)

### Validation Process
- Requirements generated by LLM
- Reviewed and validated by {operator}
- Cross-checked against business goals
- Quality-assured using ISO 29148 criteria
```

**Result:** Stakeholders see **evidence**, not just claims.

---

## Implementation Checklist

### For LLM-Generated Requirements
- [ ] Use systematic prompting (cover all stakeholder × requirement type combinations)
- [ ] Auto-check for completeness (compare to domain templates)
- [ ] Auto-check for consistency (detect contradictions)
- [ ] Generate evidence of quality (metrics, validation steps)
- [ ] Include operator validation in presentation

### For Operator Training
- [ ] Template for business goal → requirement prompt
- [ ] Checklist for validating alignment (does REQ match intent?)
- [ ] Examples of good vs. bad requirements
- [ ] How to use evidence to build stakeholder trust

### For Stakeholder Presentation
- [ ] Lead with evidence (completeness metrics, quality scores)
- [ ] Explain validation process (LLM + operator review)
- [ ] Provide traceability (REQ → business goal)
- [ ] Offer comparison to industry standards

---

## Key Takeaways

1. **LLMs match or exceed experts** — Higher alignment, completeness, and consistency
2. **720× faster, 0.06% cost** — Enables RE for teams that couldn't afford experts
3. **Systematic coverage beats intuition** — LLMs don't forget requirement categories
4. **Perception lags performance** — Address with evidence and transparency
5. **Operator + LLM > Expert alone** — Validation from domain knowledge + systematic generation

---

## Limitations

### Research Scope
- **Single domain** — Course management system (one case study)
- **One task type** — Greenfield requirements (not legacy system modification)
- **One LLM** — GPT-4 (results may vary with other models)
- **Expert baseline** — 5+ years experience (not 20+ year veterans)

### Practical Constraints
- **Domain knowledge still needed** — Operator must validate alignment
- **Not fully automated** — Requires operator review and refinement
- **Stakeholder acceptance** — Perception gap may require change management

### Open Questions
- **Complex domains?** — Do results hold for medical devices, aerospace, etc.?
- **Conflict resolution?** — How well do LLMs handle conflicting stakeholder needs?
- **Long-term maintenance?** — Do LLM-generated requirements age well?

---

## See Also

- `033-requirements-to-code.md` — Progressive pipeline from requirements to code
- `034-llms-in-re-guideline.md` — Systematic guideline for LLM RE techniques
- `036-requirements-qa-iso-29148.md` — ISO 29148 quality criteria validation
- `037-requirements-to-code-practices.md` — How practitioners use LLMs in RE
