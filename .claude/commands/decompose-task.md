# Decompose Task - Turn a Phase Plan into Beads

Break a phase from your master plan into atomic tasks (beads) and subtasks (sub-beads).

## Usage

```
/decompose-task <phase-description-or-id>
```

## Arguments

- `$ARGUMENTS` - The phase to decompose. Can be:
  - A phase ID from your planning doc (e.g., `Phase 2: Authentication`)
  - A description of what the phase covers
  - A bead ID if the phase is already tracked as a bead

---

## The Workflow This Fits Into

This command is part of a larger planning workflow:

```
1. IDEATION
   └─ Use frontier reasoning models with thinking cranked up:
      - Claude Opus 4.5 (ultrathink mode)
      - GPT 5.2 Xtra High (high reasoning effort)
      - Gemini 3 Pro (thinking mode)
   └─ Multiple iterative sessions: explore → debate → research → specify
   └─ Go from idea → massive, fully fleshed out plan (3,000-5,000+ lines)

2. PHASE BREAKDOWN (Human)
   └─ YOU break the massive plan into phases
   └─ Each phase is a logical chunk of work (1-2 weeks typically)

3. TASK DECOMPOSITION ← YOU ARE HERE
   └─ /decompose-task takes ONE phase
   └─ Turns it into beads (tasks) and sub-beads (subtasks)
   └─ Each task is ~500 lines, 30-120 minutes of work

4. EXECUTION
   └─ Agents claim and execute individual beads
   └─ Use /prime, /next-bead for coordination
```

---

## Why We Break Plans Into Phases

**Agents perform poorly on large documents.**

This is not about human convenience — it's about agent performance:

- Agents work best on files under **1000 lines**, preferably closer to **500 lines**
- Give an agent a 5000-line planning document and it **will** turn that into subtasks horribly
- Large documents cause "context bombing" — the agent gets lazy and skips detail
- You'll see vague summaries like "implement authentication" instead of the specific methods, types, and tests from your plan

**This is a content loss problem.**

Your massive plan contains critical detail: specific method signatures, edge cases, integration points. When you hand a 5000-line doc to an agent, that detail gets summarized away. The agent can't hold it all in working memory, so it approximates.

**The solution: make content digestible.**

By breaking your plan into phases BEFORE giving it to an agent, you're setting the agent up for success:

- Each phase is small enough to fit in working memory
- The agent can preserve ALL the detail from your plan
- You get LOSSLESS decomposition instead of lossy summarization

Think of it like chunking for LLMs — you wouldn't stuff 100k tokens into a prompt and expect perfect recall. Same principle applies here.

---

## What This Command Does

Takes a phase description or plan section and:

1. Analyzes the scope and components
2. Creates a parent bead for the phase (if not already a bead)
3. Breaks it into atomic sub-beads following the standard structure
4. Ensures LOSSLESS decomposition (nothing from the plan is lost)

---

## Standard Sub-Bead Structure

When decomposing a phase, use this pattern:

| Suffix | Content | Purpose |
|--------|---------|---------|
| `.0` | **Context Brief** | WHY this phase exists, architecture decisions, system map |
| `.1` | **Schema/Types** | Database migrations, type definitions, interfaces |
| `.2-.3` | **Implementation** | Core code with full imports, every method |
| `.4` | **Tests: Happy Path** | Success scenario tests |
| `.5` | **Tests: Edge Cases** | Boundary conditions, unusual inputs |
| `.6` | **Tests: Error Handling** | Failure modes, exceptions |
| `.7` | **Tests: Property-Based** | Hypothesis/fuzzing tests for invariants |
| `.8` | **Tests: Integration** | Cross-component verification |
| `.9` | **Reference Data** | Constants, addresses, lookup tables |
| `.10` | **Verification Checklist** | Acceptance criteria, completion checks |

Adjust based on what the phase actually contains. Not every phase needs all suffixes.

---

## Instructions

### Step 1: Understand the Phase

Read the phase description carefully. Identify:

