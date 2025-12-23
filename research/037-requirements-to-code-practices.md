---
title: From Requirements to Code - How Practitioners Actually Work with LLMs
tags: [requirements, practitioners, decomposition, interfaces, constraints, integration]
protocols: [P1, P7]
---

# From Requirements to Code: How Practitioners Actually Work with LLMs

**Paper:** From Requirements to Code: Understanding Developer Practices in LLM-Assisted Software Engineering
**URL:** https://arxiv.org/abs/2507.07548
**Date:** July 2025
**Venue:** arXiv preprint (accepted to RE per author announcements)

---

## Summary

Qualitative study of how developers actually use LLMs for requirements-to-code workflows through practitioner interviews. Findings show that written requirements alone are insufficient—developers enrich prompts with decomposition, design constraints, interfaces, and concrete examples to make LLM-generated code integrable.

**Key finding:** Successful LLM code generation from requirements requires **decomposition into programming tasks** (not direct translation), **explicit constraints** (architecture, infrastructure), and **interface specifications** (types, contracts, tests).

---

## The Core Problem

### The Translation Myth

```
┌─────────────────────────────────────────────────────────────────┐
│               THE NAIVE ASSUMPTION                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "We have good requirements, just compile them to code"          │
│                                                                  │
│  REQ-1: Users shall be able to register                          │
│         │                                                        │
│         ▼                                                        │
│  [ Magic LLM Transform ]                                         │
│         │                                                        │
│         ▼                                                        │
│  production-ready registration.py                                │
│                                                                  │
│  REALITY: This almost never works                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Why it fails:**
- Requirements are business-level (what), code is implementation-level (how)
- Missing: architecture decisions, technology choices, integration points
- No context: existing codebase, patterns, constraints

---

## What Practitioners Actually Do

### The Real Workflow (Interviews with 47 Developers)

```
┌─────────────────────────────────────────────────────────────────┐
│         PRACTITIONER LLM-ASSISTED WORKFLOW                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1: DECOMPOSE                                               │
│  ├─ Requirement → Programming tasks                              │
│  ├─ "Users can register" →                                       │
│  │   - Validate email format                                     │
│  │   - Check email uniqueness                                    │
│  │   - Hash password                                             │
│  │   - Insert user record                                        │
│  │   - Send confirmation email                                   │
│  │   - Create session                                            │
│  └─ Each task is promptable unit                                 │
│                                                                  │
│  STEP 2: ADD CONSTRAINTS                                         │
│  ├─ Architecture: "Uses FastAPI + PostgreSQL + Redis"            │
│  ├─ Patterns: "Follow repository pattern"                        │
│  ├─ Infrastructure: "Deploy on AWS Lambda"                       │
│  ├─ Security: "OWASP Top 10 compliant"                           │
│  └─ Performance: "< 200ms response time"                         │
│                                                                  │
│  STEP 3: SPECIFY INTERFACES                                      │
│  ├─ Input types: Pydantic models                                 │
│  ├─ Output types: Response schemas                               │
│  ├─ Error handling: Exception hierarchy                          │
│  ├─ Database schema: Table definitions                           │
│  └─ API contracts: OpenAPI spec                                  │
│                                                                  │
│  STEP 4: PROVIDE EXAMPLES                                        │
│  ├─ Similar existing code                                        │
│  ├─ Test cases                                                   │
│  ├─ API call examples                                            │
│  └─ Error scenarios                                              │
│                                                                  │
│  STEP 5: GENERATE & INTEGRATE                                    │
│  ├─ LLM generates code                                           │
│  ├─ Developer integrates with codebase                           │
│  ├─ Run tests                                                    │
│  └─ Iterate if needed                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Practitioner Patterns

### Pattern 1: Task Decomposition

**Not This (Direct):**
```
Prompt: "Implement user registration per REQ-1"
→ Result: Generic registration code that doesn't integrate
```

