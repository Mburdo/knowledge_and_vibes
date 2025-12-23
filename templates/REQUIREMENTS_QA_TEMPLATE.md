---
title: Requirements QA Template
description: Remove ambiguity and make requirements compile into tests and implementation tasks.
category: templates
stage: 2
---

# Requirements QA (Template)

Use this after drafting `REQ-*` / `AC-*`. The goal is to remove ambiguity and make requirements "compile" into tests and implementation tasks.

**Why this matters:** Requirements quality is the highest-leverage intervention. Ambiguity in requirements becomes divergent implementations. Fix it early when it's cheap.

**Evidence base:**
- `research/036-requirements-qa-iso-29148.md`: LLMs can QA requirements effectively
- `research/033-requirements-to-code.md`: Requirements quality is decisive
- `research/037-requirements-to-code-practices.md`: Practitioners must decompose into concrete tasks

---

## Template

```markdown
# Requirements QA

## Scope
- Project:
- Date:
- Reviewer(s):
- Rigor tier:

## QA Rules
- P0 requirements must be **unambiguous** and **verifiable**
- If a requirement can't be tested or falsified, it must be rewritten or split
- Any rewrite must be confirmed by the operator ("yes, that's what I mean")

---

## Phase 1: Quality Characteristics (per REQ-*)

For each `REQ-*`, score and fix:

| REQ | Atomic? | Unambiguous? | Verifiable? | Complete? | Consistent? | Feasible? | Notes / Fix |
|-----|---------|--------------|-------------|-----------|-------------|-----------|-------------|
| REQ-1 | ✅/⚠️/❌ | ✅/⚠️/❌ | ✅/⚠️/❌ | ✅/⚠️/❌ | ✅/⚠️/❌ | ✅/⚠️/❌ | ... |

### Quality Definitions

| Quality | Pass | Fail |
|---------|------|------|
| **Atomic** | Single testable outcome | "System should be fast AND secure" |
| **Unambiguous** | One interpretation possible | "Should handle large files" (how large?) |
| **Verifiable** | Can write a test for it | "Should be user-friendly" |
| **Complete** | All conditions specified | "Validate input" (which inputs? what validation?) |
| **Consistent** | No conflicts with other REQs | REQ-3 says X, REQ-7 says not-X |
| **Feasible** | Achievable with current constraints | Requires tech that doesn't exist |

---

## Phase 2: Common Failure Patterns

Check each P0 requirement against these failure patterns:

| Pattern | Example | Fix |
|---------|---------|-----|
| **Vague quantifiers** | "Handle many users" | Specify: "Handle 10,000 concurrent users" |
| **Implicit assumptions** | "Save user data" | Specify: where, format, retention, encryption |
| **Missing error cases** | "Upload files" | Add: size limits, type restrictions, failure handling |
| **Undefined terms** | "Real-time updates" | Define: <100ms latency, WebSocket, polling interval |
| **Hidden dependencies** | "Integrate with auth" | Specify: which auth system, what tokens, what flows |
| **Security gaps** | "Accept user input" | Add: validation rules, sanitization, rate limits |

---

## Phase 3: Security-Specific Checks (P0 Requirements)

For any P0 requirement involving user input, auth, or data:

| REQ | Input validated? | Auth specified? | Data protected? | Rate limits? | Notes |
|-----|------------------|-----------------|-----------------|--------------|-------|
| REQ-1 | ✅/❌/N/A | ✅/❌/N/A | ✅/❌/N/A | ✅/❌/N/A | ... |

**If any ❌ for P0:** Requirement is incomplete. Add security constraints before proceeding.

---

## Phase 4: AC Coverage Check

| REQ | AC Exists? | AC Is Measurable? | AC Has Test Shape? | Notes |
|-----|------------|-------------------|--------------------|-------|
| REQ-1 | ✅/❌ | ✅/⚠️/❌ | ✅/⚠️/❌ | ... |

### AC Quality Checks

| Quality | Good AC | Bad AC |
|---------|---------|--------|
| **Measurable** | "Response time < 200ms" | "Fast response" |
| **Test-shaped** | "Given X, when Y, then Z" | "Works correctly" |
| **Boundary-aware** | "Handles 0, 1, max items" | "Handles items" |
| **Error-aware** | "Returns 400 for invalid input" | "Validates input" |

---

## Rewrite Log

| REQ | Before | After | Why |
|-----|--------|-------|-----|
| REQ-3 | "Handle large files" | "Handle files up to 100MB; reject larger with error message" | Added measurable constraint |

---

## Missing Requirements (Discovered During QA)

| New REQ | Discovered From | Priority |
|---------|-----------------|----------|
| REQ-?: Rate limiting for API | REQ-3 security check | P0 |

---

## Operator Confirmations Needed

| Item | Question | Decision |
|------|----------|----------|
| REQ-3 rewrite | Is 100MB the right limit? | [pending] |
| REQ-7 ambiguity | Which auth provider: OAuth, JWT, or both? | [pending] |

---

## Output
- Updated `PLAN/01_requirements.md`
- Updated `PLAN/02_decisions_adrs.md` if any requirement implies a decision
- All P0 requirements pass Phase 1-4 checks
```

---

## Agent Prompt (Recommended)

Use this as a message to a planning agent:

```markdown
You are doing Requirements QA. You must:
1) Evaluate each REQ-* against: atomicity, unambiguity, verifiability, completeness, consistency, feasibility.
2) Propose rewrites only when necessary, and preserve user intent.
3) For every rewrite, ask for operator confirmation if meaning could change.
4) Ensure every P0 REQ has at least one measurable AC-*.

Input:
- North Star Card: <paste>
- Requirements list: <paste REQ/AC>

Output:
- QA table
- rewrite log
- operator questions
```

