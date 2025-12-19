# The Knowledge & Vibes Philosophy

**How to actually code with AI agents effectively.**

Based on research showing 40-60% productivity gains with zero increase in bugs when done right.

---

## The Core Truth

> "AI amplifies what you know. It doesn't replace knowing things."

You still need to understand your systems. The AI is a brutally efficient junior dev that works at 5x speed and never sleeps—but still needs direction, review, and discipline.

---

## The 4-Phase Framework

Every task, every feature, every bug fix follows this loop:

### Phase 1: Requirements Gathering

**Goal**: Understand the problem completely before touching code.

```
1. Feed codemaps to the LLM (architecture, boundaries, patterns)
2. Front-load explicit context (imports, dependencies, related files)
3. Have the LLM restate the problem in its own words
4. Verify understanding before proceeding
```

**Our tools for this**:
- `cm context "task" --json` → Get past learning
- CODEMAPS/ folder → Architecture documentation
- AGENTS.md → Project-specific patterns
- Warp-Grep → "How does X work?"

**Key question**: "Does the AI actually understand what we're building?"

### Phase 2: Solution Planning

**Goal**: Create a detailed plan BEFORE writing any code.

```
1. Use frontier reasoning models with thinking cranked up:
   - Claude Opus 4.5 (ultrathink mode)
   - GPT 5.2 Xtra High (high reasoning effort)
   - Gemini 3 Pro (thinking mode)
2. Multiple iterative sessions: explore → debate → research → specify
3. Go from idea → massive, fully fleshed out plan (3,000-5,000+ lines)
4. Human breaks the plan into phases (logical chunks, 1-2 weeks each)
5. Use /decompose-task to turn each phase into beads + sub-beads
6. Ground the plan with real documentation (Exa MCP)
7. Review for architectural fit before implementation
```

**Our tools for this**:
- Frontier models with extended thinking → Heavy reasoning, full plan generation
- `/decompose-task` → Turn phases into atomic beads
- `web_search_exa()` → Current documentation
- `get_code_context_exa()` → Implementation examples
- `bv --robot-plan` → Dependency-respecting execution order (sanity-check deps)
- `bv --robot-suggest` → Catch missing deps / cycles early (planning hygiene)
- `bv --robot-insights` → Deep graph metrics + cycle detection (structural health)

**Key question**: "Does this plan fit our architecture and constraints?"

### Phase 3: Implementation

**Goal**: Build incrementally with continuous verification.

```
1. Write tests FIRST (TDD)
2. Create scaffolding (interfaces, signatures)
3. Implement to make tests pass
4. One piece at a time, incremental commits
```

**Our tools for this**:
- `bd create` → Track each piece as a bead (~500 lines)
- `bd dep add` → Explicit dependencies
- Tests → Reward function for AI
- File reservations → Prevent conflicts

**Key question**: "Can this be tested independently?"

### Phase 4: Reflection

**Goal**: Verify before shipping.

```
1. Run all tests, fix failures
2. Scan for bugs (static analysis)
3. Check for edge cases
4. Read every line yourself
5. Lint and format
```

**Our tools for this**:
- `ubs --staged --fail-on-warning` → Bug scanning
- `cass search` → Did we solve this before?
- Manual review → You still read the code
- `bd close --reason` → Document what was done

**Key question**: "Would I bet my job on this code?"

---

## Context Engineering

### The Problem

> "The LLM has none of your context. Zero. It's like an extremely confident junior dev with extreme amnesia."

You know why that weird workaround exists. You know which databases talk to which services. The LLM doesn't—unless you tell it.

### The Solution: Codemaps

Codemaps are Markdown files that outline your architecture:
- Modules and their responsibilities
- Data flows between services
- Key dependencies and integration points
- Common patterns ("we always do X when Y happens")

Keep them:
- **Concise**: Token-aware navigation aids, not comprehensive docs
- **Current**: Stale codemaps are worse than none
- **Scoped**: One per major area (API, database, auth, etc.)

See [CODEMAPS_TEMPLATE.md](./CODEMAPS_TEMPLATE.md) for structure.

### Grounding Tools

**AI-generated code follows training data, which may be outdated, deprecated, or wrong.**

LLMs are trained on snapshots of documentation, tutorials, and code samples from months or years ago. This means:

- **Deprecated APIs**: The AI might use `library.oldMethod()` when `library.newMethod()` replaced it 6 months ago
- **Outdated patterns**: The "recommended approach" from 2023 might be an anti-pattern in 2024
- **Hallucinated methods**: For niche libraries, the AI may confidently invent methods that never existed
- **Wrong defaults**: Configuration options, flags, and parameters change across versions

**This is critical for non-technical users** who can't spot when AI is hallucinating outdated patterns. If you can't read the code and know "that method was deprecated in v3.0," grounding is your safety net.

**The solution: ground everything in current documentation.**

```
# Check current documentation
web_search_exa("library-name API 2024")

# Find real implementation examples
get_code_context_exa("pattern implementation")

# Pull specific page content
crawling("https://docs.example.com/api")
```

Use the `/ground` command to choose the right grounding tool:
- **Repo truth** → Warp-Grep (local codebase discovery)
- **Web truth** → Exa (current external docs/APIs)
- **History truth** → CASS / cass-memory (what we did before)

If you're getting outdated API suggestions, you're not using grounding tools.

---

## Test-Driven Development (TDD)

### Why TDD is Mandatory

> "Tests fill the context gap. They act as a reward function for the AI."

You can't give the LLM full context—ever. Tests catch when it breaks something it didn't know about.

### The Process

