# Philosophy: Make It Hard To Be Wrong

## The Gap Nobody Talks About

There's a widening divide in software development right now. Not between languages or frameworks. Between developers who've restructured how they work around AI and those still operating at manual speed.

The second group doesn't see it yet. They're keeping up—for now. But the first group is shipping in days what used to take weeks, and the quality metrics aren't suffering. They're flat or improving.

This isn't about tools. Cursor vs Claude Code vs whatever dropped last week—that's noise. The gap is **workflow**. It's how you structure the work, what you verify, when you decompose, and what you refuse to skip.

---

## What The Research Actually Shows

| Finding | Data | Source |
|---------|------|--------|
| Improvement from writing tests first | **+45.97%** | `054-tdd-ai-code-gen.md` |
| Vulnerability rate in unscanned AI code | **~40%** | `052-llm-security-vulnerabilities.md` |
| Capability decay after 2-3 failed attempts | **60-80%** | `060-debugging-decay-index.md` |
| Correct answers flipped by "are you sure?" | **58%** | `006-dark-side-self-correction.md` |
| Quality loss from irrelevant context | **30-50%** | `004-context-length-hurts.md` |

The models are good. Really good. But without the right workflow, you're leaving massive gains on the table and introducing avoidable risks.

TDD nearly doubles your success rate. Security scanning catches vulnerabilities before they ship. Iteration caps prevent the model from degrading its own work. These aren't workarounds for weak models—they're how you extract maximum value from strong ones.

---

## The Real Skill Is Management, Not Prompting

The developers getting the most out of AI aren't prompt engineers. They're the ones who spent years learning to decompose problems, specify requirements, and catch architectural drift.

Those skills transfer directly.

When you've managed junior developers, you know you can't just say "build the auth system." You scope the work. You define what done looks like. You review before it ships. You catch the stuff that technically works but doesn't fit the larger design.

AI is the same problem at higher speed. The model will confidently produce something that compiles and passes basic tests but quietly violates patterns that matter. If you can't see that, you're not getting value—you're accumulating debt.

> **Your leverage is bounded by what you understand.** The model amplifies your knowledge. It doesn't substitute for it.

---

## The Fundamental Reframe

Most people try to make the model produce correct output. That's the wrong goal.

> **Build a workflow where being wrong gets caught before it matters.**

Even great models make mistakes. The question isn't whether errors happen—it's whether your system catches them.

- **Verification** catches failures early
- **Iteration caps** prevent degradation spirals
- **Security scanning** blocks vulnerable code
- **Tests** provide ground truth the model can optimize against

You're not compensating for a weak model. You're building a system that maximizes a strong one.

---

## Why Skilled Developers Resist

If you've spent a decade getting good at something, there's gravity pulling you toward "this is how real work gets done." The grind is the point. If it doesn't hurt, it doesn't count.

That identity is expensive now.

Here's the thing though: working effectively with AI *is* demanding. You're orchestrating, verifying, and course-correcting constantly. The cognitive load shifts from "figure out the syntax" to "catch the subtle architectural mistake in 200 lines of generated code."

The challenge didn't go away. It moved.

And yes, you can get lazy and let the tool rot your skills. That's a choice. Reading every line, understanding what got generated, staying sharp—that's also a choice. The tool doesn't decide which one you make.

---

## Context Is The Bottleneck

The model doesn't know your system. It doesn't know why that workaround exists, which services talk to each other, or what patterns the team agreed on.

You have years of accumulated context. The model has whatever you put in the prompt.

> "The smallest set of high-signal tokens that enables correct behavior." — `057-anthropic-context-engineering.md`

More context isn't better. The *right* context is better. Irrelevant information degrades performance 30-50% (`004-context-length-hurts.md`). You want architecture docs, the specific files that matter, and nothing else.

This is why codemaps exist. This is why grounding tools matter. You're not fighting hallucinations with hope—you're giving the model access to verified sources so it doesn't have to guess.

**Evidence:** With grounding, the 38% validity rate on rare APIs (`008-api-hallucination.md`) becomes a non-issue. The model looks it up instead of inventing it.

---

## Tests Are The Reward Function

You can't give the model complete context. Ever. Your codebase is too big, the history too deep. The model operates on a compressed map.

Tests fill the gap.

When the model breaks something outside its context window, tests catch it. The model sees the failure, adjusts, and iterates toward correct behavior. You've given it a feedback loop it can actually optimize against.

This is why TDD isn't optional anymore. Writing tests first doesn't just catch bugs—it fundamentally changes how the model approaches the problem. The 45.97% improvement in `054-tdd-ai-code-gen.md` isn't a prompting trick. It's structural.

**The pattern:**
1. Define behavior as tests
2. Let the model implement against those tests
3. Iterate until passing
4. The tests become the spec

