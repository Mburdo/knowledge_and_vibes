---
title: Requirements Are All You Need - From Requirements to Code with LLMs
tags: [requirements, progressive-prompting, checkpoints, pipeline, design, stages]
protocols: [P1, P3]
---

# Requirements Are All You Need: From Requirements to Code with LLMs

**Paper:** Requirements are All You Need: From Requirements to Code with LLMs
**URL:** https://arxiv.org/abs/2406.10101
**Date:** June 2024
**Venue:** arXiv preprint (presented at RE 2024, RE@Next! track)

---

## Summary

Research arguing that **high-quality, structured requirements** are the highest-leverage input for LLM-based software development. Instead of "one-line requirement → code," the paper proposes progressive prompting: requirements → design → tests → code, creating intermediate artifacts that humans can inspect.

**Key finding:** Structured requirements through a progressive pipeline (REQ → Design → Tests → Code) produces higher-quality code than direct generation, with **human-inspectable checkpoints** at each stage.

---

## The Core Problem

### The Direct Generation Fallacy

```
┌────────────────────────────────────────────────────────┐
│         TYPICAL LLM CODE GENERATION                     │
├────────────────────────────────────────────────────────┤
│                                                          │
│  "Build a user login system"                            │
│              │                                           │
│              ▼                                           │
│  [ LLM generates 1000 lines of code ]                   │
│              │                                           │
│              ▼                                           │
│  Human reviews code directly                            │
│              │                                           │
│  Problems:                                               │
│  ✗ No visibility into design decisions                  │
│  ✗ No testable intermediate artifacts                   │
│  ✗ Can't validate requirements before implementation    │
│  ✗ All-or-nothing: code works or it doesn't             │
│  ✗ Hard to debug (which requirement was misunderstood?) │
│                                                          │
└────────────────────────────────────────────────────────┘
```

### The Nontechnical Operator Problem

For operators without deep technical expertise:
- **Can't validate code quality** directly
- **Can't debug** when code fails
- **Can't modify** implementation details
- **Must trust** LLM blindly

But they **can** validate:
- Business requirements
- Expected behavior (tests)
- Design decisions (architecture)

---

## The Solution: Progressive Prompting

### Four-Stage Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│              PROGRESSIVE PROMPTING PIPELINE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STAGE 1: REQUIREMENTS ANALYSIS                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │  Natural Language Requirements                             │  │
│  │              │                                             │  │
│  │              ▼                                             │  │
│  │  LLM: Extract Functional Needs                             │  │
│  │              │                                             │  │
│  │              ▼                                             │  │
│  │  Output: Structured Requirements List                      │  │
│  │  - Feature 1: User registration                            │  │
│  │  - Feature 2: Login/logout                                 │  │
│  │  - Feature 3: Password reset                               │  │
│  │  - Constraints: Security, validation                       │  │
│  │                                                            │  │
│  │  ✓ CHECKPOINT: Human validates completeness                │  │
│  │                                                            │  │
│  └────────────────────┬───────────────────────────────────────┘  │
│                       │                                          │
│  STAGE 2: DESIGN                                                 │
│  ┌────────────────────▼──────────────────────────────────────┐  │
│  │                                                            │  │
│  │  Structured Requirements                                   │  │
│  │              │                                             │  │
│  │              ▼                                             │  │
│  │  LLM: Generate OO Design Model                             │  │
│  │              │                                             │  │
│  │              ▼                                             │  │
│  │  Output: Design Artifacts                                  │  │
│  │  - Class diagram                                           │  │
│  │  - Component interactions                                  │  │
│  │  - Data models                                             │  │
│  │  - API contracts                                           │  │
│  │                                                            │  │
│  │  ✓ CHECKPOINT: Human validates architecture                │  │
│  │                                                            │  │
│  └────────────────────┬───────────────────────────────────────┘  │
│                       │                                          │
│  STAGE 3: TEST GENERATION                                        │
│  ┌────────────────────▼──────────────────────────────────────┐  │
│  │                                                            │  │
│  │  Design Model + Requirements                               │  │
│  │              │                                             │  │
│  │              ▼                                             │  │
│  │  LLM: Generate Unit Tests                                  │  │
│  │              │                                             │  │
│  │              ▼                                             │  │
│  │  Output: Test Suite                                        │  │
│  │  - Test cases per requirement                              │  │
│  │  - Expected behaviors                                      │  │
│  │  - Edge cases                                              │  │
│  │  - Integration tests                                       │  │
│  │                                                            │  │
│  │  ✓ CHECKPOINT: Human validates expected behavior           │  │
│  │                                                            │  │
│  └────────────────────┬───────────────────────────────────────┘  │
│                       │                                          │
│  STAGE 4: CODE GENERATION                                        │
│  ┌────────────────────▼──────────────────────────────────────┐  │
│  │                                                            │  │
│  │  Design + Tests + Requirements                             │  │
│  │              │                                             │  │
│  │              ▼                                             │  │
│  │  LLM: Generate Code                                        │  │
│  │              │                                             │  │
│  │              ▼                                             │  │
│  │  Output: Implementation                                    │  │
│  │  - Passes all tests                                        │  │
│  │  - Follows design                                          │  │
│  │  - Meets requirements                                      │  │
│  │                                                            │  │
│  │  ✓ VALIDATION: Run tests (automated)                       │  │
│  │                                                            │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Why Progressive Prompting Works

