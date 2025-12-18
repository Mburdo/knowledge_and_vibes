# Planning and Decomposition

**How to turn an idea into executable beads that agents can implement flawlessly.**

---

## The Core Problem

You have a big goal: "Build a payment system" or "Add real-time collaboration" or "Migrate to new architecture."

Dumping this into one task creates unclear progress, no parallelization, overwhelming scope, and merge conflicts.

**But there's a deeper problem**: Agents perform poorly on large documents.

---

## Why Agents Need Small Documents

**This is not about your convenience. It's about agent performance.**

Agents work best on files under **1000 lines**, preferably **500 lines**. Give an agent a 5000-line planning document and it **will** turn that into subtasks horribly.

What happens with large documents:

| Problem | What You'll See |
|---------|-----------------|
| **Context bombing** | Agent gets overwhelmed, starts skipping detail |
| **Lazy summarization** | Vague outputs like "implement authentication" instead of specific methods |
| **Content loss** | Your edge cases and method signatures get paraphrased away |

**The solution**: Break your plan into phases BEFORE giving it to an agent. Each phase fits in working memory, the agent preserves ALL detail, and you get LOSSLESS decomposition instead of lossy summarization.

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
- **Claude Opus 4.5** (ultrathink mode)
- **GPT 5.2 Xtra High** (high reasoning effort)
- **Gemini 3 Pro** (thinking mode)

### The Goal

Transform a vague idea into a comprehensive plan with architecture decisions, data structures, API contracts, error handling, testing strategy, and integration points.

### The Iterative Ideation Process

**This is NOT a single prompt.** Good plans emerge from multiple sessions of reasoning, debate, research, and refinement.

#### Session 1: Initial Exploration

Start with high-level questions. Let the model reason through the problem space.

> *"I want to build [FEATURE] for [PROJECT]. Here's my existing architecture. What are the major components we'd need? What are the key architectural decisions? What are the risks and tradeoffs?"*

Don't accept the first answer. Ask follow-ups. Challenge assumptions. Push for alternatives.

#### Session 2: Decision Debates

For each major architectural decision, have the model argue both sides:

> *"You suggested JWT for auth. Let's debate this. Make the strongest case FOR JWTs. Now make the strongest case AGAINST. What would we use instead? Given our constraints, which is actually better and why?"*

This surfaces edge cases you'd miss with a single-pass answer.

#### Session 3: Research and Grounding

Have the model research current best practices. Don't trust training data:

> *"Search for current best practices on [decision]. What do the 2024/2025 docs say? Are there deprecations or breaking changes? What's the community consensus?"*

Ground every external dependency in current reality.

#### Session 4: Specification Deep Dives

Once decisions are made, get extremely specific:

> *"Spec out the data structures for [component]. For each model: every field with exact types, validation rules, relationships, indexes, constraints. Show valid and invalid examples."*

#### Session 5: Failure Mode Analysis

> *"What are ALL the ways [component] could fail? For each: what causes it, how do we detect it, how do we handle it, what test would catch it? Write actual test code, not descriptions."*

#### Session 6: Integration and Anti-Patterns

> *"How does [component] connect to our existing systems? Draw the data flow. What mistakes would a developer likely make? What should we explicitly NOT do, and why?"*

### Why Iteration Beats Single Prompts

| Single Prompt | Iterative Sessions |
|---------------|-------------------|
| Model guesses at decisions | Decisions debated from multiple angles |
| Generic best practices | Researched against current documentation |
| Surface-level coverage | Deep exploration of edge cases |
| Assumptions unchallenged | Every assumption questioned |

### What the Massive Plan MUST Contain

| Section | What It Contains | Why It Matters |
|---------|-----------------|----------------|
| **Architecture Decisions** | ADR tables: Decision, Choice, Alternatives, Rationale | Agents need WHY, not just WHAT |
| **Data Structures** | Full types, validation rules, field specs | No guessing about schema |
| **API Contracts** | Request/response shapes, status codes, errors | Exact contracts, not descriptions |
| **Error Handling** | Every failure mode, exceptions, recovery | Agents won't invent this |
| **Full Code Examples** | Actual implementation with imports | Copy-paste ready |
| **Testing Strategy** | Unit, integration, e2e, property-based | Actual test code |
| **Anti-Patterns** | What NOT to do with explanations | Prevents common mistakes |
| **Dependencies** | What must exist before, what this enables | Ordering requirements |
| **Acceptance Criteria** | How to verify it's done | Definition of done |

### The "Any Agent Can Implement" Test

Your plan passes if any agent can pick up any section and implement it perfectly without needing external docs. If an agent would need to ask "but how exactly should I...?" then the plan lacks detail.

### Example: Inadequate vs Adequate

**INADEQUATE:**
> "Add user authentication using JWT tokens. The system should validate users and issue tokens. Include appropriate tests."

This is worthless. What token format? What validation rules? What error codes? What tests exactly?