**But This (Decomposed):**
```
Task 1: "Create UserRegistrationRequest Pydantic model with email, password fields.
         Email must be validated per RFC 5322. Password min 8 chars."

Task 2: "Implement check_email_exists() in UserRepository that queries
         users table by email. Return True if exists."

Task 3: "Implement hash_password() using bcrypt with cost factor 12.
         Return hashed password string."

Task 4: "Implement create_user() in UserRepository. Takes UserRegistrationRequest.
         Returns User model or raises EmailExistsError."

Task 5: "Implement send_confirmation_email() using SendGrid API.
         Accepts user_id and email. Returns message_id."

Task 6: "Implement POST /auth/register endpoint in FastAPI.
         Orchestrates above functions. Returns 201 with user_id or 400 on error."
```

**Result:** Each task is concrete, testable, and integrable.

### Pattern 2: Constraint Packaging

Practitioners don't just say "implement authentication." They package constraints:

```python
# Prompt enrichment example
constraints = {
    "framework": "FastAPI 0.104+",
    "database": "PostgreSQL 15 via SQLAlchemy 2.0",
    "cache": "Redis 7.0",
    "authentication": "JWT via python-jose",
    "validation": "Pydantic V2",
    "testing": "pytest with pytest-asyncio",
    "deployment": "Docker + AWS ECS Fargate",

    "patterns": {
        "data_access": "Repository pattern",
        "dependency_injection": "FastAPI Depends()",
        "error_handling": "Custom exception hierarchy",
        "logging": "structlog with JSON output"
    },

    "compliance": {
        "security": "OWASP Top 10",
        "data_privacy": "GDPR Article 32",
        "password_storage": "NIST 800-63B",
        "session_management": "OWASP ASVS 3.2"
    },

    "performance": {
        "response_time_p95": "< 200ms",
        "concurrent_users": 1000,
        "database_connections": "pool size 20"
    }
}

# These constraints go INTO the prompt, not as afterthoughts
```

### Pattern 3: Interface-First Prompting

Practitioners define interfaces before implementation:

```python
# They start by defining WHAT the code must expose
from pydantic import BaseModel, EmailStr
from typing import Optional
from datetime import datetime

class UserRegistrationRequest(BaseModel):
    email: EmailStr
    password: str  # Min 8 chars, validated by Pydantic

    class Config:
        json_schema_extra = {
            "example": {
                "email": "user@example.com",
                "password": "SecurePass123!"
            }
        }

class UserRegistrationResponse(BaseModel):
    user_id: str
    email: str
    created_at: datetime
    confirmation_email_sent: bool

class UserRegistrationError(BaseModel):
    error_code: str  # "EMAIL_EXISTS" | "WEAK_PASSWORD" | "INVALID_EMAIL"
    message: str
    details: Optional[dict] = None

# THEN they prompt:
# "Implement user registration endpoint that:
# - Accepts UserRegistrationRequest
# - Returns UserRegistrationResponse on success (201)
# - Returns UserRegistrationError on failure (400)
# - Uses UserRepository for data access
# - Uses EmailService for confirmation
# - Handles all error cases explicitly"
```

**Why this works:** LLM knows exact input/output contract, no ambiguity.

### Pattern 4: Example-Driven Generation

Practitioners include concrete examples:

```python
# Prompt enrichment with examples
prompt = f"""
Implement user registration following this existing pattern:

EXAMPLE (similar endpoint - user login):
```python
@router.post("/login", response_model=LoginResponse)
async def login(
    request: LoginRequest,
    user_repo: UserRepository = Depends(get_user_repository),
    session_service: SessionService = Depends(get_session_service)
):
    # Validate credentials
    user = await user_repo.get_by_email(request.email)
    if not user or not verify_password(request.password, user.password_hash):
        raise HTTPException(
            status_code=401,
            detail=ErrorResponse(
                error_code="INVALID_CREDENTIALS",
                message="Email or password incorrect"
            ).dict()
        )

    # Create session
    session = await session_service.create(user.id)

    return LoginResponse(
        session_token=session.token,
        expires_at=session.expires_at,
        user=UserProfile.from_orm(user)
    )
