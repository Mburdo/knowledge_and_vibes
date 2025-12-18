# Planning and Decomposition

**How to turn an idea into executable beads that agents can implement flawlessly.**

---

## The Core Problem

You have a big goal: "Build a payment system" or "Add real-time collaboration" or "Migrate to new architecture."

Dumping this into one task creates:
- Unclear progress tracking
- No parallelization opportunities
- Overwhelming scope for agents
- Merge conflicts when multiple agents work

**But there's a deeper problem**: Agents perform poorly on large documents.

---

## Why Agents Need Small Documents

**This is not about your human convenience. It's about agent performance and this means doing extra leg work to make the agent's job as easy as possible.**

In my experience agents work best on files under **1000 lines**, preferably closer to **500 lines**. Give an agent a 5000-line planning document and it **will** turn that into subtasks horribly.

What happens with large documents:
- **Context bombing**: The agent gets overwhelmed and starts skipping detail
- **Lazy summarization**: You'll see vague outputs like "implement authentication" instead of specific methods, types, and tests
- **Content loss**: Your carefully written edge cases, integration points, and method signatures get paraphrased away

**Your plan contains critical detail.** Specific methods, edge cases, validation rules, integration points. When you hand a 5000-line doc to an agent, that detail gets summarized away. The agent can't hold it all in working memory, so it approximates.

**The solution: make content digestible.**

By breaking your plan into phases BEFORE giving it to an agent:
- Each phase is small enough to fit in working memory
- The agent can preserve ALL the detail from your plan
- You get LOSSLESS decomposition instead of lossy summarization

Think of it like chunking for LLMs — you wouldn't stuff 100k tokens into a prompt and expect perfect recall. Same principle applies here.

---

## The Complete Planning Workflow

```
1. IDEATION (Frontier Model - ITERATIVE)
   └─ Use Opus 4.5 (ultrathink), GPT 5.2 Xtra High, Gemini 3 Pro
   └─ Multiple sessions: explore → debate → research → specify
   └─ Output: 3,000-5,000+ line comprehensive plan

2. PHASE BREAKDOWN (Human + Agent Collaboration)
   └─ YOU break the massive plan into phases (agent can assist)
   └─ Each phase is a logical chunk (~1-2 weeks of work)
   └─ Phases should be ~500-1000 lines max
   └─ NEVER let the agent do this unsupervised

3. TASK DECOMPOSITION (Agent + /decompose-task)
   └─ Feed ONE phase to the agent
   └─ Agent creates parent bead + atomic sub-beads
   └─ Each sub-bead is ~500 lines, 30-120 minutes of work

```

---

## Phase 1: Ideation with Frontier Models

Use the most powerful reasoning models available:
- **Claude Opus 4.5** (use ultrathink mode)
- **GPT 5.2 Xtra High** with high reasoning effort
- **Gemini 3 Pro** in thinking mode

### The Goal

Transform a vague idea into a comprehensive, detailed plan that includes things like:
- Architecture decisions with rationale
- Data structures and type definitions
- API contracts and endpoints
- Error handling and edge cases
- Testing strategy
- Integration points

### The Iterative Ideation Process

**This is NOT a single prompt.** Good plans emerge from multiple sessions of reasoning, debate, research, and refinement. You're having a conversation with the frontier model over many iterations. The following (Session 1, Session 2, Session 3) are not meant to be absolute, i'm simply trying to show how the idea evolves into a plan.

#### Session 1: Initial Exploration

Start with high-level questions. Let the model reason through the problem space:

```
I want to build [FEATURE] for [PROJECT].

Here's my existing architecture: [paste CLAUDE.md, codemaps]

What are the major components we'd need?
What are the key architectural decisions we'll face?
What are the risks and tradeoffs?
```

Don't accept the first answer. Ask follow-up questions. Challenge assumptions. Push for alternatives.

#### Session 2: Decision Debates

