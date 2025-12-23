# Enhanced Bead Structure Template

A properly enhanced bead is a complete specification that any agent can implement correctly.

> **Verification Focus:** This template maximizes success through TDD-first, security gates, and iteration limits. Workflow beats prompting.

## The TDD Flow (AlphaCodium Pattern)

```
1. SPEC: Read this bead description
2. TESTS: Tests are already written (in section below) ← START HERE
3. RUN: Run tests (expect failure - nothing implemented yet)
4. IMPLEMENT: Write minimal code to pass tests
5. RUN: Run tests (expect success)
6. REPAIR: If failing, fix (max 3 iterations)
7. SECURITY: Run `ubs --staged`
8. CLOSE: Only if tests pass AND ubs clean
```

If still failing after 3 iterations: STOP, spawn sub-bead for the failing part (ADaPT pattern), notify operator.

---

## Critical Rules (Non-Negotiable)

| Rule | Why | Evidence |
|------|-----|----------|
| **Tests FIRST** | 45.97% higher success rate | `research/054-tdd-ai-code-gen.md` |
| **`ubs --staged` before close** | ~40% of LLM code has vulnerabilities | `research/052-llm-security-vulnerabilities.md` |
| **Max 3 repair iterations** | Security degrades with more | `research/053-feedback-loop-security.md` |

---

## Template

```markdown
# [Bead Title]

## Complexity Assessment (P15)

**Files touched:** {count}
**Cross-module:** {yes/no}
**Ambiguity level:** {low/medium/high}

→ **Complexity:** {SIMPLE/MEDIUM/COMPLEX/ARCHITECTURAL}
→ **Verification required:** {see table below}

| Complexity | Verification |
|------------|-------------|
| SIMPLE | Tests + ubs |
| MEDIUM | Tests + ubs + code review |
| COMPLEX | Tests + ubs + human review |
| ARCHITECTURAL | Human-led |

## North Star Card (Verbatim)
Copy the North Star Card from the master plan.

Alignment note: [One sentence explaining how this bead advances the North Star]

## ⚠️ TESTS FIRST (Write Before Implementation)

**Why first:** TDD yields 45.97% pass@1 improvement. Tests in bead description, written BEFORE implementation.

### Tests That Prove Correctness
```python
# Write these FIRST - before any implementation
def test_success_scenario():
    """Happy path - must pass for bead to close"""
    # Full test code here - not placeholders

def test_edge_case():
    """Boundary condition"""
    # Full test code here

def test_error_handling():
    """Expected failure modes"""
    # Full test code here
```

### Test Coverage Requirements
- [ ] Happy path covered
- [ ] Edge cases covered
- [ ] Error conditions covered
- [ ] Security-relevant inputs validated

## Grounding Status
| Pattern | Source | Status |
|---------|--------|--------|
| [library.method] | [Exa query or doc URL] | ✅ Verified |

## Localization (Edit Locus)
If this bead changes existing code, record **where** and **why** before writing patches.

| Scope | Location | Why here? | Evidence |
|------|----------|-----------|----------|
| File | `path/to/file` | ... | grep/link |
| Symbol | `Class.method` / `function()` | ... | code refs |
| Lines | `file.py:123` | ... | failing stack trace |

## Context Sufficiency Check
- What *invariant* could this change break?
- What file/module is the “source of truth” for that invariant?
- What would we need to read/verify to be confident we’re not missing a dependency?

## Context & Motivation

### WHY This Exists
[2-3 sentences explaining the purpose]

### What Happens If Built Wrong
[Consequences of incorrect implementation]

### Architectural Decisions
| Decision | Choice | Rationale |
|----------|--------|-----------|
| [decision point] | [what we chose] | [why] |

## System Integration Map
```
┌─────────────┐    ┌─────────────┐
│  Component  │───▶│  Component  │
└─────────────┘    └─────────────┘
        │
        ▼
┌─────────────┐
│  Component  │
└─────────────┘
```

## Prerequisites
- [ ] [Dependency 1] — [why needed]
- [ ] [Dependency 2] — [why needed]

## Detailed Specification

### Inputs
| Field | Type | Validation | Description |
|-------|------|------------|-------------|

### Outputs
| Field | Type | Description |
|-------|------|-------------|

### Data Structures
```python
@dataclass
class ExampleModel:
    field: str  # Description of field
```

### Error Conditions
| Error | Cause | Handling |
|-------|-------|----------|

## Implementation Guidance

### Structured CoT Plan (For Complex Logic — +9.7% success rate)

If this bead involves complex control flow, document the logic structure:

```
1. SEQUENCE: {initialization steps}
   - Step A
   - Step B

2. LOOP: {iteration logic}
   - For each {item} in {collection}:

   2.1 BRANCH: IF {condition}
       - SEQUENCE: {then actions}

   2.2 ELSE:
       - SEQUENCE: {else actions}

3. SEQUENCE: {finalization}
   - Return {result}
```

**Evidence:** `research/070-structured-cot-code.md` — Using SEQUENCE/BRANCH/LOOP structures improves code generation by 9.7%, with biggest gains on complex control flow.

### File Structure
```
app/
├── module/
│   ├── __init__.py
│   └── implementation.py
```

### Anti-Patterns
```python
# BAD - Don't do this
def wrong_way():
    pass

# GOOD - Do this instead
def right_way():
    pass
```

## Security Checklist (P13 Gate — Language-Aware)

**Primary Language:** {language}
**Risk Level:** {see table below}

| Language | OWASP Failure Rate | Risk Level |
|----------|-------------------|------------|
| Java | 72% | CRITICAL |
| C# | 45% | HIGH |
| JavaScript | 43% | HIGH |
| Python | 38% | STANDARD |

### Universal Checks (All Languages)
- [ ] No hardcoded credentials/secrets
- [ ] User input validated before use
- [ ] No SQL string concatenation
- [ ] No command injection vectors
- [ ] Output encoding for user-facing data
- [ ] Authentication checks present where needed

### Java-Specific (CRITICAL — 72% failure rate)
- [ ] Manual SQL injection review complete
- [ ] Input validation audit complete
- [ ] No `Runtime.exec()` with user input

### JavaScript/TypeScript (HIGH — XSS missed 86% of time)
- [ ] No `innerHTML` or `dangerouslySetInnerHTML` with user data
- [ ] User input escaped before HTML output
- [ ] Content-Security-Policy headers present
- [ ] JSON responses use proper content-type

### Python (STANDARD)
- [ ] No `eval()` or `exec()` with user input
- [ ] subprocess calls use list form, not shell=True

## Verification Checklist (Before Closing)

### Tests
- [ ] All tests written FIRST (before implementation)
- [ ] All tests passing
- [ ] Edge cases covered
- [ ] Error conditions handled

### Security
- [ ] `ubs --staged` returns zero high/critical
- [ ] Security checklist above completed
- [ ] Medium findings documented if present

### Quality
- [ ] All data structures implemented
- [ ] Integration verified
- [ ] Code reviewed (P14 for P0 requirements)

### Iteration Tracking
- [ ] Completed within 3 repair iterations
- [ ] If >3 iterations needed: spawned spike, notified operator
```
