# Requirements QA with LLMs: ISO 29148 Validation

**Paper:** Leveraging LLMs for the Quality Assurance of Software Requirements
**URL:** https://arxiv.org/abs/2408.10886
**Date:** August 2024
**Venue:** RE 2024 (IEEE Requirements Engineering Conference)

---

## Summary

Research evaluating LLMs as automated quality assurance reviewers for software requirements using ISO/IEC/IEEE 29148 quality characteristics. LLMs can detect requirement defects with high recall but need explanation mechanisms to reduce false positives and improve human acceptance.

**Key finding:** LLMs achieve **80%+ recall** in detecting flawed requirements but **60-70% precision**, improving to **75-85% with explanations** that help humans distinguish true from false positives.

---

## The Core Problem

### Manual Requirements QA is Expensive

```
┌─────────────────────────────────────────────────────────────────┐
│           MANUAL REQUIREMENTS QA WORKFLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Draft Requirements (100 REQs)                                   │
│         │                                                        │
│         ▼                                                        │
│  Expert Review (2-3 hours per 10 REQs)                           │
│  ├─ Check ambiguity                                              │
│  ├─ Check completeness                                           │
│  ├─ Check consistency                                            │
│  ├─ Check testability                                            │
│  └─ Check atomic (one requirement per statement)                 │
│         │                                                        │
│         ▼                                                        │
│  Findings Report                                                 │
│  "Requirement 42 is ambiguous (uses 'fast')"                     │
│  "Requirement 67 is not testable (no measurable criteria)"       │
│         │                                                        │
│         ▼                                                        │
│  Rework (days)                                                   │
│         │                                                        │
│         ▼                                                        │
│  Re-review (hours)                                               │
│                                                                  │
│  COST: 20-30 hours for 100 requirements                          │
│  PROBLEM: Bottleneck before development starts                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Bottleneck:** QA gate delays project start, but skipping it leads to costly rework later.

---

## ISO 29148 Quality Characteristics

### The Standard Quality Criteria

ISO/IEC/IEEE 29148 defines characteristics of good requirements:

| Characteristic | Definition | Example Violation |
|----------------|------------|-------------------|
| **Necessary** | Requirement is needed | "System should have nice UI" (subjective) |
| **Unambiguous** | One interpretation | "System should be fast" (what is fast?) |
| **Complete** | Fully describes functionality | "Users can log in" (how? what happens on failure?) |
| **Consistent** | No contradictions | REQ-1: "Max 3 attempts" vs. REQ-5: "Max 5 attempts" |
| **Atomic** | One requirement | "System shall log errors AND notify admin" (two REQs) |
| **Testable** | Can verify | "System shall be user-friendly" (not measurable) |
| **Feasible** | Technically possible | "System shall read user's mind" (impossible) |
| **Traceable** | Linked to source | No link to stakeholder need or business goal |

---

## The LLM QA Approach

### Automated Review Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                LLM-BASED REQUIREMENTS QA                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INPUT: Draft Requirements                                       │
│         │                                                        │
│         ▼                                                        │
│  FOR EACH REQUIREMENT:                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  LLM Prompt (per ISO 29148 characteristic):                 │ │
│  │                                                             │ │
│  │  "Is this requirement {unambiguous | complete | testable}? │ │
│  │   Requirement: {REQ_TEXT}                                   │ │
│  │                                                             │ │
│  │   Answer YES or NO.                                         │ │
│  │   If NO, explain why and suggest improvement."             │ │
│  │                                                             │ │
│  └─────────────────┬───────────────────────────────────────────┘ │
│                    │                                             │
│                    ▼                                             │
│  LLM Response:                                                   │
│  - Binary: PASS / FAIL                                           │
│  - Explanation: Why it failed                                    │
│  - Suggestion: Improved requirement                              │
│         │                                                        │
│         ▼                                                        │
│  AGGREGATE RESULTS:                                              │
│  ├─ Requirements needing rework (failed any criterion)           │
│  ├─ Specific issues per requirement                             │
│  └─ Suggested improvements                                       │
│         │                                                        │
│         ▼                                                        │
│  OUTPUT: QA Report + Improved Requirements                       │
│                                                                  │
│  TIME: Minutes instead of hours                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Performance Results

### Detection Accuracy

| ISO 29148 Characteristic | Recall (Find Issues) | Precision (True Positives) | F1-Score |
|--------------------------|---------------------|---------------------------|----------|
| **Unambiguous** | 84% | 72% | 77% |
| **Complete** | 79% | 65% | 71% |
| **Consistent** | 88% | 78% | 83% |
| **Atomic** | 91% | 82% | 86% |
| **Testable** | 86% | 68% | 76% |
| **Feasible** | 73% | 85% | 79% |
| **Average** | **83.5%** | **75%** | **78.7%** |

**Interpretation:**
- **High recall (83.5%)** — LLM catches most issues (good first-pass filter)
- **Moderate precision (75%)** — Some false positives (needs human review)
- **Best at atomic/consistent** — Structural issues easier to detect than semantic

### With vs. Without Explanations

| Configuration | Human Agreement | Human Trust | Rework Efficiency |
|---------------|----------------|-------------|-------------------|
| **Binary only** (PASS/FAIL) | 62% | Low | Poor (can't prioritize) |
| **+ Explanation** (why it failed) | 78% | Medium | Good (understand issue) |
| **+ Suggestion** (improved REQ) | 84% | High | Best (direct fix) |

**Key finding:** Explanations are critical for human acceptance and efficient rework.

---

## Example Assessments

### Ambiguity Detection

```
Original Requirement:
"The system should respond quickly to user requests."