---

## Three Tries, Then Stop

Extended retry loops make things worse.

This is counterintuitive. You'd expect more attempts to converge on a solution. The research says otherwise:

- Security quality degrades with each self-correction cycle (`053-feedback-loop-security.md`)
- "Are you sure?" without new evidence flips 58% of *correct* answers (`006-dark-side-self-correction.md`)
- Capability drops 60-80% within 2-3 attempts (`060-debugging-decay-index.md`)

**The rule:** Three iterations maximum. If it's still failing:

1. Stop the current approach
2. Decompose—break the problem into smaller pieces
3. Or escalate to human judgment

Continuing past three is actively counterproductive. You're not being persistent. You're degrading the output.

---

## Disagreement Resolves Through Tests, Not Discussion

When two approaches conflict, extended debate makes things worse. Voting alone outperforms unstructured back-and-forth (`003-debate-or-vote.md`).

**What works:**
1. Write a discriminating test—one that passes for approach A, fails for approach B
2. Run it
3. Accept the result

If you can't write a test that discriminates, you don't have a technical disagreement. You have a values disagreement. Preserve both positions and escalate to human decision.

Rhetorical cross-examination doesn't converge on truth. Tests do.

---

## The Working Pattern

### 1. Requirements
- Model restates the problem (catches misunderstanding early)
- Load relevant context only (codemaps, specific files)
- Declare TDD as the approach

### 2. Planning
- Outline steps before writing code
- Ground against real docs (verify APIs exist, patterns match)
- Approve before implementation starts

### 3. Implementation
- Tests first
- Interfaces and signatures before bodies
- Implement to pass tests
- Small increments, verified individually

### 4. Verification
- Run tests, analyze failures
- Fix and re-run (max 3 iterations)
- Security scan (`ubs --staged`)
- Read every line that got generated

**Research backing:**
- Requirements: `057-anthropic-context-engineering.md`
- Planning: `002-planning-driven-programming.md` (+2-10%)
- Implementation: `054-tdd-ai-code-gen.md` (+45.97%)
- Verification: `022-chatrepair.md`, `053-feedback-loop-security.md`

---

## Decompose When Reality Demands It

Don't pre-plan everything. You don't know what's actually hard until you try.

**ADaPT pattern:**
1. Start coarse
2. Attempt execution
3. If it fails after 3 iterations, decompose *that specific part*
4. Repeat

This discovers actual complexity instead of guessing at it. Pre-decomposition is speculation. Failure-driven decomposition is evidence.

**Evidence:** `038-adapt.md`, `011-agentless.md` (simple pipelines beat complex agents), `069-adacoder.md`

---

## What Still Fails (With Evidence)

| Pattern | Failure Mode | Research |
|---------|--------------|----------|
| Tests after implementation | Miss 45.97% improvement | `054-tdd-ai-code-gen.md` |
| Unlimited retries | Security + capability degrade | `053-feedback-loop-security.md`, `060-debugging-decay-index.md` |
| Skipping security scan | ~40% vulnerability rate | `052-llm-security-vulnerabilities.md` |
| Pre-decomposing everything | Guessing at complexity | `038-adapt.md` |
| Extended debate | Degrades vs voting alone | `003-debate-or-vote.md` |
| Huge context windows | 30-50% quality loss | `004-context-length-hurts.md` |
| "Are you sure?" without evidence | Flips 58% of correct answers | `006-dark-side-self-correction.md` |

---

## Reading Matters More Than Generating

When generation is cheap, comprehension becomes the bottleneck.

Every line the model produces needs to be understood. Not skimmed—understood. The model will slip in changes that technically work but don't fit. That's your job to catch.

When you stop reading:
- Refactors slip through without understanding
- You lose track of what exists in the codebase
- Prompts get vaguer because you don't know what to specify
- Six months later you're dependent on a system you can't audit

Your eyes are the final gate. Tests catch functional breaks. You catch architectural ones.

---

## The Bar Moved

This isn't about whether AI is good or bad for engineering. That's not the debate.

The capability exists. Some people are using it effectively. The gap between them and everyone else is real and widening.

The patterns that work are clear: verification over trust, tests over debate, iteration caps over persistence, decomposition on failure, reading over hoping.

The tools will keep changing. These principles won't.

---

## Next

- **Lifecycle and gates:** `docs/workflow/EVIDENCE_BASED_GUIDE.md`
- **Protocols:** `docs/workflow/PROTOCOLS.md`
- **Decomposition mechanics:** `docs/workflow/DECOMPOSITION.md`
- **Planning deep dive:** `docs/workflow/PLANNING_DEEP_DIVE.md`
- **Research index:** `research/README.md`
