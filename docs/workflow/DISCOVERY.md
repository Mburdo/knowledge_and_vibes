---
title: Discovery
description: The pre-pipeline phase where you surface every decision hiding in your idea through curiosity-driven interrogation of frontier models.
category: workflow
---

<div align="center">

# Discovery

### From Idea to Architecture

</div>

The phase before planning. Where you surface every decision hiding in your idea through curiosity-driven interrogation of frontier models.

**Who this is for:** Anyone with an idea they want to build—especially non-technical builders who need to specify systems they don't yet fully understand.

**What comes next:** Once discovery is complete, formalize into the pipeline starting with [Ideation to Production](./IDEATION_TO_PRODUCTION.md) Stage 0.

---

## Table of Contents

- [The Core Principle](#the-core-principle)
- [The Compression Problem](#the-compression-problem)
- [The Test](#the-test)
- [Curiosity-Driven Discovery](#curiosity-driven-discovery)
- [The Questions That Matter](#the-questions-that-matter)
- [What You're Producing](#what-youre-producing)
- [For Non-Technical People](#for-non-technical-people)
- [The Three Rules of Lossless Decomposition](#the-three-rules-of-lossless-decomposition)
- [Stress Testing Your Plan](#stress-testing-your-plan)
- [How This Connects to the Pipeline](#how-this-connects-to-the-pipeline)
- [Research Support](#research-support)

---

## The Core Principle

**Plan as much as appropriate. Give the AI as few decisions as possible.**

Any decision you don't claim, you implicitly delegate. When you prompt, you're delegating the choices you didn't lock down and hoping the model makes them correctly. When you plan, you're making those decisions yourself and handing off only the execution.

This is the difference between hoping and knowing.

There are two ways to work with coding agents:

1. **Hope.** You write a prompt, hand it off, and hope the output matches what you had in mind. Sometimes it does. Often it doesn't. And when it doesn't, you iterate—adjusting, re-prompting, trying to steer toward what you wanted.

2. **Control.** You specify exactly what you want, in enough detail that the model has nothing to decide on its own. Then you execute—not hoping for a good outcome, but knowing what the outcome will be.

> **Same hours. Different outcomes.** The time gets spent either way. You either spend it upfront, thinking through decisions before the model touches them. Or you spend it later, untangling decisions the model made without you.

---

## The Compression Problem

You have an idea. And that idea—whether you realize it or not—contains thousands of tiny decisions. The data model. The user flow. What happens when someone clicks that button. What error messages look like. How authentication works.

All of it is in your head. A high-definition vision of what "done" looks like.

Then you open an agent and type: "Build me a dashboard for tracking personal finances."

In that moment, you just took that entire vision—thousands of decisions—and compressed it into twelve words.

**That's the fatal error. And almost everyone makes it.**

The model doesn't have your context. It can't read your mind. So it has to fill the gaps. And it fills them with training data. Not your intent. Training data.

The output might look right. The UI renders. Buttons click. But underneath? A stack of defaults, assumptions, and placeholder logic that have nothing to do with what you actually wanted.

### Why This Destroys Projects

Vibe-coding feels productive because motion feels like progress. You see something on screen. Your brain releases dopamine. You think you're building.

But you're not building. You're *accumulating assumptions*. And those assumptions are about to destroy everything.

Here's what happens:

1. You ask for "user login." The agent makes dozens of decisions you never specified—session length, password rules, error messages, token storage.

2. You add payments. The agent picks an approach based on the login decisions it already made.

3. You add subscriptions. Those inherit the payment decisions. Which inherited the login decisions.

Three layers deep. Assumptions stacked on assumptions stacked on assumptions.

Now—six features later—something breaks. You try to debug it. But the bug isn't in the feature you're looking at. It's three layers down, in a decision you never made, in a session you no longer have.

That's not a bug. That's an architecture problem. And by the time you see it, it's load-bearing.

> **Gaps become assumptions. Assumptions become architecture. Bad architecture becomes a rewrite.**

---

## The Test

**The plan is complete when there's nothing left to interpret.**

Could any capable agent pick up any section of this plan and build exactly what you envisioned without asking a single clarifying question?

If they'd need to ask "what should happen here?" or "which approach do you want?"—the plan isn't done. That's a gap. And gaps become assumptions.

This is the test you apply throughout discovery. Every thread you explore, every decision you surface—ask yourself: would an agent need to guess here?

If yes, keep going.

---

## Curiosity-Driven Discovery

If you build software professionally, you already feel the questions before you write anything. You see a feature and immediately start thinking about the decisions that need to be made. That intuition comes from experience.

If you don't have that background, you can still get there. The lever is **relentless curiosity**.

Your tool is a **frontier reasoning model**. That's where you do discovery.

### The Process

1. **Bring your idea** to a frontier reasoning model (Claude, GPT-4, etc.)
2. **Ask:** "What would it actually take to build this? What decisions are hiding inside it? What am I missing?"
3. **Pull on every thread** until you understand

The model mentions authentication. You don't just nod and move on. You ask: "What are my options? What's the difference between them? What happens if I choose wrong? What do you recommend for my situation, and why?"

Databases come up. You ask: "Which one? What are the tradeoffs? What happens at scale? What happens if the data gets inconsistent?"

Every answer is a new thread. You keep pulling until you actually understand—and can explain it back in plain language.

### Example: Personal Finance Dashboard

> **You:** "I want to build a personal finance dashboard."
>
> **Model:** "What's the data source? Are you connecting to bank accounts?"
>
> **You:** "I don't know. What are my options?"
>
> **Model:** "Three options: Plaid for automatic bank connections, manual entry for users to input transactions, or CSV import for spreadsheet uploads."
>
> **You:** "What are the tradeoffs? What happens if Plaid changes their API? What's the cost structure?"
>
> **Model:** [Detailed comparison of costs, reliability, user experience, maintenance burden]
>
> **You:** "You mentioned a database. Which one? What happens if my data gets inconsistent? What happens at 10,000 users?"
>
> **Model:** [Detailed analysis of database options, consistency guarantees, scaling considerations]

After 15-30 minutes of this on a single topic, you understand things about your own idea that you didn't know you didn't know.

That's curiosity-driven discovery. You're not passively receiving information. You're interrogating the model until there are no gaps left for it to fill with assumptions.

---

## The Questions That Matter

These questions work even when you don't know the domain. Use them constantly:

### Clarification Questions
- "What do you need to know before recommending anything?"
- "What assumptions are you making?"
- "Which parts of what you just said are facts, and which are guesses?"

### Options Questions
- "What are my options here? Give me A/B/C."
- "What are the tradeoffs between these options?"
- "What would change your recommendation?"

### Failure Questions
- "How does this fail? What does the user see?"
- "What happens in month 6? What's the maintenance burden?"
- "What breaks first at scale?"

### Verification Questions
- "How will we know this works?"
- "What test would prove this is correct?"
- "What would prove this approach is wrong?"

### Simplicity Questions
- "What's the minimum that satisfies our requirements?"
- "What complexity can we remove?"
- "What are we over-engineering?"

The model will ask you questions too—about users, constraints, priorities. But you're never passive. You're always asking "why?" and "what could go wrong?"

This curiosity is the engine. Without it, you get surface-level plans that crack under pressure. With it, you surface every decision that matters—and you make those decisions informed.

---

## What You're Producing

All of that exploration, all of those questions, all of that stress-testing—it produces one thing.

**A Master Plan.** A synthesis of everything you discovered across those sessions.

This isn't a traditional requirements doc. It's a full blueprint. On a real project, you're looking at three, four, five thousand lines of planning. That sounds like a lot. It is. That's what thoroughness looks like.

The less technical you are, the more time you have to spend here—rigorously.

### What Goes in the Master Plan

Not just what you want—**why you want it**. Not just how it should work—**what you considered and rejected**. The reasoning behind the choices, so the agent can follow the thinking.

The details get specific. Really specific:

| Vague (Gaps) | Specific (No Gaps) |
|--------------|-------------------|
| "A user profile with relevant information" | User profile contains: display name (string, 50 char max), email (validated format), avatar (URL or null), created_at (timestamp), subscription_tier (enum: free/pro/enterprise) |
| "Handle payments" | Stripe integration: checkout session creation, webhook handling for payment_intent.succeeded, subscription lifecycle (create/update/cancel), idempotency keys on all mutations |
| "Show error messages" | Error display: toast notification (top-right, 5s auto-dismiss), error codes mapped to user-friendly messages, retry button for transient failures, support contact for fatal errors |

This includes:
- Architecture decisions with rationale
- Data structures with types and constraints
- API contracts with request/response shapes
- Error handling for every failure mode
- Test expectations tied to acceptance criteria
- Edge cases explicitly addressed

### Plan Hardest Around Irreversible Decisions

Here's the cascade that kills projects:

> **Gaps become assumptions. Assumptions become architecture. Bad architecture becomes a rewrite.**

Plan hardest around the irreversible stuff:
- Databases and data models
- Authentication systems
- Core domain logic
- External API integrations
- Security boundaries

Get those wrong early, and you're rebuilding later.

---

## For Non-Technical People

You won't become a professional programmer. You won't build these systems from scratch or debate architecture at a deep technical level.

But you'll understand them well enough to:
- Plan accurately
- Ask the right questions
- Make informed decisions

That's enough. That's what closes the gap between your vision and working software.

### The Learning Curve

The first time through, this will feel alien. Terms you don't know. Concepts that don't click yet. That's normal. Embrace it.

This system compounds over time.

Each project you work through, you learn more. The terminology starts to make sense. The patterns become familiar. What felt overwhelming on project one feels manageable by project five.

It's okay to put a complex project on hold and build something simpler first. Reps matter. Experience counts. You don't need to understand everything right away.

### What You Can Evaluate (Even Without Technical Background)

Even if you can't evaluate technical details, you can evaluate:

- Whether the system is aligned to your **goals** (North Star)
- Whether requirements are **testable** (you can understand tests as "proof")
- Whether tradeoffs match your **priority order** (e.g., safety > correctness > UX > speed)
- Whether unknowns are handled honestly (explicit questions rather than silent assumptions)

This is enough to drive high-quality outcomes.

### The Non-Technical Advantage

Technical experts often jump to solutions. They have patterns they like, tools they're comfortable with, approaches that worked before.

You don't have that baggage. You can ask naive questions that expose hidden assumptions. You can insist on understanding before approving. You can demand that complexity justify itself.

Your fresh eyes are an asset, not a liability.

---

## The Three Rules of Lossless Decomposition

Once you have a Master Plan, you need to break it into pieces agents can execute. This is where most plans die—through lossy summarization.

### Rule 1: Size Your Chunks

Agents perform poorly on large documents. Give them a 5,000-line markdown file and they'll start summarizing, paraphrasing, losing details. Give them 500-1,500 lines and they can hold every specification in working memory.

Keep each phase document at a manageable size. That's the secret.

### Rule 2: Decompose Losslessly

When you split a large plan into smaller pieces, you do not summarize. You *reorganize*. Every decision survives. Every edge case lives somewhere.

Think of it like Russian nesting dolls. Your master plan is the biggest doll. You open it, and inside are your phases—smaller, but each one complete and fully formed. You open a phase, and inside are your tasks. Open a task, and inside are your sub-tasks.

The key: each layer is whole. Not a summary of the layer above it. Not a simplified version. A complete specification at a smaller scale.

**If a detail doesn't appear in a sub-task, that detail is gone forever.**

There's one word that makes or breaks this: **LOSSLESS.**

### Rule 3: Verify With Fresh Eyes

The agent that created the breakdown can't audit its own work. Bring in a different agent. Have it check:

- Does every section from the original appear somewhere?
- Can each task stand alone?
- Could someone implement any piece without asking a single question?

If the answer is no, you're not done.

### The Mantra

**Bounded. Complete. Verified.**

- **Bounded** = scoped to what an agent can hold in working memory
- **Complete** = every detail survives the decomposition
- **Verified** = audited by someone who wasn't there

Master these three, and your plans survive execution.

---

## Stress Testing Your Plan

Before you consider discovery complete, stress-test.

### Adversarial Review

Ask the model to attack its own suggestions:

```
Review this plan adversarially. Find:
1. Decisions that will be revisited because they weren't thought through
2. Edge cases that aren't handled
3. Failure modes without recovery strategies
4. Assumptions that could be wrong
5. Dependencies that could break
```

### Multi-Model Validation

Bring in different models and have them review:

```
A different team created this plan. Review it for:
1. Missing requirements
2. Inconsistencies between sections
3. Gaps where an implementer would need to guess
4. Over-engineering (complexity without justification)
5. Under-engineering (missing critical elements)
```

### The Implementation Test

For each section, ask:

> "If I handed this to a capable developer with no other context, could they build exactly what I want without asking me any questions?"

If the answer is "they'd probably need to ask about X"—X is a gap. Fill it.

---

## How This Connects to the Pipeline

Discovery is Stage -1. It happens before the formal pipeline begins.

```
Discovery (this document)
    ↓
Master Plan complete
    ↓
Stage 0: North Star Card (formalize goals)
    ↓
Stage 1-2: Requirements + QA (formalize REQ/AC)
    ↓
Stage 3-5: Decisions, Spikes, Plan Pack
    ↓
Stage 6-7: Phase breakdown, Bead decomposition
    ↓
Stage 8-10: Execution, Calibration, Release
```

The discovery phase isn't separate from the pipeline—it's what makes the pipeline work. Without it, you're starting with compressed requirements and hoping the model fills gaps correctly.

### What Discovery Produces vs. What the Pipeline Formalizes

| Discovery Produces | Pipeline Formalizes Into |
|-------------------|-------------------------|
| Understood architecture | System map, ADRs |
| Known data structures | Data model specification |
| Identified APIs | API contracts + error model |
| Explored failure modes | Failure modes + recovery |
| Tested assumptions | Spikes with evidence |
| Comprehensive understanding | Plan Pack artifacts |

Discovery is exploration. The pipeline is formalization. You need both.

---

## Research Support

This approach aligns with evidence from the research corpus:

### Requirements Quality is the Multiplier

Everything downstream inherits the quality of your requirements. Ambiguous requirements produce ambiguous implementations.

- `research/033-requirements-to-code.md` — Requirements quality determines code quality
- `research/036-requirements-qa-iso-29148.md` — LLMs can QA requirements effectively
- `research/037-requirements-to-code-practices.md` — Practitioners must decompose into concrete tasks

### Long Context Degrades Reasoning

Large documents cause silent omission. Chunking preserves semantic reasoning.

- `research/004-context-length-hurts.md` — 30-50% quality loss with irrelevant context
- `research/005-lost-in-middle-code.md` — Information in the middle gets ignored

### Grounding Prevents Hallucination

Verify claims against real documentation before committing to architecture.

- `research/008-api-hallucination.md` — 38% validity on rare APIs without grounding
- `research/017-context-retrieval-repo-editing.md` — Right context enables correct behavior

### Iterative Discovery Beats Single Prompts

Progressive elaboration outperforms "one big answer."

- `research/022-chatrepair.md` — Execution feedback loops improve correctness
- `research/019-plansearch.md` — Searching over diverse plans improves outcomes

---

## Next Steps

When your Master Plan is complete (nothing left to interpret):

1. **Formalize:** Enter the pipeline at Stage 0 with `IDEATION_TO_PRODUCTION.md`
2. **Decompose:** Break into phases with `DECOMPOSITION.md`
3. **Execute:** Follow TDD-first execution with calibration gates

---

## Quick Reference

### The Core Principle
Plan as much as appropriate. Give the AI as few decisions as possible.

### The Test
The plan is complete when there's nothing left to interpret.

### The Cascade
Gaps become assumptions. Assumptions become architecture. Bad architecture becomes a rewrite.

### The Three Rules
Bounded. Complete. Verified.

### The Outcome
Same hours. Different outcomes.