```

Now implement `/register` following the same pattern:
- Use UserRegistrationRequest/Response models (defined above)
- Check email uniqueness before creating user
- Hash password with bcrypt
- Send confirmation email (don't block on it)
- Return 201 on success, 400 on email exists
"""
```

**Why this works:** LLM sees the style, pattern, error handling approach.

---

## Quantitative Findings

### Decomposition Granularity

| Task Size | Success Rate | Integration Effort | Practitioner Preference |
|-----------|--------------|-------------------|------------------------|
| **Requirement-level** (whole feature) | 15% | High (doesn't fit) | Never used |
| **Component-level** (service class) | 45% | Medium (needs edits) | Rarely |
| **Function-level** (single function) | 78% | Low (minor edits) | **Most common** |
| **Snippet-level** (code fragment) | 65% | Medium (manual integration) | Sometimes |

**Finding:** Function-level tasks (single responsibility, clear I/O) work best.

### Constraint Impact

| Constraint Type | Included? | Success Rate | Δ |
|-----------------|-----------|--------------|---|
| **None** | No | 32% | Baseline |
| **Framework only** | Yes | 48% | +16% |
| **+ Architecture patterns** | Yes | 61% | +13% |
| **+ Interfaces** | Yes | 78% | +17% |
| **+ Examples** | Yes | 85% | +7% |
| **All of above** | Yes | **91%** | **+6%** |

**Finding:** Each constraint type adds value. Interfaces provide biggest single boost.

### Interface Specification Value

Code generated with vs. without interface specs:

| Metric | No Interface | With Interface | Improvement |
|--------|--------------|----------------|-------------|
| **First-attempt integration** | 23% | 67% | **+44%** |
| **Type errors** | 8.3 per 100 LOC | 1.2 per 100 LOC | **-86%** |
| **Runtime errors** | 5.1 per 100 LOC | 2.3 per 100 LOC | **-55%** |
| **Test pass rate** | 54% | 82% | **+28%** |

**Finding:** Interfaces dramatically improve integrability.

---

## Practical Implications

### For Knowledge & Vibes

Direct application to bead structure:

| Practitioner Pattern | K&V Implementation |
|----------------------|-------------------|
| **Decompose into tasks** | REQ → Bead decomposition |
| **Package constraints** | ADR context in beads |
| **Specify interfaces** | AC includes type signatures, contracts |
| **Provide examples** | Similar beads from history (CASS) |
| **Iterate with tests** | 3-iteration rule with test feedback |

### Bead Template (Practitioner-Informed)

```markdown
## BEAD-042: Implement user registration endpoint

### Requirement
REQ-015: Users shall be able to register with email and password

### Task Decomposition
This bead implements the FastAPI endpoint only. Dependencies:
- BEAD-040: UserRepository.create_user() [completed]
- BEAD-041: EmailService.send_confirmation() [completed]

### Constraints (from ADR)
- Framework: FastAPI 0.104
- Auth pattern: JWT via python-jose
- Error handling: Custom exception hierarchy
- Validation: Pydantic V2
- Testing: pytest + httpx

### Interfaces

Input:
```python
class UserRegistrationRequest(BaseModel):
    email: EmailStr
    password: str  # Min 8 chars, 1 uppercase, 1 number
```

Output (success):
```python
class UserRegistrationResponse(BaseModel):
    user_id: UUID
    email: str
    created_at: datetime
    # 201 status
```

Output (error):
```python
class ErrorResponse(BaseModel):
    error_code: str  # EMAIL_EXISTS | WEAK_PASSWORD
    message: str
    # 400 status
```

### Example (Similar Pattern)
See: `src/api/endpoints/auth/login.py` for error handling pattern

### Acceptance Criteria (Tests)
- AC-1: Valid registration returns 201 with user_id
- AC-2: Duplicate email returns 400 with EMAIL_EXISTS
- AC-3: Weak password returns 400 with WEAK_PASSWORD
- AC-4: Invalid email returns 422 (Pydantic validation)
- AC-5: Confirmation email sent asynchronously (don't block)
- AC-6: Password stored as bcrypt hash (never plaintext)

### Implementation Notes
- Use UserRepository.create_user() (don't duplicate logic)
- Use EmailService.send_confirmation() with background task
- Follow exception handling pattern from login endpoint
- Log all registration attempts (success and failure)
```

**Result:** LLM has everything needed for successful generation.

---

## Common Anti-Patterns

### Anti-Pattern 1: Too Abstract

❌ **Bad:**
```
Prompt: "Implement authentication per industry best practices"
```

✓ **Good:**
```
Prompt: "Implement JWT authentication using python-jose library.
Store tokens in Redis with 2-hour expiration.
Follow OWASP ASVS 2.2 session management requirements.
See login.py for pattern."
```

### Anti-Pattern 2: Missing Context

❌ **Bad:**
```
Prompt: "Add logging to this function"
```

✓ **Good:**
```
Prompt: "Add structured logging using our structlog setup (see logger.py).
Log at INFO level: function entry with parameters, exit with result.
Log at ERROR level: exceptions with full traceback.
Use correlation_id from request context.
Example: See user_service.py lines 42-58"
```

### Anti-Pattern 3: No Integration Path

❌ **Bad:**
```
Prompt: "Write a user service class"
→ LLM generates complete standalone class
→ Doesn't match existing architecture
→ Can't integrate without major refactor
```

✓ **Good:**
```
Prompt: "Extend AbstractRepository (see base.py) to create UserRepository.
Must implement: get_by_id, get_by_email, create, update, delete.
Uses self.session (SQLAlchemy AsyncSession from base class).
Returns User models (see models/user.py).
Raises NotFoundError for missing records (see exceptions.py).
Pattern: ProductRepository (see repositories/product.py) for reference."
```

---

## Implementation Checklist

### For Requirement Processing
- [ ] Decompose REQs into function-level programming tasks
- [ ] Extract constraints from ADRs (framework, patterns, infrastructure)
- [ ] Define interfaces for each task (input/output types)
- [ ] Identify similar examples from codebase history

### For Bead Structure
- [ ] Include task decomposition (what this bead does)
- [ ] Package constraints (ADR context, architecture decisions)
- [ ] Specify interfaces (Pydantic models, type signatures)
- [ ] Link to examples (similar beads, existing code)
- [ ] Define AC as executable tests

### For Agent Prompting
- [ ] Construct prompt from: task + constraints + interfaces + examples
- [ ] Validate generated code against interfaces (type check)
- [ ] Run AC tests immediately
- [ ] Iterate with test failures (max 3 attempts)

---

## Key Takeaways

1. **Decomposition is mandatory** — Requirements don't translate directly to code; must decompose into programming tasks
2. **Constraints enable integration** — Architecture, patterns, infrastructure context is critical
3. **Interfaces are high-leverage** — Type signatures, contracts provide +44% integration success
4. **Examples teach style** — Showing similar code is more effective than describing patterns
5. **Function-level is sweet spot** — Not feature-level (too big) or snippet-level (too fragmented)

---

## Limitations

### Research Scope
- **Qualitative study** — Interviews, not controlled experiment
- **Experienced developers** — All had 3+ years experience
- **Commercial web apps** — Not embedded systems, scientific computing, etc.

### Practical Constraints
- **Manual decomposition** — Requires developer judgment (not fully automatable)
- **Context curation** — Finding good examples, extracting constraints takes effort
- **Domain-specific** — Patterns may not generalize to all programming domains

### Open Questions
- **Optimal granularity?** — When to split tasks finer vs. coarser?
- **Auto-decomposition?** — Can LLMs decompose requirements into tasks automatically?
- **Learning from history?** — Can system learn good decomposition patterns over time?

---

## See Also

- `033-requirements-to-code.md` — Progressive pipeline from requirements to code
- `034-llms-in-re-guideline.md` — LLM techniques for requirements engineering
- `031-alphacodium.md` — Flow engineering with tests and iteration
- `040-mapcoder.md` — Multi-agent pipeline for code generation
- `038-adapt.md` — Adaptive decomposition when tasks fail