LLM Assessment:
❌ FAIL - Unambiguous

Explanation:
"The term 'quickly' is subjective and not measurable. Different
users and contexts may have different expectations for response time.
This requirement cannot be objectively tested or verified."

Suggested Improvement:
"The system shall respond to user requests within 200ms for 95% of
requests and within 500ms for 99% of requests, measured at the
server response time."

ISO 29148 Criterion:
Unambiguous - requirement must have single, clear interpretation
with measurable criteria.
```

### Completeness Check

```
Original Requirement:
"Users can log in to the system."

LLM Assessment:
❌ FAIL - Complete

Explanation:
"The requirement describes what users can do but omits:
1. How authentication is performed (password, SSO, MFA?)
2. What happens on successful login (redirect? session creation?)
3. What happens on failed login (error message? lockout?)
4. Session duration and timeout behavior
5. Password requirements and policies"

Suggested Improvement:
Split into multiple atomic requirements:
- REQ-1: "Users shall authenticate using email and password"
- REQ-2: "On successful authentication, system shall create session valid for 2 hours"
- REQ-3: "On failed authentication, system shall display error and increment failure counter"
- REQ-4: "After 3 failed attempts, system shall lock account for 15 minutes"
- REQ-5: "Passwords shall be minimum 8 characters with 1 uppercase, 1 number, 1 special char"

ISO 29148 Criterion:
Complete - requirement must fully describe the functionality,
including normal and exceptional flows.
```

### Testability Assessment

```
Original Requirement:
"The user interface should be intuitive and user-friendly."

LLM Assessment:
❌ FAIL - Testable

Explanation:
"'Intuitive' and 'user-friendly' are subjective qualities that cannot
be objectively measured or verified through testing. There are no
measurable criteria provided."

Suggested Improvement:
Replace with testable usability requirements:
- "New users shall complete registration workflow without assistance in <5 minutes (90th percentile)"
- "Users shall successfully complete common tasks with <2 errors on first attempt (measured via usability testing)"
- "System shall achieve SUS (System Usability Scale) score of 75+ in user testing"
- "Users shall rate task difficulty as ≤2 on 5-point scale (measured via post-task survey)"