**ADEQUATE:**

The adequate version includes:
- ADR table with token format (JWT HS256), storage (HTTP-only cookie), lifetime (15 min access, 7 day refresh)
- Full Pydantic models for `TokenPayload` and `AuthResponse` with every field typed
- Error table: INVALID_CREDENTIALS (401), TOKEN_EXPIRED (401), TOKEN_INVALID (401) with exact JSON responses
- Complete test classes: `TestAuthHappyPath`, `TestAuthEdgeCases` with actual pytest code
- Anti-patterns: don't store in localStorage (XSS), don't use long-lived access tokens, don't put sensitive data in JWT payload

---

## Phase 2: Collaborative Phase Breakdown

**This step requires YOUR active participation.** The agent can help, but you MUST stay involved.

The massive plan (3000-5000+ lines) needs to be chunked before agents can work with it. You have two options:

1. **Do it yourself** — Read, identify boundaries, create phases manually
2. **Work with an agent** — Have the agent help, but with extreme care

### Why There's No `/break-into-phases` Command

This step requires LOSSLESS thinking at high stakes. If you hand an agent a 5000-line plan and say "break this into phases":

- Critical details will be lost (agent summarizes instead of preserves)
- Dependencies will be missed (subtle ordering overlooked)
- Scope will be wrong (some phases too big, others too small)
- You'll throw away your work (carefully created plan gets mangled)

The agent can help, but it needs to work **WITH** you, not **FOR** you.

### The Collaborative Process

**Step 1: Prime the agent**

Give the agent the full plan, your architecture docs, and constraints. Ask it to identify logical breakpoints WITHOUT creating the phases yet.

**Step 2: Review boundaries TOGETHER**

For each proposed breakpoint, validate:
- Does this boundary make sense architecturally?
- What must exist before this phase can start?
- Can this phase be tested independently?
- Is the size right (~500-1000 lines)?

Push back. The agent will miss things.

**Step 3: Iteratively refine**

Common corrections: "Phase 3 is too big—split it." "These phases are too coupled—merge or reorder." "You missed the error handling section." "Phase 2 depends on Phase 4—fix ordering."

**Step 4: Verify NOTHING was lost**

Before finalizing, have the agent map every section from the original plan to a phase. Flag anything not assigned.

**Step 5: Create phase documents**

Only after validation, have the agent create the actual phase documents, copying content VERBATIM.

### What Agents WILL Miss (Without Your Help)

| Commonly Missed | Why |
|----------------|-----|
| Cross-cutting concerns | Error handling, logging scattered across sections |
| Implicit dependencies | "This assumes X exists" buried in prose |
| Edge cases | Validation rules in detailed specs |
| Integration points | Mentioned once then assumed |
| Anti-patterns | "Don't do X" warnings summarized away |
| Rationale | The WHY lost to "implement X" |

Your job is to catch these. If you catch one thing missed, ask about everything related—it likely missed more.

---

## What Phase Documents MUST Contain

Each phase document is a self-contained specification with these 10 mandatory sections:

| # | Section | Content |
|---|---------|---------|
| 1 | **Context & Motivation** | WHY this exists, risks, user value |
| 2 | **Architecture Decisions** | Table: Choice, Alternatives, Rationale |
| 3 | **System Integration Map** | ASCII diagram of component connections |
| 4 | **Prerequisites** | What must exist before starting |
| 5 | **Detailed Specification** | Inputs, outputs, data structures, validation, errors |
| 6 | **Implementation Guidance** | File locations, code with imports, anti-patterns |
| 7 | **Testing Specification** | All 4 test types with FULL code |
| 8 | **Verification Checklist** | Functional, quality, security checks |
| 9 | **Cross-References** | Depends on, blocks, related beads |
| 10 | **Acceptance Criteria** | Explicit "done when" conditions |

### Testing Specification Requirements

This is where most plans fail. Include:

- **Unit Tests** (≥3 classes): `TestHappyPath`, `TestEdgeCases`, `TestErrorHandling`
- **Integration Tests**: Full API cycles, database transactions
- **E2E Tests**: Playwright/Cypress code, user journeys
- **Property-Based Tests**: Hypothesis strategies, invariant verification

Each type must include **actual executable code**, not "4 tests for validation."

### Size Verification

| Metric | Target | Action if Wrong |
|--------|--------|-----------------|
| Total lines | 500-1000 | Split if large, merge if small |
| Code blocks | ≥5 | Add implementation details |
| Test code | ≥100 lines | Add actual tests |
| Tables | ≥3 | Add decisions/errors/specs |

---

## Phase 3: Task Decomposition

Feed ONE phase to an agent using `/decompose-task`.

### What the Agent Creates

1. **Parent bead** — The phase as an epic-level task
2. **Sub-beads** — Atomic tasks that sum to the parent

### Sub-Bead Structure