| Stage | Human-Understandable? | Validation Method | If Wrong |
|-------|----------------------|-------------------|----------|
| Requirements | ✓ Yes | Business logic review | Regenerate requirements |
| Design | ✓ Yes (with guidance) | Architecture review | Regenerate design |
| Tests | ✓ Yes (behavior-focused) | Expected outcome review | Regenerate tests |
| Code | ✗ No (for nontechnical) | **Automated test execution** | Regenerate code |

**Key insight:** Failures are caught **before code** is written, when fixes are cheap.

---

## Detailed Stage Breakdown

### Stage 1: Requirements Analysis

**Input:**
```markdown
Natural language requirement:
"Users should be able to create an account, log in, and reset their password if forgotten."
```

**LLM Prompt:**
```
Extract discrete functional requirements from this specification:
{requirement_text}

For each requirement, identify:
1. The functional need
2. Actors involved
3. Preconditions
4. Postconditions
5. Constraints
```

**Output:**
```markdown
REQ-1: User Registration
  Actor: New user
  Precondition: User has valid email
  Action: Submit registration form
  Postcondition: Account created, confirmation email sent
  Constraints: Email must be unique, password must meet security policy

REQ-2: User Login
  Actor: Registered user
  Precondition: User has active account
  Action: Submit credentials
  Postcondition: User authenticated, session created
  Constraints: Max 3 failed attempts, then lockout

REQ-3: Password Reset
  Actor: Registered user
  Precondition: User has forgotten password
  Action: Request reset via email
  Postcondition: Reset link sent, temporary token generated
  Constraints: Token expires in 1 hour, single use
```

**Human Checkpoint:**
- Are all requirements present?
- Are constraints correct?
- Any ambiguities?

### Stage 2: Design Generation

**Input:** Structured requirements from Stage 1

**LLM Prompt:**
```
Generate an object-oriented design for these requirements:
{structured_requirements}

Include:
1. Class diagram (entities, relationships)
2. Component architecture (layers, responsibilities)
3. Data models (fields, types, constraints)
4. API contracts (endpoints, inputs, outputs)
```

**Output:**
```markdown
## Classes

User
  - id: UUID
  - email: String (unique, validated)
  - password_hash: String (bcrypt)
  - created_at: DateTime
  - is_active: Boolean

Session
  - id: UUID
  - user_id: UUID (FK to User)
  - token: String
  - expires_at: DateTime

PasswordResetToken
  - id: UUID
  - user_id: UUID (FK to User)
  - token: String
  - used: Boolean
  - expires_at: DateTime

## Components

AuthService
  - register(email, password) → User
  - login(email, password) → Session
  - logout(session_id) → void
  - request_reset(email) → void
  - reset_password(token, new_password) → void

ValidationService
  - validate_email(email) → Boolean
  - validate_password(password) → Boolean
  - check_unique_email(email) → Boolean

## API Endpoints

POST /auth/register
  Input: {email, password}
  Output: {user_id, message}

POST /auth/login
  Input: {email, password}
  Output: {session_token, expires_at}

POST /auth/logout
  Input: {session_token}
  Output: {message}

POST /auth/reset-request
  Input: {email}
  Output: {message}

POST /auth/reset-password
  Input: {token, new_password}
  Output: {message}
```

**Human Checkpoint:**
- Does architecture make sense?
- Are components properly separated?
- Any missing entities or relationships?

### Stage 3: Test Generation

**Input:** Design + requirements

**LLM Prompt:**
```
Generate unit tests for each requirement using this design:
{design_model}

For each test:
1. Setup preconditions
2. Execute action
3. Assert postconditions
4. Include edge cases
```