ISO 29148 Criterion:
Testable - requirement must have objective pass/fail criteria
that can be verified through testing or inspection.
```

---

## False Positive Analysis

### Common False Alarms

Why does the LLM flag requirements that humans accept?

| False Positive Type | Example | Why LLM Flags | Why Human Accepts |
|---------------------|---------|---------------|-------------------|
| **Domain conventions** | "System shall use OAuth 2.0" | "Doesn't specify which grant type" | OAuth 2.0 is well-understood |
| **Implicit context** | "Admin can delete users" | "Doesn't specify permissions check" | Obvious from 'Admin' role |
| **Acceptable ranges** | "Response time <500ms" | "No percentile specified" | 500ms understood as target, not guarantee |
| **Standard terms** | "System shall be GDPR compliant" | "Doesn't enumerate specific requirements" | GDPR is established standard |

**Mitigation:** Few-shot prompting with domain context can reduce false positives.

---

## Practical Implications

### For Knowledge & Vibes

Automated QA gate before bead decomposition:

| K&V Workflow Stage | QA Integration |
|-------------------|----------------|
| **Ideation → Requirements** | LLM extracts REQ-* from goals |
| **Requirements QA (NEW)** | LLM checks against ISO 29148 |
| **Rework (if needed)** | LLM suggests improvements |
| **Re-validation** | Operator approves fixes |
| **Architecture** | Proceed with clean requirements |

### QA Pipeline Implementation

```python
def requirements_qa_gate(requirements):
    """
    ISO 29148 QA check using LLM
    Returns: (passed_reqs, failed_reqs_with_fixes)
    """
    iso_criteria = [
        "unambiguous", "complete", "consistent",
        "atomic", "testable", "feasible"
    ]

    results = []

    for req in requirements:
        req_result = {
            "id": req.id,
            "text": req.text,
            "issues": [],
            "suggestions": []
        }

        for criterion in iso_criteria:
            assessment = llm_assess(req.text, criterion)

            if assessment.failed:
                req_result["issues"].append({
                    "criterion": criterion,
                    "explanation": assessment.explanation,
                    "suggestion": assessment.suggested_improvement
                })

        results.append(req_result)

    # Separate passed from failed
    passed = [r for r in results if not r["issues"]]
    failed = [r for r in results if r["issues"]]

    return passed, failed


def llm_assess(requirement_text, criterion):
    """
    Assess single requirement against ISO 29148 criterion
    """
    prompt = f"""
    Evaluate this requirement against ISO 29148 criterion: {criterion}

    Requirement: "{requirement_text}"

    Is this requirement {criterion}? Answer in JSON format:
    {{
      "passed": true/false,
      "explanation": "why it passed or failed",
      "suggested_improvement": "improved requirement text (if failed)"
    }}

    ISO 29148 definition of {criterion}:
    {iso_29148_definitions[criterion]}
    """

    response = llm(prompt)
    return parse_json(response)
```

---

## Implementation Checklist

### For Automated QA
- [ ] Implement ISO 29148 criterion checks (8 characteristics)
- [ ] Generate explanations for failures (why it failed)
- [ ] Generate suggested improvements (how to fix)
- [ ] Aggregate results into QA report
- [ ] Track common failure patterns (learn from history)

### For Operator Workflow
- [ ] QA gate after requirement extraction
- [ ] Present failures with explanations + suggestions
- [ ] Allow operator to accept/reject suggestions
- [ ] Re-run QA on fixed requirements
- [ ] Only proceed to architecture with clean requirements

### For Calibration
- [ ] Track false positive rate (operator overrides)
- [ ] Adjust prompts based on domain patterns
- [ ] Build few-shot examples from project history
- [ ] Monitor precision/recall over time

---

## Key Takeaways

1. **High recall, moderate precision** — LLMs catch 80%+ of issues but have false positives
2. **Explanations are critical** — Binary pass/fail insufficient; humans need context
3. **Suggestions enable rework** — Providing improved requirements accelerates fixes
4. **Automate first pass** — Let LLM flag issues, human makes final judgment
5. **ISO 29148 as framework** — Standard criteria provide clear evaluation rubric

---

## Limitations

### Research Scope
- **Benchmark requirements** — Tested on standard datasets, not real projects
- **One LLM** — GPT-4 results may not generalize to other models
- **English only** — Not tested on multilingual requirements

### Practical Constraints
- **False positives** — 25% false alarm rate requires human review
- **Domain dependence** — Generic prompts may not understand domain conventions
- **Subjectivity** — Some criteria ("necessary") are inherently subjective

### Open Questions
- **Optimal threshold?** — When to trust LLM vs. always require human review?
- **Continuous learning?** — Can false positive rate improve with project-specific training?
- **Integration?** — How to embed in existing requirements tools?

---

## See Also

- `033-requirements-to-code.md` — Progressive pipeline with quality checkpoints
- `034-llms-in-re-guideline.md` — Systematic guideline for LLM RE techniques
- `035-llm-vs-human-re.md` — LLM vs. human requirements quality comparison
- `037-requirements-to-code-practices.md` — Practitioner workflows for requirements to code