- What is the goal of this phase?
- What components/files will be created or modified?
- What are the dependencies (what must exist first)?
- What are the deliverables (how do we know it's done)?

If the phase is vague, **ask for clarification before decomposing**.

### Step 2: Create the Parent Bead (if needed)

If the phase isn't already tracked as a bead:

```bash
bd create "Phase N: <Phase Title>" -t epic -p 1 -d '<Phase description from the plan>'
```

Note the bead ID for creating sub-beads.

### Step 3: Create Content Manifest

Before creating sub-beads, list everything that needs to be captured:

```markdown
## Content Manifest for <Phase>

### Components
- [ ] Component 1: <description>
- [ ] Component 2: <description>

### Files to Create/Modify
- [ ] <path/to/file1.py>: <what changes>
- [ ] <path/to/file2.py>: <what changes>

### Tests Required
- [ ] Happy path: <scenarios>
- [ ] Edge cases: <scenarios>
- [ ] Error handling: <scenarios>

### Dependencies
- [ ] Depends on: <other beads/phases>
- [ ] Enables: <downstream beads/phases>

### Acceptance Criteria
- [ ] <Criterion 1>
- [ ] <Criterion 2>
```

### Step 4: Create Sub-Beads

For each logical chunk, create a sub-bead:

```bash
bd create "<Sub-task title>" --parent <phase-bead-id> --priority 1 -d '<Full description>'
```

**Critical Rules:**

1. **LOSSLESS** - Every detail from the phase plan must appear in a sub-bead
2. **FULL CODE** - Include complete code with all imports, not snippets
3. **FULL TESTS** - Include actual test code, not "4 tests for X"
4. **NO SUMMARIZING** - Copy content verbatim, don't paraphrase
5. **STANDALONE** - Each sub-bead must be executable without referencing others

### Step 5: Verify Completeness

Check that:

- [ ] Every item from the manifest is assigned to a sub-bead
- [ ] No content was summarized or lost
- [ ] Each sub-bead is atomic (~500 lines, 30-120 min)
- [ ] Dependencies between sub-beads are explicit

### Step 6: Set Dependencies

```bash
bd dep add <child-id> <blocker-id> --type blocks
```

Common patterns:
- Schema (.1) blocks implementation (.2, .3)
- Implementation blocks tests (.4-.8)
- All sub-beads block parent phase completion

### Step 7: Validate the Graph (bv)

Once dependencies are set, use `bv` to catch structural planning errors early (cycles, missing deps, impossible ordering).

```bash
# Hygiene: duplicates / missing deps / cycle break suggestions
bv --robot-suggest

# Can work proceed in dependency order? (tracks + what unblocks what)
bv --robot-plan

# Proactive warnings (stale, cascades, drift signals)
bv --robot-alerts
```

If anything looks wrong, fix the dependency structure *before* agents start executing.

---

## Sizing Guidelines

Each sub-bead should be:

- **~500 lines of code** (including tests)
- **30-120 minutes of focused work**
- **Single responsibility** (one thing done well)
- **Independently testable** (can verify without other sub-beads)

If a sub-bead is too large, decompose it further.

---

## Example Decomposition

**Phase:** "User Authentication"

```
user-auth (parent bead - epic)
├── user-auth.0   Context Brief (WHY, ADR, integration map)
├── user-auth.1   Database schema (users table, migrations)
├── user-auth.2   Password hashing service
├── user-auth.3   JWT token service
├── user-auth.4   Registration endpoint
├── user-auth.5   Login endpoint
├── user-auth.6   Auth middleware
├── user-auth.7   Tests: Happy path (registration, login, token refresh)
├── user-auth.8   Tests: Edge cases (invalid email, weak password)
├── user-auth.9   Tests: Error handling (expired tokens, rate limits)
└── user-auth.10  Verification checklist
```

---

## When to STOP and Ask

Stop and ask the user if:

1. **Phase is too vague** - Can't identify concrete deliverables
2. **Phase is too large** - Would result in 20+ sub-beads (suggest splitting into multiple phases)
3. **Unclear dependencies** - Don't know what must exist first
4. **Multiple valid approaches** - Need architectural decision before decomposing

---

## Anti-Patterns

**DON'T:**
- Summarize: "4 tests for validation" → Include full test code
- Reference elsewhere: "See phase plan for details" → Copy content verbatim
- Skip imports: Partial code blocks → Full runnable code
- Create huge beads: 2000+ lines → Split into multiple beads
- Forget tests: Implementation without test beads → Always include .4-.8

**DO:**
- Copy verbatim from the phase plan
- Include full code with all imports
- Create more sub-beads rather than fewer
- Make each sub-bead independently executable
- Verify nothing was lost

---

## Output

After decomposition, output:

```markdown
## Decomposition Complete: <Phase>

**Parent Bead:** <id>
**Sub-Beads Created:** <count>

| ID | Title | Est. Time | Depends On |
|----|-------|-----------|------------|
| <id>.0 | Context Brief | 15 min | - |
| <id>.1 | Schema | 30 min | - |
| <id>.2 | Implementation | 90 min | .1 |
| ... | ... | ... | ... |

**Total Estimated Time:** X hours
**Ready to Start:** <id>.0, <id>.1 (no blockers)

### Next Steps
1. Agent claims parent + all sub-beads together
2. Reserves relevant files
3. Announces [CLAIMED]
4. Executes in dependency order
```
