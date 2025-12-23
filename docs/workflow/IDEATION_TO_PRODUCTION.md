---
title: Ideation to Shipped Product
description: Complete end-to-end pipeline from idea to shipped product. Single-page reference for the full 10-stage lifecycle.
category: workflow
---

<div align="center">

# Ideation → Shipped Product

### Complete end-to-end pipeline from idea to ship

</div>

The complete end-to-end pipeline from idea to shipped product. This is the single-page reference for the full lifecycle.

---

## Table of Contents

- [How This Relates to Other Docs](#how-this-document-relates-to-others)
- [Why This Pipeline Exists](#why-this-pipeline-exists)
- [First Principles](#first-principles-what-were-optimizing-for)
- [The Pipeline](#the-pipeline-operatorled-stages--gates)
  - [Stage -1: Discovery](#stage-1--discovery-before-the-pipeline)
  - [Stage 0: Rigor Mode](#stage-0--set-the-rigor-mode-context-drives-process)
  - [Stage 1: Requirements](#stage-1--requirements-that-compile-reqac-as-the-operator-interface)
  - [Stage 2: Requirements QA](#stage-2--requirements-qa-pass-isostyle-make-ambiguity-expensive)
  - [Stage 3: Decision Search](#stage-3--decision-search-multiple-viable-plans-then-select)
  - [Stage 4: Spikes](#stage-4--riskdriven-spikes-collapse-uncertainty-before-committing)
  - [Stage 5: Plan Pack](#stage-5--plan-pack-assembly-turn-decisions-into-lossless-specs)
  - [Stage 6: Phase Breakdown](#stage-6--phase-breakdown-short-context-planning-to-avoid-omission)
  - [Stage 7: Bead Decomposition](#stage-7--bead-decomposition-convert-phases-to-executable-task-units)
  - [Stage 8: Execution](#stage-8--execution-loops-generate--run--fix-not-think-harder)
  - [Stage 9: Calibration](#stage-9--calibrate-search-controller--drift-correction)
  - [Stage 10: Release](#stage-10--release-readiness-make-done-real)
- [The Operator's Superpower](#the-operators-superpower-what-you-actually-need-to-be-good-at)
- [Minimal Checklist](#minimal-how-to-run-this-one-checklist)

---

## How This Document Relates to Others

| Document | Focus |
|----------|-------|
| `DISCOVERY.md` | Pre-pipeline: curiosity-driven architecture |
| `EVIDENCE_BASED_GUIDE.md` | Same pipeline, more philosophy |
| `PROTOCOLS.md` | Individual protocol cards |
| `PLANNING_DEEP_DIVE.md` | Deep dive on planning |
| `DECOMPOSITION.md` | Decomposition mechanics |
| This document | **Complete lifecycle overview** |

All of these are consistent. This one shows the full picture. Start with `DISCOVERY.md` for new projects.

---

## Why This Pipeline Exists

AI-assisted development fails predictably without structure:
- **~40%** of AI code has vulnerabilities without scanning
- **Long context** causes silent omission of requirements
- **Unlimited retries** degrade quality (60-80% capability loss after 2-3 attempts)
- **No verification** lets mistakes compound silently

This pipeline exists because **workflow beats prompting**. You can't prompt your way to reliability, but you can build a system where failures get caught.

---

## First Principles (What We’re Optimizing For)

### Goal
Maximize probability of: **correct software that matches the user’s real intent**, under two constraints:
1) the “operator” cannot personally validate technical details  
2) LLMs are powerful but fallible (hallucinations, long‑context brittleness, tool errors)

### Therefore, the pipeline must:
- convert intent into **testable requirements** (so “correct” is measurable)
- force key decisions into **explicit artifacts** (so agents can’t silently diverge)
- prefer **execution and retrieval** over “trusting reasoning”
- use **diversity + selection** instead of betting on one answer
- keep reasoning in **short, well‑scoped contexts**

**Evidence base (selected):**  
Repo‑scale evaluation + bottlenecks: `research/009-swe-bench.md`, `research/016-res-q.md`  
Interfaces/tools matter: `research/010-swe-agent.md`  
Simple fixed pipelines often win: `research/011-agentless.md`, `research/012-autocoderover.md`  
Long context harms reasoning: `research/004-context-length-hurts.md`, `research/005-lost-in-middle-code.md`  
Execution‑feedback loops help; unguided self‑correction can hurt: `research/022-chatrepair.md`, `research/006-dark-side-self-correction.md`  
Diversity + search improves outcomes: `research/019-plansearch.md`, `research/020-codetree.md`, `research/042-rankef.md`  
Adaptive decomposition helps when reality shifts: `research/038-adapt.md`, `research/039-tdag.md`  
Multi-stage “human loop” pipelines work: `research/040-mapcoder.md`  
Requirements quality is leverage: `research/033-requirements-to-code.md`, `research/036-requirements-qa-iso-29148.md`, `research/037-requirements-to-code-practices.md`

---

## The Pipeline (Operator‑Led Stages + Gates)

Each stage has:
- **Operator job** (what a nontechnical person can do)
- **Agent output artifacts** (what must exist on disk)
- **Gate** (what must be true before moving on)

### Stage -1 — Discovery (Before the Pipeline)

Before the formal pipeline, there's the work that determines everything else.

**The Core Principle:** Plan as much as appropriate. Give the AI as few decisions as possible.

Any decision you don't claim, you implicitly delegate. When you delegate to the model, it fills gaps with training data—not your intent. Those gaps become assumptions. Assumptions become architecture. Bad architecture becomes a rewrite.

**Operator job**
- Interrogate a frontier reasoning model with relentless curiosity
- Surface every decision hiding in your idea
- Document architecture choices before any agent writes code
- Pull on every thread until you understand: "What are my options? What are the tradeoffs? What happens if I choose wrong?"

**Artifacts**
- Master Plan (comprehensive specification, typically 1,000-5,000 lines depending on project complexity)
- All architecture decisions explicit
- All data models defined
- All error handling specified
- All edge cases documented

**Gate**
- The plan is complete when there's nothing left to interpret
- Any capable agent could implement any section without asking clarifying questions

**The test:** If an agent would need to ask "what should happen here?" or "which approach do you want?"—the plan isn't done. That's a gap. And gaps become assumptions.

**For non-technical users:** The lever is relentless curiosity, not technical skill. You won't become a programmer, but you'll understand enough to plan accurately and ask the right questions. Each project you work through, you learn more. What felt overwhelming on project one feels manageable by project five.

**Evidence base:** Requirements quality is the multiplier—everything downstream inherits the quality of this phase.
`research/033-requirements-to-code.md`, `research/036-requirements-qa-iso-29148.md`, `research/037-requirements-to-code-practices.md`

**Full guide:** `docs/workflow/DISCOVERY.md`

> **Same hours. Different outcomes.** The time gets spent either way. You either spend it upfront, thinking through decisions before the model touches them. Or you spend it later, untangling decisions the model made without you.

---

### Stage 0 — Set the Rigor Mode (Context drives process)

**Operator job**
- Choose the build profile + rigor tier in the North Star Card.

**Artifacts**
- `templates/NORTH_STAR_CARD_TEMPLATE.md` filled and pinned at top of plan.

**Gate**
- The card is explicit enough that agents can resolve tradeoffs without guessing.

**Evidence base:** requirements/intent quality is the multiplier (RE papers) and weak signals inflate confidence (SWE‑Bench+).  
`research/035-llm-vs-human-re.md`, `research/036-requirements-qa-iso-29148.md`, `research/021-swe-bench-plus.md`

---

### Stage 1 — Requirements That "Compile" (REQ/AC as the operator interface)

**Operator job**
- Answer clarifying questions about outcomes and constraints (not architecture).
- Confirm priorities and non‑goals.

**Artifacts**
- `PLAN/01_requirements.md` containing:
  - `REQ-*` (observable outcomes/constraints)
  - `AC-*` (acceptance criteria, ideally test‑shaped)
  - priority (P0/P1/P2)
  - owner of truth (doc/person/system)

**Gate**
- Every `REQ-*` has at least one `AC-*` that is falsifiable.

**Evidence base:** tests improve correctness; requirements quality is decisive; practitioners must decompose into concrete tasks.  
`research/032-tdd-code-generation.md`, `research/033-requirements-to-code.md`, `research/037-requirements-to-code-practices.md`

---

### Stage 2 — Requirements QA Pass (ISO‑style "make ambiguity expensive")

**Operator job**
- Review and approve requirement rewrites when the LLM flags ambiguity.

**Artifacts**
- Updated `PLAN/01_requirements.md` with:
  - ambiguous requirements rewritten
  - missing constraints added
  - inconsistent requirements resolved
  - QA log (copy `templates/REQUIREMENTS_QA_TEMPLATE.md` into your plan pack or paste results into the requirements doc)

**Gate**
- No P0 requirement remains ambiguous/untestable.

**Evidence base:** LLMs can effectively QA requirements and propose improvements; explanations help humans accept changes.  
`research/036-requirements-qa-iso-29148.md`, `research/034-llms-in-re-guideline.md`

---

### Stage 3 — Decision Search (multiple viable plans, then select)

**Operator job**
- Choose between options based on tradeoffs you understand (cost/speed/risk/UX/security), not implementation details.

**Artifacts**
- `PLAN/02_decisions_adrs.md` containing ADRs with:
  - Options A/B/C
  - rationale
  - risks
  - “what would change our mind” (reversal triggers)

**Gate**
- Every “architecturally significant requirement” has a recorded decision or an explicit “TBD pending spike”.

**Evidence base:** searching over diverse plans improves outcomes; tree search + pruning beats single-path.  
`research/019-plansearch.md`, `research/020-codetree.md`

---

### Stage 4 — Risk‑Driven Spikes (collapse uncertainty before committing)

**Operator job**
- Approve which unknowns matter most (top 5–10).

**Artifacts**
- `PLAN/06_risks_and_spikes.md` (copy `templates/RISKS_AND_SPIKES_TEMPLATE.md`) listing:
  - top risks / unknowns
  - spike beads that produce evidence (docs, prototype, measurement)
  - expected decision impact (which ADR it unlocks)

**Gate**
- High‑risk unknowns are converted into spikes with explicit outputs.

**Evidence base:** flow engineering + execution feedback loops improve reliability; adaptive planning via oracles prevents rigid plans.  
`research/031-alphacodium.md`, `research/014-codeplan.md`, `research/022-chatrepair.md`

---

### Stage 5 — Plan Pack Assembly (turn decisions into lossless specs)

**Operator job**
- Confirm the plan matches the North Star and constraints.
- Answer any remaining “stop/ask” questions.

**Artifacts**
At minimum (folder or single master file):
- `PLAN/03_system_map.md`
- `PLAN/04_data_model.md`
- `PLAN/05_api_and_error_model.md`
- `PLAN/08_testing_spec.md`
- `PLAN/09_ops_security.md`
- `PLAN/10_dependencies_and_phases.md`

**Gate**
- A competent engineer could implement from the Plan Pack without guessing.

**Evidence base:** requirements→design→tests→code progressive artifact chains improve inspectability; repo work needs explicit constraints packaged into tasks.  
`research/033-requirements-to-code.md`, `research/037-requirements-to-code-practices.md`

---

### Stage 6 — Phase Breakdown (short-context planning to avoid omission)

**Operator job**
- Approve phase boundaries and calibration points (where you want a “hard stop”).

**Artifacts**
- Phase documents (from `.claude/templates/planning/phase-document.md`)
- Calibration points marked in dependency graph.

**Gate**
- No phase is so large that critical constraints are “lost in the middle.”

**Evidence base:** long contexts degrade reasoning even with perfect retrieval; chunking keeps semantic reasoning reliable.  
`research/004-context-length-hurts.md`, `research/005-lost-in-middle-code.md`

---

### Stage 7 — Bead Decomposition (convert phases to executable task units)

**Operator job**
- Approve bead titles and ordering (you don’t need to judge code).

**Artifacts**
- Beads created via `bd` and `/decompose-task`:
  - each sub‑bead is implementable in a bounded time window
  - each bead includes North Star + REQ/AC references + verification plan
  - decompose **as-needed**: start coarse, then split when execution reveals uncertainty (don’t pre-split everything)

**Gate**
- Every bead has a “definition of done” that maps to tests/AC.

**Evidence base:** practitioners decompose requirements into programming tasks; fixed pipelines reduce chaos.  
`research/037-requirements-to-code-practices.md`, `research/011-agentless.md`, `research/038-adapt.md`, `research/039-tdag.md`

---

### Stage 8 — Execution Loops (generate → run → fix, not "think harder")

**Operator job**
- Only intervene when agents hit Stop/Ask (intent ambiguity) or when calibration requests a decision.

**Artifacts**
Per bead:
- code changes
- tests added/updated for `AC-*`
- verification evidence (commands run, outputs)
  - if blocked: capture the failing signal (test, stack trace, repro) and switch into an explicit repair loop (localize → patch → re-run)

**Gate**
- Beads are not “done” until verification passes.

**Evidence base:** iterative execution feedback loops improve correctness; unguided self-correction can harm.  
`research/022-chatrepair.md`, `research/018-livecodebench.md`, `research/006-dark-side-self-correction.md`, `research/045-repairagent.md`, `research/043-rlef.md`, `research/048-eg-cfg.md`, `research/046-d4c.md`

---

### Stage 9 — /calibrate (search controller + drift correction)

**Operator job**
- Choose between evidence-backed branches, approve plan changes.

**Artifacts**
- Calibration Summary + Decisions + Plan changes (see `.claude/templates/calibration/*.md`)

**Gate**
- No next phase begins until the calibration decision memo is produced.
  - disagreements about correctness must cash out into: a test that distinguishes options, or a grounded citation, or an experiment spike

**Evidence base:** unstructured debate can degrade; independent proposals + selection is stronger; planning should branch/prune.  
`research/003-debate-or-vote.md`, `research/041-debatecoder.md`, `research/019-plansearch.md`, `research/020-codetree.md`, `research/042-rankef.md`

---

### Stage 10 — Release Readiness (make "done" real)

**Operator job**
- Confirm the product meets your success metrics and acceptable risk level.

**Artifacts**
- Release checklist tied to rigor tier (tests, observability, rollback plan, security posture)

**Gate**
- The system can be deployed and supported at the chosen rigor tier.

**Evidence base:** benchmark hygiene + drift: strong results on static benchmarks don’t guarantee robustness; fresh evaluation reveals brittleness.  
`research/028-swe-bench-live.md`, `research/029-swe-rebench.md`, `research/021-swe-bench-plus.md`, `research/047-humaneval-pro.md`, `research/044-iris.md`

---

## The Operator’s Superpower (What You Actually Need to Be Good At)

You don’t need to be a software engineer. You need to be able to:
- state goals and priorities clearly (North Star)
- answer clarifying questions honestly
- choose between tradeoffs (options A/B/C)
- insist that “correct” means **testable** (AC and verification)

Everything else is engineered into the workflow.

---

## Minimal "How to Run This" (One Checklist)

**Stage -1: Discovery** (see `DISCOVERY.md`)
0. Interrogate a frontier model with relentless curiosity until you understand every decision hiding in your idea
0. Produce a Master Plan where nothing is left to interpret (test: could any agent implement without asking questions?)

**Stages 0-10: The Pipeline**
1. Fill and pin `templates/NORTH_STAR_CARD_TEMPLATE.md`
2. Produce `REQ-*` + `AC-*` (use `templates/REQUIREMENTS_TEMPLATE.md`) and run a requirements QA pass (use `templates/REQUIREMENTS_QA_TEMPLATE.md`)
3. Generate 2–4 architecture candidates, select, write ADRs (use `templates/DECISIONS_ADRS_TEMPLATE.md`)
4. Convert top unknowns into spike beads; run them (use `templates/RISKS_AND_SPIKES_TEMPLATE.md`)
5. Assemble the Plan Pack (contracts/data/errors/tests)
6. Break into phases and mark calibration points — **Bounded. Complete. Verified.**
7. Decompose phases into beads; validate graph with `bv --robot-*`
8. Execute beads with test-driven loops
9. Run `/calibrate` between phases; update plan
10. Ship with a rigor-tier release checklist

> **Same hours. Different outcomes.** The discovery phase isn't extra work—it's where you spend the time you'd otherwise spend debugging assumptions.