For each major architectural decision, have the model argue both sides and offer multiple options(A,B,C):

```
You suggested using JWT for auth. Let's debate this.

First, make the strongest case FOR JWTs.
Now make the strongest case AGAINST JWTs.
What would we use instead?
Given our constraints [X, Y, Z], which is actually better and why?
```

This surfaces edge cases and considerations you'd miss with a single-pass answer.

#### Session 3: Research and Grounding

Have the model research current best practices. Don't trust training data:

```
Search for current best practices on [specific technical decision].
What do the 2024/2025 docs say about [library/pattern]?
Are there any deprecations or breaking changes we should know about?
What's the community consensus on [approach]?
```

Ground every external dependency in current reality.

#### Session 4: Specification Deep Dives

Once decisions are made through debate, get extremely specific:

```
Now let's spec out the data structures for [component].

For each model, I need:
- Every field with exact types and validation rules
- Relationships, indexes, constraints
- Example instances showing valid and invalid data
```

#### Session 5: Failure Mode Analysis

```
What are ALL the ways [component] could fail?
For each failure mode:
- What causes it?
- How do we detect it?
- How do we handle it?
- What test would catch it?

Write the actual test code, not descriptions.
```

#### Session 6: Integration and Anti-Patterns

```
How does [component] connect to our existing [X, Y, Z]?
Draw the data flow.

What mistakes would a developer likely make implementing this?
What should we explicitly NOT do, and why?
```

### Why Iteration Beats Single Prompts

| Single Prompt | Iterative Sessions |
|---------------|-------------------|
| Model guesses at decisions | Decisions are debated from multiple angles |
| Generic best practices from training | Researched against current documentation |
| Surface-level coverage | Deep exploration of edge cases |
| One perspective | Multiple perspectives argued and reconciled |
| Assumptions go unchallenged | Every assumption questioned |

### What You're Building Toward

Through these iterations, you accumulate the components of a comprehensive plan:

| Component | How It Emerges |
|-----------|---------------|
| Architecture Decisions (ADRs) | Session 2 debates, with alternatives considered |
| Data Structures | Session 4 deep dives, with validation rules |
| Error Handling | Session 5 failure analysis |
| Test Code | Session 5, actual code not descriptions |
| Anti-Patterns | Session 6 + lessons from debates |
| Integration Points | Session 6 + architecture discussions |

The final plan is a **synthesis** of all these sessions — not output from one prompt.

### What the Massive Plan MUST Contain

This is not optional. Every massive plan from ideation should include:

| Section | What It Contains | Why It Matters |
|---------|-----------------|----------------|
| **Architecture Decisions** | ADR tables with Decision, Choice, Alternatives, Rationale | Agents need to understand WHY, not just WHAT |
| **Data Structures** | Full type definitions, validation rules, field-by-field specifications | No guessing about schema |
| **API Contracts** | Request/response shapes, status codes, error formats | Exact contracts, not descriptions |
| **Error Handling** | Every failure mode, exception types, recovery strategies | Agents won't invent error handling |
| **Full Code Examples** | Actual implementation code with imports, not pseudocode | Copy-paste ready |
| **Testing Strategy** | All 4 types: unit, integration, e2e, property-based | With actual test code, not "4 tests for X" |
| **Anti-Patterns** | What NOT to do with explanations | Prevents common mistakes |
| **Integration Points** | How this connects to existing systems | System context |
| **Dependencies** | What must exist before, what this enables | Ordering requirements |
| **Acceptance Criteria** | How to verify it's done correctly | Definition of done |

### The "Any Agent Can Implement" Test

Your massive plan passes if:

> Any agent picks up any section → Implements it perfectly without needing external docs

If an agent would need to ask "but how exactly should I...?" then the plan lacks detail.

### Example: Inadequate vs Adequate Plan Content