```
1. Write test first (what should happen?)
2. Let AI write code to pass test
3. Run test → failure → iterate
4. Run test → success → next piece
5. Repeat
```

### Coverage Target

Aim for **80%+ coverage**: unit, integration, E2E.

"Sounds insane, but AI is really effective at writing tests now."

### Tests as Specification

Write detailed user journeys first:
- "As an admin, I want to view all users enrolled in all courses"
- "As a user, I want to sign up and see my dashboard"

Then generate test cases for each journey. The tests become your specification.

---

## Reading Code > Writing Code

> "In a world where LLMs can blast out code all day, your real leverage isn't how fast you can type. It's how quickly you can understand what the machine just wrote."

### Make AI Code Readable

Feed your code style guidelines to the model:
- Naming conventions
- Structure patterns
- Comment requirements
- Error handling standards

Every AI-generated file should look like a human on your team wrote it.

### Read Every Line

When people stop reviewing:
- Refactors slip through without understanding
- Context on the codebase is lost
- Prompts become increasingly generic
- Six months later, dependency on tools you can't supervise

Reading every line:
- Catches logic bugs tests miss
- Keeps your skills sharp
- Deepens your architectural understanding

---

## Scaffolding Before Implementation

### The Problem

> "Models jump straight to 'final code,' which looks right but is structurally wrong."

Research shows models skip scaffolding 30-55% of the time.

### The Solution

Force scaffolding:

```
1. Write interfaces first
2. Write function signatures
3. Write type definitions
4. THEN implement the bodies
```

This produces better results than "generate the whole thing."

### In Practice

```typescript
// Step 1: Interface
interface PaymentService {
  processPayment(amount: number, method: PaymentMethod): Promise<PaymentResult>;
  refund(transactionId: string): Promise<RefundResult>;
}

// Step 2: Signatures with types
async function processPayment(
  amount: number,
  method: PaymentMethod
): Promise<PaymentResult> {
  // TODO: implement
}

// Step 3: Tests for each method
describe('processPayment', () => {
  it('should process valid payment', async () => { /* ... */ });
  it('should reject invalid amount', async () => { /* ... */ });
});

// Step 4: Implementation to pass tests
```

---

## Capturing External Reasoning

### The Problem

Sometimes you need heavy reasoning from frontier models (Claude Opus 4.5 ultrathink, GPT 5.2 Xtra High, Gemini 3 Pro) before implementation. That reasoning lives in a chat interface, not your codebase.

### The Solution: csctf

Use `csctf` to capture conversations from ChatGPT, Gemini, Grok:

```bash
# Capture a planning conversation from ChatGPT
csctf https://chatgpt.com/share/abc123...

# Output: clean markdown file
# Now feed it to your local agent for implementation
```

### The Workflow

```
1. Use frontier model for heavy reasoning/planning
2. Share the conversation (get share link)
3. Run csctf to capture as markdown
4. Feed markdown to local agent
5. Agent converts plan to beads
6. Implement with full context
```

This lets you leverage expensive reasoning models for planning, then execute with faster/cheaper models.

---

## The Skill Gap

> "The developers who are crushing it with AI aren't the ones who just learned to code last year. It's the senior people."

Why? Senior developers have spent years learning:
- How to break down complex problems
- How to explain intent clearly
- How to think strategically about architecture
- How to spot when something "technically works but architecturally makes no sense"

These skills translate directly to working with AI.

### For Juniors

The path forward is the same as it's always been:
- Get good at understanding systems
- Learn to think architecturally
- Build deep context
- These skills compound, with or without AI

### For Everyone

> "You can't use an LLM to act on what you don't understand."

Your agency is bounded by your own knowledge. The AI amplifies what you know—it doesn't replace knowing things.

---

## Infrastructure for Speed

### The Meta-Point

You're building infrastructure for AI-assisted development:
- Skills that encode your team's patterns
- Slash commands for common workflows
- Hooks that catch mistakes automatically
- Codemaps that provide context

Initial setup takes time. But once it's there:
- Prompts get shorter (context is baked in)
- Workflows get faster (automation)
- Output gets cleaner (hooks catch mistakes)

### Our Infrastructure

| Tool | Purpose |
|------|---------|
| Beads | Task tracking with dependencies |
| Beads Viewer | Graph intelligence for what-to-do-next |
| CASS | Search past solutions |
| cass-memory | Distilled learning from sessions |
| UBS | Automated bug detection |
| Agent Mail | Multi-agent coordination |
| Warp-Grep | Fast codebase exploration |
| Exa | Current documentation |
| csctf | Capture external reasoning |

---

## The Framework Summary

```
REQUIREMENTS → PLAN → IMPLEMENT → REFLECT
     ↓            ↓         ↓           ↓
  Codemaps    Frontier   TDD/Beads     UBS
  cm context   Models    Scaffolding   Tests
  Warp-Grep  /decompose  Incremental   Review
              -task
```

Each tool maps to a phase. Use them together, not in isolation.

---

## Final Thought

> "The tools will change. The model names will change. The fundamentals won't: clear thinking, solid systems, disciplined workflows, and the willingness to adapt."

You can fight it, or you can get very, very good at it.

---

## Further Reading

- [PLANNING_AND_DECOMPOSITION.md](./PLANNING_AND_DECOMPOSITION.md) - Planning and breaking work into beads
- [CODEMAPS_TEMPLATE.md](./CODEMAPS_TEMPLATE.md) - Architecture documentation
- [SETUP_GUIDE.md](./SETUP_GUIDE.md) - Getting started
- [TUTORIAL.md](./TUTORIAL.md) - Complete workflow