**Output:**
```python
# REQ-1: User Registration

def test_successful_registration():
    """User can register with valid email and password"""
    result = AuthService.register("user@example.com", "SecurePass123!")
    assert result.success == True
    assert User.exists(email="user@example.com")
    # Verify confirmation email sent
    assert email_sent_to("user@example.com")

def test_duplicate_email_registration():
    """Registration fails if email already exists"""
    AuthService.register("user@example.com", "Pass1")
    result = AuthService.register("user@example.com", "Pass2")
    assert result.success == False
    assert result.error == "Email already registered"

def test_weak_password_registration():
    """Registration fails if password doesn't meet policy"""
    result = AuthService.register("user@example.com", "weak")
    assert result.success == False
    assert result.error == "Password does not meet security requirements"

# REQ-2: User Login

def test_successful_login():
    """User can log in with correct credentials"""
    user = create_test_user("user@example.com", "Pass123!")
    session = AuthService.login("user@example.com", "Pass123!")
    assert session is not None
    assert session.user_id == user.id

def test_failed_login_wrong_password():
    """Login fails with incorrect password"""
    create_test_user("user@example.com", "Pass123!")
    session = AuthService.login("user@example.com", "WrongPass")
    assert session is None

def test_account_lockout_after_3_failures():
    """Account locks after 3 failed login attempts"""
    create_test_user("user@example.com", "Pass123!")
    AuthService.login("user@example.com", "Wrong1")
    AuthService.login("user@example.com", "Wrong2")
    AuthService.login("user@example.com", "Wrong3")
    result = AuthService.login("user@example.com", "Pass123!")
    assert result.error == "Account locked due to too many failed attempts"

# REQ-3: Password Reset

def test_password_reset_flow():
    """User can reset password via email token"""
    user = create_test_user("user@example.com", "OldPass123!")
    AuthService.request_reset("user@example.com")
    token = get_latest_reset_token(user.id)
    result = AuthService.reset_password(token, "NewPass123!")
    assert result.success == True
    # Verify can login with new password
    session = AuthService.login("user@example.com", "NewPass123!")
    assert session is not None

def test_reset_token_expires():
    """Reset token cannot be used after expiration"""
    user = create_test_user("user@example.com", "Pass123!")
    AuthService.request_reset("user@example.com")
    token = get_latest_reset_token(user.id)
    expire_token(token)  # Simulate 1 hour passing
    result = AuthService.reset_password(token, "NewPass!")
    assert result.success == False
    assert result.error == "Reset token has expired"

def test_reset_token_single_use():
    """Reset token cannot be reused"""
    user = create_test_user("user@example.com", "Pass123!")
    AuthService.request_reset("user@example.com")
    token = get_latest_reset_token(user.id)
    AuthService.reset_password(token, "NewPass1!")
    result = AuthService.reset_password(token, "NewPass2!")
    assert result.success == False
    assert result.error == "Reset token already used"
```

**Human Checkpoint:**
- Do tests cover all requirements?
- Are edge cases included?
- Do expected behaviors match intent?

### Stage 4: Code Generation

**Input:** Design + Tests + Requirements

**LLM Prompt:**
```
Implement the following design to pass all tests:
{design_model}
{test_suite}

Generate production-ready code that:
1. Passes all tests
2. Follows the design architecture
3. Includes error handling
4. Has appropriate logging
```

**Output:** Full implementation (omitted for brevity)

**Automated Validation:**
```bash
pytest tests/
# All tests must pass before proceeding
```

---

## Key Benefits

### For Nontechnical Operators

| Benefit | Description | Impact |
|---------|-------------|--------|
| **Inspectable checkpoints** | Can validate requirements, design, tests before code | Catch errors early |
| **Language they understand** | Requirements and tests are behavior-focused | No code review needed |
| **Fail fast** | Errors caught at requirements/design stage | Avoid wasted implementation |
| **Iterative refinement** | Can modify requirements and regenerate downstream | Cheap changes |
| **Trust through tests** | Tests encode expected behavior explicitly | Automated validation |

### Comparison to Direct Generation

| Aspect | Direct Generation | Progressive Prompting |
|--------|-------------------|----------------------|
| Validation points | 1 (code works or not) | 4 (req, design, tests, code) |
| Debug difficulty | High (where did it go wrong?) | Low (stage that failed is clear) |
| Change cost | High (regenerate all code) | Low (regenerate from failing stage) |
| Operator confidence | Low (blind trust) | High (validated each step) |
| Time to first code | Fast | Slower (4 stages) |
| Time to correct code | Slow (many regenerations) | Fast (catch errors early) |

---

## Performance Results

### Case Study: Web Application

The paper presents a case study of building a web application:

