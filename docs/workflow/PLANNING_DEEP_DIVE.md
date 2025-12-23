---
title: Planning Deep Dive
description: From vague idea to complete master plan that AI agents can execute without guessing.
category: workflow
---

<div align="center">

# Planning Deep Dive

### From vague idea to complete master plan

</div>

How to take a vague idea and turn it into a complete master plan that AI agents can execute without guessing.

**Who this is for:** Non-technical builders, technical users who want structure, anyone who's had AI "build the wrong thing."

**What comes next:** Once you have a master plan, see [Decomposition](./DECOMPOSITION.md) to break it into executable tasks.

---

## Table of Contents

- [Why Planning Matters More With AI](#why-planning-matters-more-with-ai)
- [The Goal: Nothing Left to Interpret](#the-goal-nothing-left-to-interpret)
- [Prerequisites](#prerequisites)
- [Design Principles](#design-principles)
- [What "Done Planning" Looks Like](#what-done-planning-looks-like)
- [The Core Idea: Thread Protocol](#the-core-idea-a-thread-protocol-not-one-big-prompt)
- [How This Maps to Tier-1 Artifacts](#how-this-maps-to-tier1-planning-artifacts)
- [Required Inputs](#required-inputs-your-truth-pack)
- [Plan Pack Layout](#plan-pack-recommended-file-layout)
- [Steering a Model Toward Truth](#the-soft-skills-steering-a-model-toward-truth-when-youre-nontechnical)
  - [Curiosity-Driven Discovery](#curiosity-driven-discovery-the-engine)
- [Why Multiple Sessions](#why-this-takes-many-sessions-not-one)
- [Why Multiple Models](#why-use-multiple-frontier-models-different-providers)
- [Test-Based Disagreement Resolution](#multimodel-disagreement-resolution-tests-adjudicate-not-rhetoric)
- [Step 0: North Star](#step-0--north-star-ground-truth)
- [Step 1: Requirements](#step-1--requirements-outcomes-not-solutions)
- [Step 2: Run Threads](#step-2--run-threads-repeat-until-no-guessing)
- [Step 3: Compile Master Plan](#step-3--compile-the-master-plan-lossless)
- [Step 4: Audit](#step-4--audit-adversarial-pass)
- [Execution Handoff](#execution-handoff-after-planning)
- [Selected Research](#selected-research-optional-reading)

---

## Why Planning Matters More With AI

Without explicit planning, AI-assisted development fails in predictable ways:

| What Happens | Why |
|--------------|-----|
| AI builds the wrong thing | Goals weren't explicit enough |
| AI makes different choices | Decisions weren't recorded |
| AI drops requirements | Context was too long |
| AI introduces bugs | Requirements weren't testable |

This guide exists because **the quality of your plan determines the quality of the output**. A vague plan produces vague code. An explicit plan produces code you can verify.

> **Reality check:** This workflow maximizes your odds, but does not guarantee success. Vigilant verification is not optional—the best outcomes come from combining AI capability with human oversight.

---

## The Goal: Nothing Left to Interpret

**Plan as much as appropriate. Give the AI as few decisions as possible.**

Any decision you don't claim, you implicitly delegate. When you delegate to the model, it fills gaps with training data—not your intent. Those gaps become assumptions. Assumptions become architecture. Bad architecture becomes a rewrite.

**The plan is complete when there's nothing left to interpret.**

Could any capable agent pick up any section and build exactly what you envisioned without asking a single clarifying question? If they'd need to ask "what should happen here?"—the plan isn't done.

This is the test you apply throughout planning. Every thread you explore, every decision you surface—ask yourself: would an agent need to guess here?

> **Same hours. Different outcomes.** The time gets spent either way. You either spend it upfront, thinking through decisions before the model touches them. Or you spend it later, untangling decisions the model made without you.

---

## Prerequisites

Before using this guide:
- **If you're starting from scratch:** Begin with `DISCOVERY.md` — the pre-pipeline phase where you surface every decision hiding in your idea through curiosity-driven interrogation.

Then read:
- `START_HERE.md` — System overview
- `EVIDENCE_BASED_GUIDE.md` — The full 10-stage pipeline
- `PROTOCOLS.md` — Protocol cards

For execution after planning, see:
- `IDEATION_TO_PRODUCTION.md` — Full end-to-end pipeline
- `DECOMPOSITION.md` — Breaking plans into executable phases

---

## Design Principles

This guide is designed to be *hard to mess up* by:
- Pinning a **North Star** (what we're building and why)
- Converting intent into **testable requirements** (what must be true)
- Locking key choices as **explicit decisions** (ADRs)
- Producing **lossless specs + tests** (not vibes)
- Running an **audit pass** before execution (catch gaps early)

Every artifact is "agent-friendly": small, explicit, and verifiable.

---

## What "Done Planning" Looks Like

At the end, you have a **Plan Pack** that contains only what's needed for the project's complexity:

**Always required (minimal viable plan):**
- **North Star Card** (goal, stakes, rigor, stop/ask)
- **Requirements** (`REQ-*`) + **Acceptance Criteria** (`AC-*`) with tests
- **Decision log** (ADRs with alternatives + rationale)

**Include when complexity demands (adaptive):**
- **System map** (components + data flows + boundaries) — when >2 components
- **Data model** (types + validation + constraints + examples) — when persistence exists
- **API contracts + error model** — when external interfaces exist
- **Failure modes + recovery** — for P0 requirements
- **Ops + security** — for production deployments
- **Dependencies + phases** — when parallel execution is needed

> **Key principle (Anthropic Context Engineering, 2025):** "The smallest set of high-signal tokens that enables correct behavior." Plan size should match project complexity. A 200-line script doesn't need a 50-page plan.

If a section is missing and you'd have to guess during implementation, it's not done yet.

---

## The Core Idea: A Thread Protocol (Not “One Big Prompt”)

You don’t “prompt” your way into a great plan. You **work threads**:

> Pick a decision area → dive until it’s clear → exit with a decision (or a user question) → capture it in the Plan Pack.

This mirrors how strong teams build software: clarify intent, make tradeoffs explicit, write decisions down, and treat tests/contracts as the spec.

Evidence‑based framing: this is “deliberate search + iterative refinement”.
Instead of trusting a single completion, you explore multiple candidate approaches and then converge using explicit criteria and verification signals. In code-focused work, this shows up as: searching over diverse plans/strategies (`research/019-plansearch.md`, `research/020-codetree.md`), improving selection/reranking (`research/042-rankef.md`), and using structured multi-stage flows with execution feedback (`research/031-alphacodium.md`, `research/043-rlef.md`).

---

## How This Maps to Tier‑1 Planning Artifacts

This process deliberately recreates the outputs you’d expect inside strong engineering orgs, but in a way a non‑technical user can drive with frontier model help.

| Tier‑1 artifact (common name) | What it is | In this system |
|---|---|---|
| Product brief / PRD | user, problem, success metrics, scope | North Star Card + `REQ-*` / `AC-*` |
| Design doc / RFC | architecture, tradeoffs, interfaces | system map + contracts + data model |
| ADR log | durable record of key choices | `PLAN/02_decisions_adrs.md` |
| Test plan | “how we’ll know it works” | `PLAN/08_testing_spec.md` (with code) |
| Threat model / security review | top risks + mitigations | `PLAN/09_ops_security.md` |
| Operational readiness review | logs/metrics/alerts/rollout/rollback | ops/observability + rollout steps |
| Project plan / milestone plan | ordering + dependencies | `PLAN/10_dependencies_and_phases.md` |

The key difference: agents execute the plan, but only after the plan is **lossless** enough to minimize guessing, though execution still requires verification, security gates, and human oversight for critical paths.

> **"Lossless" means no guessing, not "huge."** A lossless plan for a CLI tool might be 50 lines. A lossless plan for a payment system might be 500. The goal is: another agent could implement without asking clarifying questions about intent. If you're adding detail "just in case," you're adding noise, not signal.

---

## Required Inputs (Your “Truth Pack”)

You paste this once at the start of your first session, then keep it updated:

```
PROJECT: <name>
ONE-LINER: <one sentence describing the idea>

WHY / CONTEXT:
- Who is this for?
- Why now?
- What happens if it fails? (annoying vs expensive vs dangerous)
- Build context: (personal / internal company / public SaaS / regulated)

CONSTRAINTS:
- Timeline:
- Budget:
- Data sensitivity:
- Existing systems we must integrate with:

UNKNOWN TECH:
- I am non-technical. When choices exist, present options + tradeoffs and ask me to choose.
```

---

## Plan Pack (Recommended File Layout)

Keep this as one file (`MASTER_PLAN.md`) or a folder. For non‑technical users, multiple small files are easier to edit and reuse.

```
PLAN/
  00_north_star.md
  01_requirements.md
  02_decisions_adrs.md
  03_system_map.md
  04_data_model.md
  05_api_and_error_model.md
  06_risks_and_spikes.md
  07_failure_modes.md
  08_testing_spec.md
  09_ops_security.md
  10_dependencies_and_phases.md
```

Templates that map to the first planning artifacts:
- `templates/NORTH_STAR_CARD_TEMPLATE.md`
- `templates/REQUIREMENTS_TEMPLATE.md`
- `templates/REQUIREMENTS_QA_TEMPLATE.md`
- `templates/DECISIONS_ADRS_TEMPLATE.md`
- `templates/RISKS_AND_SPIKES_TEMPLATE.md`
- `templates/TRACEABILITY_TEMPLATE.md`

---

## The Soft Skills: Steering a Model Toward Truth (When You're Non‑Technical)

If you don't have a technical background, your job is not to "judge the code". Your job is to **steer the conversation so the truth becomes obvious**.

The trick is to stop treating the model like an oracle and start treating it like a **hypothesis generator** that you interrogate.

This is a research‑supported stance: LLMs are strong generators of plausible hypotheses, but their "confidence" is not a reliable truth signal. The planning process works when you repeatedly force hypotheses into contact with **constraints + verification**, not when you accept fluent answers.

### Curiosity-Driven Discovery (The Engine)

Before you can steer toward truth, you need to discover what questions to ask. This is where **relentless curiosity** matters.

Your lever is relentless curiosity. Your tool is a frontier reasoning model.

When a topic comes up, you don't nod and move on. You interrogate:

- "What are my options?"
- "What are the tradeoffs?"
- "What happens if I choose wrong?"
- "What would change your recommendation?"
- "How does this fail? What does the user see?"

Every answer reveals three more questions. Keep pulling until you actually understand—and can explain it back in plain language.

After 15-30 minutes of this on a single topic, you understand things about your own idea that you didn't know you didn't know.

This curiosity is the engine. Without it, you get surface-level plans that crack under pressure. With it, you surface every decision that matters—and you make those decisions informed.

> For the complete pre-pipeline discovery process, see `DISCOVERY.md`.

### The "Truth‑Seeking" Mindset

You are always trying to force three things into the open:
1. **Assumptions** (what the plan silently relies on)
2. **Tradeoffs** (what you gain/lose with each option)
3. **Verification** (how we’ll know it works: tests, contracts, measurable outcomes)

If the model can’t make those explicit, it’s not done thinking yet.

### The Steering Moves (Use These Constantly)

These are simple “moves” that work even when you don’t know the domain:

- **Force clarification before solutions:** “What do you need to know before recommending anything?”
- **Separate facts vs guesses:** “Which claims are facts, and which are assumptions?”
- **Ask for options, not answers:** “Give me A/B/C options and compare them against our North Star.”
- **Make failure concrete:** “How does this fail? What does the user see? What do logs show?”
- **Ask what would change the recommendation:** “What new information would flip your choice?”
- **Demand a falsifiable definition of done:** “Which `AC-*` proves this is correct?”
- **Ask for the simplest thing that meets the bar:** “What is the minimum architecture that satisfies our rigor tier?”
- **Make hidden complexity explicit:** “What will be painful in month 6? What is the maintenance tax?”

Why these moves work: they systematically reduce common LLM failure modes (implicit assumptions, premature convergence, and unverified factual claims) by forcing (a) multiple candidates, (b) explicit decision criteria, and (c) falsifiable checks.

### The Non‑Technical Advantage (How You Still Win)

Even if you can’t evaluate technical details, you can evaluate:
- whether the system is aligned to the **North Star**
- whether requirements are **testable** (you can understand tests as “proof”)
- whether tradeoffs match your **priority order** (e.g., safety > correctness > UX > speed)
- whether unknowns are handled honestly via the **stop/ask rule**

This is enough to drive high‑quality outcomes.

---

### Grounding: How You Avoid Being Misled

When the plan touches “external truth” (framework APIs, security best practices, scaling limits), don’t accept confident claims.

Use this rule:
- **If the claim changes architecture or security, it must be grounded** (verified against current docs or the actual codebase).

Evidence‑based framing: grounding reduces confident nonsense (e.g., API hallucination) and improves repo editing when the agent must pull *the right* context before changing code. See: `research/008-api-hallucination.md`, `research/017-context-retrieval-repo-editing.md`, `research/012-autocoderover.md`.

Practical options:
- **Codebase truth:** use Warp‑Grep (how your repo actually works).
- **Web truth:** use Exa MCP (what the current docs actually say).
- **History truth:** use `cm` / `cass` (what you already did successfully before).

If grounding is not possible in the moment, record it as an explicit assumption and apply the stop/ask rule.

---

## Why This Takes Many Sessions (Not One)

One session fails because:
- you don’t know what you don’t know (unknown unknowns only appear after you see an initial draft)
- the first plausible answer creates **anchoring** (you stop exploring too early)
- real plans require **progressive elaboration** (each thread reveals new constraints)
- the plan must be **captured into artifacts** (REQ/AC/ADR/contracts/tests), not left as chat text
- good planning includes **grounding** (checking real docs and real APIs), which is naturally iterative

You’re not trying to “get an answer”. You’re trying to **reduce uncertainty** until agents can build without guessing.

Evidence‑based framing: iterative, execution‑grounded repair loops outperform “one big answer,” especially when each iteration adds new evidence (tests, traces, repros) instead of rhetorical self‑correction. See: `research/022-chatrepair.md`, `research/045-repairagent.md`, `research/043-rlef.md`, `research/014-codeplan.md`.

---

## Why Use Multiple Frontier Models (Different Providers)

Using multiple strong models is an **ensemble strategy**:
- they have different blind spots and biases
- they notice different failure modes and edge cases
- disagreement is a useful signal: it highlights **where the truth is uncertain** or requirements are underspecified

You don’t do this for “more words”. You do it to make hidden tradeoffs visible and to prevent a single model’s confident mistake from becoming your architecture.

Evidence‑based framing: this is an ensemble/search strategy. Generate diverse candidates, then select using strong discriminators (tests, execution feedback, grounded constraints) rather than confidence. See: `research/019-plansearch.md`, `research/020-codetree.md`, `research/042-rankef.md`, `research/041-debatecoder.md`.

---

### How to Interpret Disagreement

If models disagree, assume one (or more) of these is true:
- your requirements are underspecified (the plan depends on user intent)
- there are real tradeoffs and no single “best” answer
- someone made a hidden assumption that needs to be written down

Your response is not “pick a winner”. Your response is:
1) extract the assumptions and tradeoffs
2) ground any factual claims
3) decide (or stop/ask) and capture the decision as an ADR

---

## Multi‑Model Disagreement Resolution (Tests Adjudicate, Not Rhetoric)

> **Key insight (DebateCoder, 2025):** "Tests are the medium of disagreement, not rhetoric." Rhetorical debate produces persuasive noise. Test-based adjudication produces verifiable truth.

Research shows that unstructured debate **degrades** outcomes (`research/003-debate-or-vote.md`):
- Voting alone beats extended debate
- More debate rounds made things worse
- Agents compromise to reach agreement, losing correctness

**Replace cross-examination with test-based disagreement:**

### The Pattern (Orchestrator-Worker, Not Free-Form Debate)

```
1. BROADCAST: Same question to all agents (North Star + REQ/AC + decision)
2. COLLECT: Independent proposals with tests (not just opinions)
3. TEST-ADJUDICATE: Run competing tests against proposals
4. SELECT: Evidence decides winner, not persuasion
5. PRESERVE DISSENT: If tests don't resolve, document both positions for user
```

### Test-Based Disagreement Prompt

Use one Agent Mail thread per topic (e.g., `thread_id="plan-auth"`):

```markdown
SUBJECT: [PLAN THREAD] <topic>

North Star Card:
<paste>

REQ/AC:
<paste>

Question:
We need to decide <decision>. Provide A/B/C options.

Output format:
1) Options table (pros/cons/risks)
2) **Tests that discriminate** — write tests that PASS for your preferred option and FAIL for alternatives
3) Failure modes + mitigations
4) Grounded evidence (doc links, code references)
5) Recommendation OR stop/ask (what must user decide)
```

### Adversarial Test Generation (Instead of Cross-Examination)

When proposals conflict:

```markdown
Two proposals exist:
- Proposal A: <summary>
- Proposal B: <summary>

Task (DO NOT argue rhetorically):
1) Write tests that would BREAK Proposal A if it has the flaw you suspect
2) Write tests that would BREAK Proposal B if it has the flaw you suspect
3) Run both test sets against both proposals
4) Report results: which tests pass/fail for which proposal?
5) If tests don't discriminate: what measurement/experiment would?
```

### Why Tests Beat Rhetoric

| Rhetorical Debate | Test-Based Adjudication |
|-------------------|-------------------------|
| Persuasive agent wins | Correct implementation wins |
| Agents compromise | Evidence decides |
| Confidence signals quality | Test results signal quality |
| Debates can run forever | Tests terminate |
| Produces "agreement" | Produces "verification" |

Evidence: `research/003-debate-or-vote.md` (voting beats debate), `research/041-debatecoder.md` (tests as medium of disagreement), `research/056-multi-agent-orchestrator.md` (orchestrator-worker beats free-form by 90.2%).

---

## Step 0 — North Star (Ground Truth)

Start by writing the North Star Card using `templates/NORTH_STAR_CARD_TEMPLATE.md`.

**Gate:** The card has explicit:
- stakes/context (why build quality must be high or can be scrappy)
- rigor tier + rationale
- non‑goals + invariants
- stop/ask rule (when planning must pause for user input)

---

## Step 1 — Requirements (Outcomes, Not Solutions)

Convert the North Star into:
- `REQ-*`: observable outcomes / constraints
- `AC-*`: how we verify each requirement is true

If you can’t test or falsify it, it isn’t a requirement yet.

---

## Step 2 — Run Threads (Repeat Until "No Guessing")

Planning quality comes from **how you run threads**.

### The Thread Protocol (Do This Every Time)

For any thread (auth, data, APIs, scaling, etc.), run this loop:

1. **Clarify**: what decision are we making, and what’s unknown?
2. **Interrogate**: ask questions until assumptions are explicit.
3. **Generate options**: A/B/C with tradeoffs and failure modes.
4. **Decide or stop/ask**: choose, or ask the user (no silent guessing).
5. **Capture**: update the Plan Pack (ADR + specs + REQ/AC updates).
6. **Propagate**: list what else must change downstream (tests/contracts/ops).

### Thread Prompt Template (Copy/Paste)

Use this for any topic:

```
We are planning: <PROJECT>.

North Star Card:
<paste>

Current requirements (REQ/AC):
<paste>

THREAD TOPIC: <e.g., authentication, data model, API design, scaling>

Task:
1) Ask clarifying questions until assumptions are explicit.
2) Propose 2-3 viable options (A/B/C) and compare them using our North Star:
   - correctness, UX, speed, cost, security, maintenance (use our priority order)
3) For each option: risks, failure modes, and what tests would validate it.
4) Recommend one option *or* apply the stop/ask rule and tell me exactly what I must decide.
5) Output updates to these artifacts:
   - ADR entry (decision/choice/alternatives/rationale/follow-ups)
   - Any new/changed REQ-* or AC-* (keep outcomes, not solutions)
   - Any spec additions (contracts/data/error model) needed so agents won’t guess
```

### Mandatory Thread Coverage (Production‑Grade)

You should expect to run multiple sessions per thread.

**Product & scope**
- user journeys (happy + unhappy)
- non‑goals, scope boundaries, MVP vs later

**System design**
- component boundaries + integration map
- data model + migrations + retention
- API contracts + error model + idempotency

**Risk & quality**
- authn/authz + permissions model
- failure modes + recovery strategies
- performance/scaling assumptions (what breaks first)
- observability (logs/metrics/alerts) + operational readiness
- security/privacy (threats + mitigations)

**Reward function (tests)**
- testing spec with executable code tied to `AC-*`

---

## Step 3 — Compile the Master Plan (Lossless)

After threads converge, compile everything into `PLAN/10_dependencies_and_phases.md` (or `MASTER_PLAN.md`).

**Gate:** The plan passes the “Any agent can implement” test:
- no “TBD” in core flows
- contracts/types are explicit
- failure modes have explicit handling
- tests are concrete (code), not vague

---

## Step 4 — Audit (Adversarial Pass)

Run an audit session whose only job is to find gaps.

Prompt:

```
Audit this Plan Pack against `DECOMPOSITION.md`.

Output:
- PASS/FAIL
- missing sections and why they matter
- ambiguous requirements (not falsifiable)
- decisions that will be re-litigated unless captured
- concrete additions needed (exact text to add)
```

Fix until PASS.

---

## Execution Handoff (After Planning)

Once the plan is complete:
- break into phases (small, lossless)
- `/decompose-task` each phase into beads/sub‑beads
- execute in dependency order with Agent Mail coordination
- run `/calibrate` **after a phase is implemented** (between phases) to prevent drift

Planning reduces execution risk significantly. But even with a complete plan, execution requires:
- **TDD-first** (tests before implementation, 45.97% improvement per `research/054-tdd-ai-code-gen.md`)
- **Security gates** (`ubs --staged` mandatory, ~40% of LLM code has vulnerabilities)
- **Iteration caps** (max 3 repair attempts per bead, security degrades with more)
- **Human verification** (operator review for P0 requirements, AI helps most when you understand the domain)

---

## Selected Research (Optional Reading)

See `research/README.md` for the full index. If you want a fast, code-focused starting set, read:
- search/selection: `research/019-plansearch.md`, `research/020-codetree.md`, `research/042-rankef.md`
- structured workflows: `research/031-alphacodium.md`, `research/040-mapcoder.md`
- debate grounded in tests: `research/003-debate-or-vote.md`, `research/041-debatecoder.md`
- execution/repair loops: `research/022-chatrepair.md`, `research/045-repairagent.md`, `research/046-d4c.md`, `research/043-rlef.md`
- benchmark "reality checks": `research/021-swe-bench-plus.md`, `research/047-humaneval-pro.md`
- security analysis: `research/044-iris.md`

### Critical Reading

These papers shaped our verification approach:
- **LLM Security Vulnerabilities** (`research/052-llm-security-vulnerabilities.md`): Security scanning is mandatory
- **TDD for AI** (`research/054-tdd-ai-code-gen.md`): TDD yields 45.97% pass@1 improvement
- **Multi-Agent Patterns** (`research/056-multi-agent-orchestrator.md`): Orchestrator-worker outperforms single-agent by 90.2%
- **Debugging Decay** (`research/060-debugging-decay-index.md`): 3-iteration cap prevents degradation

---

## Key Refrains

**Plan as much as appropriate. Give the AI as few decisions as possible.**

**The plan is complete when there's nothing left to interpret.**

**Same hours. Different outcomes.**

**Gaps become assumptions. Assumptions become architecture. Bad architecture becomes a rewrite.**

---

## Related Documents

- `DISCOVERY.md` — Pre-pipeline curiosity-driven architecture (start here for new projects)
- `DECOMPOSITION.md` — Breaking plans into executable phases (Bounded. Complete. Verified.)
- `IDEATION_TO_PRODUCTION.md` — Full end-to-end pipeline
- `PHILOSOPHY.md` — Core principles and research backing