**INADEQUATE (Don't Do This):**
```
## User Authentication

Add user authentication using JWT tokens. The system should validate
users and issue tokens for API access. Include appropriate tests.
```

This is worthless. What token format? What validation rules? What error codes? What tests exactly?

**ADEQUATE (Do This):**
```
## User Authentication

### Architecture Decision Record

| Decision | Choice | Alternatives | Rationale |
|----------|--------|--------------|-----------|
| Token format | JWT (HS256) | Session cookies, opaque tokens | Stateless, works with mobile apps |
| Token storage | HTTP-only cookie | localStorage, memory | XSS protection |
| Token lifetime | 15 min access, 7 day refresh | Longer access tokens | Balance UX with security |

### Data Structures

```python
from pydantic import BaseModel, EmailStr
from datetime import datetime

class TokenPayload(BaseModel):
    sub: str  # user_id
    exp: datetime
    iat: datetime
    token_type: Literal["access", "refresh"]

class AuthResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
    expires_in: int  # seconds
```

### Error Handling

| Error Code | HTTP Status | When | Response |
|------------|-------------|------|----------|
| INVALID_CREDENTIALS | 401 | Wrong email/password | {"error": "invalid_credentials", "message": "..."} |
| TOKEN_EXPIRED | 401 | Access token expired | {"error": "token_expired", "message": "..."} |
| TOKEN_INVALID | 401 | Malformed/tampered token | {"error": "token_invalid", "message": "..."} |

### Tests

```python
class TestAuthHappyPath:
    async def test_login_returns_tokens(self, client, user_factory):
        user = await user_factory.create(password="testpass123")
        response = await client.post("/auth/login", json={
            "email": user.email,
            "password": "testpass123"
        })
        assert response.status_code == 200
        assert "access_token" in response.json()

    async def test_refresh_token_issues_new_access(self, client, auth_tokens):
        response = await client.post("/auth/refresh",
            cookies={"refresh_token": auth_tokens.refresh})
        assert response.status_code == 200
        # New access token should be different
        assert response.json()["access_token"] != auth_tokens.access

class TestAuthEdgeCases:
    async def test_login_wrong_password(self, client, user_factory):
        user = await user_factory.create(password="correct")
        response = await client.post("/auth/login", json={
            "email": user.email,
            "password": "wrong"
        })
        assert response.status_code == 401
        assert response.json()["error"] == "invalid_credentials"
```

### Anti-Patterns

- ❌ **Don't store tokens in localStorage** — XSS vulnerability
- ❌ **Don't use long-lived access tokens** — Increases window of attack
- ❌ **Don't include sensitive data in JWT payload** — JWTs are not encrypted
```

The second example gives an agent everything needed to implement correctly.

---

## Phase 2: Collaborative Phase Breakdown

**This step requires YOUR active participation. The agent can help, but you MUST stay involved.**

The massive plan from Phase 1 (often 3000-5000+ lines) needs to be chunked into digestible phases before agents can work with it effectively. You have two options:

1. **Do it yourself** — Read the plan, identify logical boundaries, create phases manually
2. **Work with an agent** — Have the agent help you break it down, but with extreme care

### Why There's No Command For This

There is intentionally **no `/break-into-phases` command**. Here's why:

This step requires the same LOSSLESS thinking as `/decompose-task`, but at a larger scale with higher stakes. If you hand an agent a 5000-line plan and say "break this into phases," I promise you:

- **Critical details will be lost** — The agent will summarize instead of preserve
- **Dependencies will be missed** — Subtle ordering requirements get overlooked
- **Scope will be wrong** — Some phases too big, others too small
- **You'll throw away all your work** — The plan you carefully created gets mangled

The agent can absolutely help you, but it needs to work **WITH** you, not **FOR** you.

### The Collaborative Process

**Step 1: Prime the agent with context**

Before asking for phase breakdown, give the agent:
- The full plan (yes, all 5000 lines)
- Your architecture docs and codemaps
- Any constraints (team size, deadlines, parallel work needs)

```
I have a comprehensive plan for [FEATURE]. It's [X] lines.

I need to break this into phases for agent execution. Each phase
should be 500-1000 lines, represent a logical chunk of work, and
have clear dependencies.

Help me identify the natural boundaries. Don't create the phases
yet — just tell me where you see the logical breakpoints and why.
```

**Step 2: Review the proposed boundaries TOGETHER**

The agent will suggest breakpoints. For EACH one, validate:

| Question | Why It Matters |
|----------|---------------|
| Does this boundary make sense architecturally? | Phases should be cohesive units |
| What must exist before this phase can start? | Dependencies must be explicit |
| Can this phase be tested independently? | Isolated verification is critical |
| Is the size right (~500-1000 lines)? | Too big = context bombing, too small = overhead |

Push back. Ask questions. The agent will miss things — that's guaranteed.

**Step 3: Iteratively refine**

Common corrections you'll need to make:

- **"Phase 3 is too big"** — Ask agent to split it further
- **"These two phases are too coupled"** — Merge them or reorder
- **"You missed the error handling section"** — Point out what was skipped
- **"Phase 2 depends on Phase 4"** — Fix the ordering

```
Looking at your proposed Phase 3, I see it's ~1800 lines which is
too large. Also, I notice you didn't mention the rate limiting
logic from lines 2340-2450 of the original plan.

Let's split Phase 3 and make sure that rate limiting content
lands somewhere. Where should it go?
```

**Step 4: Verify NOTHING was lost**

Before finalizing phases, explicitly verify:

```
Before we finalize these phases, I need you to verify that
EVERY section from the original plan is assigned to a phase.

Go through the original plan section by section and tell me
which phase each section maps to. Flag anything that wasn't
assigned.
```

The agent should produce a mapping like:

```
Original Plan Section → Phase Assignment
─────────────────────────────────────────
Lines 1-150: Architecture Overview → Phase 0
Lines 151-340: Database Schema → Phase 1
Lines 341-520: User Model → Phase 1
Lines 521-890: Authentication Flow → Phase 2
...
Lines 4800-4950: Deployment Notes → Phase 6
Lines 4951-5020: Monitoring Setup → Phase 6

✅ All sections assigned. No orphans.
```

If anything is missing, fix it before proceeding.

**Step 5: Create the phase documents**

Only AFTER you've validated the breakdown, have the agent create the actual phase documents:

```
Now create the Phase 1 document. Remember:
- Copy content VERBATIM from the original plan
- Include EVERY detail from sections [X-Y]
- Include all relavent detail from the original plan and elaborate where appropriate
- Do not summarize or paraphrase
- Target size: 500-1000 lines
```

Repeat for each phase, validating each one.

### What The Agent WILL Miss (Without Your Help)

From experience, agents consistently miss these when breaking down large plans:

| Commonly Missed | Why |
|----------------|-----|
| **Cross-cutting concerns** | Error handling, logging, monitoring scattered across sections |
| **Implicit dependencies** | "This assumes X exists" buried in prose |
| **Edge cases** | Validation rules, boundary conditions in detailed specs |
| **Integration points** | How components connect, mentioned once then assumed |
| **Anti-patterns** | "Don't do X" warnings that get summarized away |
| **Rationale** | The WHY behind decisions, lost to "implement X" |

Your job is to catch these. If you catch one, you need to rip it open and ask explicitly about EVERYTHING realted to that phase. If it missed one thing, it likely missed two:

- "What error handling did you include in each phase?"
- "Where does the rate limiting logic go?"
- "Did you preserve the anti-patterns section?"
- "Which phase owns the integration with [X]?"

### The Alternative: Do It Yourself

If the plan is under 2000 lines, or you know the domain well, it's often faster to just do this yourself:

1. Read the entire plan
2. Identify 4-8 natural boundaries
3. Create phase documents by copying sections
4. Verify completeness with a checklist

The agent-assisted approach is valuable when:
- The plan is very large (3000+ lines)
- You are not a technical person
- The domain is complex and you want a second perspective
- You want to validate your intuition about boundaries

But never let the agent do this unsupervised. The stakes are too high.

---

## Guidelines for Phase Breakdown

Whether you do this yourself or with agent assistance, follow these guidelines.

| Aspect | Guideline |
|--------|-----------|
| Size | 500-1000 lines per phase |
| Scope | Logical, cohesive unit |
| Dependencies | Clear what must come before/after |
| Testability | Phase should be independently verifiable |

---

## What Phase Documents MUST Contain

Each phase document is a self-contained specification. When you break the massive plan into phases, each phase document must have:

### 10 Mandatory Sections

| Section | Content | Fail If Missing |
|---------|---------|----------------|
| **1. Context & Motivation** | WHY this phase exists, what risks it addresses, user value | Agent won't understand the purpose |
| **2. Architecture Decision Record** | Decisions table with Choice, Alternatives, Rationale | Agent will make different choices |
| **3. System Integration Map** | ASCII diagram showing how components connect | Agent won't understand data flow |
| **4. Prerequisites** | What must exist before starting, with reasons | Agent will hit blockers |
| **5. Detailed Specification** | Full inputs, outputs, data structures, validation rules, error conditions | Agent will guess wrong |
| **6. Implementation Guidance** | File locations, actual code with imports, anti-patterns | Agent will put code in wrong places |
| **7. Testing Specification** | All 4 test types with FULL code | Agent will write inadequate tests |
| **8. Verification Checklist** | Functional, quality, docs, security checks | No definition of done |
| **9. Cross-References** | Depends on, blocks, related beads, source docs | Agent won't understand ordering |
| **10. Acceptance Criteria** | Explicit "done when" conditions | Unclear completion state |

### Testing Specification Requirements

This is where most plans fail. Your phase document must include:

**Unit Tests (≥3 test classes):**
- `TestHappyPath` — Success scenarios
- `TestEdgeCases` — Boundary conditions
- `TestErrorHandling` — Failure modes

**Integration Tests:**
- Full API request/response cycles
- Database transaction tests

**E2E Tests (if applicable):**
- Playwright/Cypress test code
- User journey scenarios

**Property-Based Tests:**
- Hypothesis strategies and properties
- Invariant verification

Each test type must include **actual executable code**, not descriptions like "4 tests for validation."

Your tests need to be the gatekeeper for your agent to build against. PAY ATTENTION TO THE TESTS.

### Example Phase Document Structure

```markdown
# Phase 2: User Authentication

## 1. Context & Motivation

### WHY This Exists
Authentication is required before users can create strategies...

### What Happens If Built Wrong
- Security vulnerabilities (token theft)
- Poor UX (constant re-authentication)
- System instability (session management bugs)

### User Value
Users can securely access their accounts and manage strategies...

## 2. Architecture Decision Record

| Decision | Choice | Alternatives | Rationale |
|----------|--------|--------------|-----------|
| Auth method | JWT | Sessions, OAuth | Stateless, mobile-friendly |
| Token storage | HTTP-only cookies | localStorage | XSS protection |

## 3. System Integration Map

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │───▶│   FastAPI   │───▶│  PostgreSQL │
│  (Next.js)  │    │   /auth/*   │    │    users    │
└─────────────┘    └─────────────┘    └─────────────┘
        │                 │
        │                 ▼
        │          ┌─────────────┐
        └─────────▶│    Redis    │
                   │   sessions  │
                   └─────────────┘
```

## 4. Prerequisites

- [ ] Database migrations run (users table exists)
- [ ] Redis available for session storage
- [ ] Environment variables configured (JWT_SECRET, etc.)

## 5. Detailed Specification

### Data Structures

```python
class User(SQLModel, table=True):
    id: UUID = Field(default_factory=uuid4, primary_key=True)
    email: EmailStr = Field(unique=True, index=True)
    hashed_password: str
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

### Error Conditions

| Error | Cause | HTTP Status | Response |
|-------|-------|-------------|----------|
| INVALID_CREDENTIALS | Wrong password | 401 | {"error": "invalid_credentials"} |

## 6. Implementation Guidance

### File Locations

```
app/
├── api/v1/auth.py        # Endpoints
├── core/security.py      # JWT handling
├── services/auth.py      # Business logic
└── models/user.py        # User model
```

### Anti-Patterns

- ❌ Don't store plain text passwords
- ❌ Don't use short JWT secrets
- ❌ Don't expose user IDs in URLs

## 7. Testing Specification

### Unit Tests

```python
class TestAuthHappyPath:
    async def test_login_returns_tokens(self, client):
        # Full test implementation...

class TestAuthEdgeCases:
    async def test_empty_email_rejected(self, client):
        # Full test implementation...

class TestAuthErrorHandling:
    async def test_wrong_password_returns_401(self, client):
        # Full test implementation...
```

### Property-Based Tests

```python
from hypothesis import given, strategies as st

@given(email=st.emails())
def test_valid_emails_accepted(email):
    # Full test implementation...
```

## 8. Verification Checklist

### Functional
- [ ] Login returns valid JWT
- [ ] Invalid credentials return 401
- [ ] Token refresh works

### Quality
- [ ] All tests passing
- [ ] Coverage ≥90%
- [ ] No linting errors

### Security
- [ ] Passwords hashed with bcrypt
- [ ] JWT secret ≥32 chars
- [ ] Rate limiting enabled

## 9. Cross-References

- **Depends On:** Phase 1 (Database Setup)
- **Enables:** Phase 3 (Strategy CRUD)
- **Backend Beads:** lbe-hex.2.1, lbe-hex.2.2
- **Frontend Beads:** lfe-bbi.2.1

## 10. Acceptance Criteria

- [ ] User can register with email/password
- [ ] User can login and receive tokens
- [ ] Invalid logins are rejected with proper errors
- [ ] All tests passing
- [ ] Security checklist complete
```

### Size Verification

After creating a phase document, verify:

| Metric | Target | Action if Wrong |
|--------|--------|-----------------|
| Total lines | 500-1000 | Split if too large, merge if too small |
| Code blocks | ≥5 | Add missing implementation details |
| Test code | ≥100 lines | Add actual test implementations |
| Tables | ≥3 | Add missing decisions/errors/specs |

### Phase Breakdown Process

1. **Read the entire plan** — Understand the full scope
2. **Identify natural boundaries** — Where are the logical breakpoints?
3. **Order by dependencies** — What must exist before what?
4. **Name clearly** — Phase names should be self-explanatory
5. **Verify completeness** — Does every part of the plan have a phase?

### Example Phase Structure

```
Phase 0: Foundation (Database + Auth)
  - User model with proxy wallet
  - API credential storage
  - Authentication endpoints

Phase 1: Core Kernel (Pure Evaluation)
  - Strategy manifest schema
  - Condition DSL parser
  - Pure evaluation functions

Phase 2: Risk Layer (Governor)
  - Position limits
  - Loss thresholds
  - Rate limiting

Phase 3: Execution (Trading)
  - Order placement
  - Fill tracking
  - Position reconciliation
```

---

## Phase 3: Task Decomposition with `/decompose-task`

Now you feed ONE phase to an agent using the `/decompose-task` command.

### What the Agent Creates

For each phase, the agent creates:
1. **Parent bead** — The phase itself as an epic-level task
2. **Sub-beads** — Atomic tasks that sum to the parent

### Sub-Bead Structure

| Suffix | Content | Purpose |
|--------|---------|---------|
| `.0` | **Context Brief** | WHY this exists, architecture decisions, system integration map, prerequisites |
| `.1` | **Schema/Types** | Database migrations, type definitions, interfaces |
| `.2-.3` | **Implementation** | Core code with full imports, every method signature |
| `.4` | **Tests: Happy Path** | Success scenario tests with full code |
| `.5` | **Tests: Edge Cases** | Boundary conditions, unusual inputs |
| `.6` | **Tests: Error Handling** | Failure modes, exceptions |
| `.7` | **Tests: Property-Based** | Hypothesis/fuzzing tests for invariants |
| `.8` | **Tests: Integration** | Cross-component verification |
| `.9` | **Reference Data** | Constants, addresses, lookup tables |
| `.10` | **Verification Checklist** | Acceptance criteria, completion checks |

### The LOSSLESS Rule

**CRITICAL: Everything from the phase plan must appear in a sub-bead.**

This is the most important rule of decomposition:
- **NEVER** paraphrase or summarize
- **NEVER** write "see parent bead for details"
- **NEVER** skip "obvious" content
- **COPY** content verbatim, typos and all

### Verification Process

After decomposition, verify:
1. **Character count**: Sub-beads total >= original (overhead is expected)
2. **Content check**: Every section from original appears somewhere
3. **Standalone test**: Each sub-bead makes sense without the others

---

## What Goes in an Enhanced Bead

A properly enhanced bead is a **complete specification** that any agent can pick up and implement flawlessly.

### Required Sections

```markdown
# [Bead Title]

## Grounding Status
| Pattern | Source | Status |
|---------|--------|--------|
| [library.method] | [Exa query or doc URL] | ✅ Verified |

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

## Testing Specification

### Happy Path Tests
```python
def test_success_scenario():
    # Full test code here
```

### Edge Case Tests
```python
def test_boundary_condition():
    # Full test code here
```

### Property-Based Tests
```python
from hypothesis import given, strategies as st

@given(st.integers())
def test_invariant(value):
    # Full test code here
```

## Verification Checklist
- [ ] All data structures implemented
- [ ] All error cases handled
- [ ] All tests passing
- [ ] Integration verified
```

---

## Quality Control: The Audit Process (OPTIONAL BUT RECCOMENDED FOR HIGHLY COMPLEX TASKS)

After decomposition, a DIFFERENT agent audits the work.

### Audit Criteria

1. **LOSSLESS** — No content was lost or summarized
2. **VERBATIM** — Content was copied exactly, not paraphrased
3. **COMPLETE** — All sections from original appear in sub-beads
4. **STANDALONE** — Each sub-bead makes sense independently
5. **CORRECT STRUCTURE** — Sub-beads follow the standard pattern

### Audit Report Format

```markdown
## Audit Report: [bead-id]

**Auditor:** [agent-name]
**Date:** [date]
**Decomposer:** [original-agent]

### Summary
- **Status:** PASS / FAIL
- **Original chars:** [count]
- **Sub-bead chars:** [count]
- **Difference:** [+/- chars] ([explanation])

### Sub-Beads Reviewed
| ID | Title | Chars | Status |
|----|-------|-------|--------|
| .1 | Context Brief | 5,097 | ✅ OK |
| .2 | Schema | 2,037 | ✅ OK |

### Issues Found
[If FAIL, list specific problems]

### Recommendation
[PASS: close decomp task / FAIL: create REDO task]
```

### When Audits Fail

If an audit fails:
1. Create a REDO task for the decomposer
2. List specific issues to fix
3. Decomposer re-does the work
4. Another audit cycle

---

## Grounding: Preventing Hallucination

**AI-generated code follows training data, which may be outdated.**

Before implementing anything that touches external libraries, APIs, or frameworks, GROUND your plan in current documentation.

### The Grounding Process

1. **Identify external dependencies** — Libraries, APIs, frameworks
2. **Query Exa** — Get current documentation
3. **Verify patterns** — Check if your plan matches current best practices
4. **Update if needed** — Fix outdated patterns in your beads

### Example Grounding Session

```
# Check if pattern is current
web_search_exa("Python httpx AsyncClient 2024 2025 best practices")

# Verify specific method
get_code_context_exa("Pydantic v2 model_validator field_validator")

# Result: Confirmed @model_validator(mode='after') -> Self is current
```

### Adding Grounding Headers

After verification, add a grounding status table to your bead:

```markdown
## Grounding Status
| Pattern | Exa Query | Status |
|---------|-----------|--------|
| `@pytest.mark.asyncio` | "pytest-asyncio 2024 2025" | ✅ Current |
| `@model_validator(mode='after')` | "Pydantic v2 model_validator" | ✅ Current |
| `structlog.get_logger(__name__)` | "Python structlog 2024" | ✅ Current |
```

---

## Testing Requirements

Every bead must have comprehensive test specifications.

### Test Categories

| Category | Purpose | Example |
|----------|---------|---------|
| **Happy Path** | Verify correct behavior | Valid input → expected output |
| **Edge Cases** | Boundary conditions | Empty list, max int, null |
| **Error Handling** | Failure modes | Invalid input, network error |
| **Property-Based** | Invariants | "For all valid inputs, output is valid" |
| **Integration** | Cross-component | Full request → response cycle |

### Property-Based Testing with Hypothesis

```python
from hypothesis import given, strategies as st, settings
from hypothesis.stateful import RuleBasedStateMachine, rule, invariant

# Stateless property test
@given(st.lists(st.integers()))
def test_sort_idempotent(xs):
    assert sorted(sorted(xs)) == sorted(xs)

# Stateful property test
class DatabaseStateMachine(RuleBasedStateMachine):
    def __init__(self):
        super().__init__()
        self.model = {}
        self.db = Database()

    @rule(key=st.text(), value=st.integers())
    def insert(self, key, value):
        self.model[key] = value
        self.db.insert(key, value)

    @invariant()
    def model_matches_db(self):
        for key, value in self.model.items():
            assert self.db.get(key) == value
```

---

## Quick Reference

### Commands

```bash
# Start a session
/prime [task_focus]

# Find next work
/next-bead [focus_area]

# Ground your work
/ground [question-or-task]

# Decompose a phase
/decompose-task [phase]
```

### Bead CLI

```bash
# See what's ready
bd ready --json

# Claim a task
bd update <id> --status in_progress --assignee YOUR_NAME

# Close a task
bd close <id> --reason "Completed: [summary]"

# View task
bd show <id>
```

### Sizing Guidelines

| Metric | Target |
|--------|--------|
| Phase size | 500-1000 lines |
| Sub-bead size | ~500 lines |
| Work duration | 30-120 minutes per sub-bead |
| Tests per bead | 4+ categories |

---

## Anti-Patterns to Avoid

### Planning Anti-Patterns

- **Skipping phases** — Don't give agents 5000-line docs
- **Vague plans** — "Implement auth" is not a plan
- **Missing rationale** — Every decision needs a WHY
- **No error handling** — What happens when things fail?

### Decomposition Anti-Patterns

- **Summarizing** — "4 tests for validation" instead of actual test code
- **References** — "See parent bead" instead of copying content
- **Huge sub-beads** — 2000+ lines defeats the purpose
- **Missing tests** — Implementation without test beads

### Execution Anti-Patterns

- **Skipping grounding** — Don't trust training data
- **Ignoring audits** — Quality control exists for a reason
- **Parallel file edits** — Use file reservations
- **Silent work** — Communicate via Agent Mail

---

## Further Reading

- [PHILOSOPHY.md](./PHILOSOPHY.md) — The 4-phase framework
- [CODEMAPS_TEMPLATE.md](./CODEMAPS_TEMPLATE.md) — Architecture documentation
- [TUTORIAL.md](./TUTORIAL.md) — Complete workflow walkthrough
- [AGENTS_TEMPLATE.md](./AGENTS_TEMPLATE.md) — Project-specific agent instructions