| Metric | Direct Generation | Progressive Prompting |
|--------|-------------------|----------------------|
| Requirements captured | 12 / 15 (80%) | 15 / 15 (100%) |
| Design errors | 4 | 1 |
| Tests generated | 8 | 27 |
| Code regenerations needed | 5 | 1 |
| Total LLM calls | 6 | 8 |
| Operator effort (hours) | 8 | 4 |
| Final code quality | Pass 67% tests | Pass 96% tests |

**Key finding:** More upfront structure → fewer downstream fixes → lower total cost.

---

## Practical Implications

### For Knowledge & Vibes

Perfect alignment with K&V workflow:

| Progressive Stage | K&V Phase | Artifact |
|-------------------|-----------|----------|
| Requirements extraction | Ideation | REQ-* statements |
| Design generation | Architecture | ADR-* documents |
| Test generation | Planning | AC-* test cases |
| Code generation | Execution | Bead implementation |
| Human checkpoints | Calibration | Review gates |

### Integration Points

```markdown
## K&V Progressive Pipeline

Phase 1: IDEATION (Operator-Led)
├─ Natural language goals
├─ LLM: Extract structured requirements
├─ Output: REQ-001, REQ-002, REQ-003...
└─ CHECKPOINT: Operator validates completeness

Phase 2: ARCHITECTURE (Operator + LLM)
├─ Input: All REQ-* statements
├─ LLM: Generate design alternatives
├─ Output: ADR-* documents with tradeoffs
└─ CHECKPOINT: Operator selects approach

Phase 3: PLANNING (LLM)
├─ Input: REQ-* + ADR-*
├─ LLM: Generate test specifications
├─ Output: AC-* per requirement
└─ CHECKPOINT: Operator validates expected behavior

Phase 4: DECOMPOSITION (LLM)
├─ Input: REQ-* + ADR-* + AC-*
├─ LLM: Generate bead breakdown
├─ Output: Bead DAG with dependencies
└─ CHECKPOINT: Operator reviews plan

Phase 5: EXECUTION (Agents)
├─ Input: Bead with REQ + AC + context
├─ Agent: Implement to pass AC
├─ Output: Code + test results
└─ VALIDATION: Automated (tests must pass)
```

---

## Implementation Checklist

### For Operator Interface
- [ ] Template for natural language requirements input
- [ ] LLM prompt for requirements extraction (Stage 1)
- [ ] Review UI for validating structured requirements
- [ ] LLM prompt for design generation (Stage 2)
- [ ] Visualization for design artifacts (class diagrams, etc.)
- [ ] LLM prompt for test generation (Stage 3)
- [ ] Test review UI showing expected behaviors
- [ ] Automatic progression to code gen when all checkpoints pass

### For Agent System
- [ ] Receive: REQ + Design + Tests as bead context
- [ ] Generate code to pass tests
- [ ] Run tests automatically
- [ ] Report: pass/fail status
- [ ] If fail: iterate with test feedback (max 3 attempts)
- [ ] If stuck: decompose (ADaPT)

---

## Key Takeaways

1. **Requirements are leverage** — High-quality requirements dramatically improve downstream code quality
2. **Progressive > direct** — Multi-stage pipeline with checkpoints beats one-shot generation
3. **Human-readable gates** — Operators can validate requirements, design, and tests without code expertise
4. **Fail fast, fix cheap** — Catching errors at requirements stage avoids costly code rework
5. **Tests as contract** — Tests encode expected behavior in operator-understandable form

---

## Limitations

### Research Scope
- **Case study methodology** — Single web application case study, not controlled experiment
- **Limited scale** — Small application, not enterprise-scale system
- **One domain** — Web development, may not generalize to embedded systems, etc.

### Practical Constraints
- **More LLM calls** — 4 stages means 4+ LLM invocations (higher cost)
- **Operator time** — Checkpoints require human review (slower than fully automated)
- **Learning curve** — Operators must learn to validate design artifacts

### Open Questions
- **Optimal checkpoint granularity** — When to split stages further?
- **Design notation** — What format works best for nontechnical operators?
- **Automated checkpoint validation** — Can some checkpoints be automated?

---

## See Also

- `034-llms-in-re-guideline.md` — Systematic guideline for LLMs in requirements engineering
- `035-llm-vs-human-re.md` — LLM vs. human requirements quality comparison
- `036-requirements-qa-iso-29148.md` — Requirements quality assurance with LLMs
- `037-requirements-to-code-practices.md` — Practitioner workflows for requirements to code
- `031-alphacodium.md` — Flow engineering (another progressive pipeline approach)