| Suffix | Content |
|--------|---------|
| `.0` | Context Brief: WHY, architecture decisions, integration map, prerequisites |
| `.1` | Schema/Types: migrations, type definitions, interfaces |
| `.2-.3` | Implementation: core code with full imports |
| `.4` | Tests: Happy Path |
| `.5` | Tests: Edge Cases |
| `.6` | Tests: Error Handling |
| `.7` | Tests: Property-Based (Hypothesis) |
| `.8` | Tests: Integration |
| `.9` | Reference Data: constants, addresses, lookups |
| `.10` | Verification Checklist |

### The LOSSLESS Rule

**CRITICAL: Everything from the phase plan must appear in a sub-bead.**

- **NEVER** paraphrase or summarize
- **NEVER** write "see parent bead for details"
- **NEVER** skip "obvious" content
- **COPY** content verbatim, typos and all

### Verification

After decomposition:
1. **Character count**: Sub-beads total ≥ original (overhead expected)
2. **Content check**: Every section appears somewhere
3. **Standalone test**: Each sub-bead makes sense independently

---

## Quality Control: The Audit Process

*Optional but recommended for complex tasks.*

After decomposition, a DIFFERENT agent audits the work.

### Audit Criteria

1. **LOSSLESS** — No content lost or summarized
2. **VERBATIM** — Content copied exactly
3. **COMPLETE** — All sections appear in sub-beads
4. **STANDALONE** — Each sub-bead makes sense alone
5. **CORRECT STRUCTURE** — Follows standard pattern

### Audit Outcome

- **PASS**: Close decomposition task
- **FAIL**: Create REDO task with specific issues, decomposer fixes, another audit cycle

---

## Grounding: Preventing Hallucination

**AI-generated code follows training data, which may be outdated.**

Before implementing anything touching external libraries, APIs, or frameworks, GROUND your plan in current documentation.

### The Grounding Process

1. **Identify** external dependencies
2. **Query Exa** for current documentation
3. **Verify** your plan matches current best practices
4. **Update** outdated patterns in your beads

### Grounding Status Header

Add this to your bead after verification:

| Pattern | Exa Query | Status |
|---------|-----------|--------|
| `@pytest.mark.asyncio` | "pytest-asyncio 2024 2025" | ✅ Current |
| `@model_validator(mode='after')` | "Pydantic v2 model_validator" | ✅ Current |

---

## Testing Requirements

Every bead needs comprehensive tests.

| Category | Purpose | Example |
|----------|---------|---------|
| **Happy Path** | Correct behavior | Valid input → expected output |
| **Edge Cases** | Boundaries | Empty list, max int, null |
| **Error Handling** | Failures | Invalid input, network error |
| **Property-Based** | Invariants | "For all valid X, output is valid" |
| **Integration** | Cross-component | Full request → response |

### Property-Based Testing with Hypothesis

```python
from hypothesis import given, strategies as st
from hypothesis.stateful import RuleBasedStateMachine, rule, invariant

# Stateless
@given(st.lists(st.integers()))
def test_sort_idempotent(xs):
    assert sorted(sorted(xs)) == sorted(xs)

# Stateful
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

| Command | Purpose |
|---------|---------|
| `/prime [focus]` | Start a session |
| `/next-bead [area]` | Find next work |
| `/ground [task]` | Ground your work |
| `/decompose-task [phase]` | Decompose a phase |

### Bead CLI

| Command | Purpose |
|---------|---------|
| `bd ready --json` | See what's ready |
| `bd update <id> --status in_progress --assignee NAME` | Claim a task |
| `bd close <id> --reason "..."` | Complete a task |
| `bd show <id>` | View task details |

### Sizing Guidelines

| Metric | Target |
|--------|--------|
| Phase size | 500-1000 lines |
| Sub-bead size | ~500 lines |
| Work duration | 30-120 min per sub-bead |
| Tests per bead | 4+ categories |

---

## Anti-Patterns

### Planning
- Skipping phases (don't give agents 5000-line docs)
- Vague plans ("implement auth" is not a plan)
- Missing rationale (every decision needs WHY)
- No error handling

### Decomposition
- Summarizing ("4 tests for validation" → include actual code)
- References ("see parent bead" → copy content)
- Huge sub-beads (2000+ lines defeats purpose)
- Missing tests

### Execution
- Skipping grounding (don't trust training data)
- Ignoring audits
- Parallel file edits without reservations
- Silent work (communicate via Agent Mail)

---

## Further Reading

- [PHILOSOPHY.md](./PHILOSOPHY.md) — The 4-phase framework
- [CODEMAPS_TEMPLATE.md](./CODEMAPS_TEMPLATE.md) — Architecture documentation
- [TUTORIAL.md](./TUTORIAL.md) — Complete workflow walkthrough
- [AGENTS_TEMPLATE.md](./AGENTS_TEMPLATE.md) — Project-specific agent instructions
