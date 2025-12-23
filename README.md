<div align="center">

# Knowledge & Vibes

### A framework for building software with AI

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Research-Backed](https://img.shields.io/badge/Research-73%20Papers-green.svg)](./research/README.md)

</div>

---

## What This Is

A structured workflow for building real software with AI assistance. Plans are explicit, work is tracked, and verification is mandatory.

The core insight: **truth lives outside the model.** The AI's confident output is not truth. Truth is tests that pass, code that compiles, documentation that exists. Everything else is a hypothesis that needs verification.

---

## The Problem It Solves

AI-assisted development fails in predictable ways:

- **The AI builds the wrong thing** because goals weren't explicit
- **Requirements vanish mid-project** because context windows have limits
- **Multiple agents conflict** because there's no coordination protocol
- **Bugs compound silently** because there are no verification gates
- **"It works" isn't evidence** because confidence doesn't equal correctness

You can't prompt your way out of these problems. You need a system where failures get caught before they matter.

---

## How It Works

### 1. Plan Explicitly

Before anyone writes code, the goal is pinned down. A North Star Card captures what success looks like, what's out of scope, and when the AI should stop and ask. Requirements are written in testable terms. Decisions are recorded so they're not relitigated.

### 2. Track Everything

Work is broken into **beads**: tasks with dependencies, status, and verification requirements. Nothing gets forgotten. Nothing falls through the cracks. The graph of work is explicit and queryable.

### 3. Coordinate Agents

When multiple AI agents work simultaneously, they need a protocol. File reservations prevent conflicts. Claim/close announcements tell everyone what's taken. Calibration checkpoints catch drift before it compounds.

### 4. Verify Continuously

Tests are written before implementation (TDD). Security scans run before every commit. If something fails after three attempts, it gets decomposed or escalated rather than retried indefinitely.

---

## Full Workflow

```mermaid
flowchart TB
    START([AI Software Build]) --> ENTRY{How to run?}
    ENTRY -->|Automated| AUTO[Run /full-pipeline]
    ENTRY -->|Manual| MANUAL[Follow stages 0-10]
    AUTO --> S0
    MANUAL --> S0

    subgraph PLAN[Stages 0-5: Planning]
        S0[Stage 0: North Star] --> G0{Intent clear?}
        G0 -->|no| S0
        G0 -->|yes| S1

        S1[Stage 1: Requirements] --> G1{P0 REQs have ACs?}
        G1 -->|no| S1
        G1 -->|yes| S2

        S2[Stage 2: QA] --> G2{No ambiguity?}
        G2 -->|no| S2
        G2 -->|yes| S3

        S3[Stage 3: Decisions] --> G3{All decided?}
        G3 -->|no| S3
        G3 -->|yes| S4

        S4[Stage 4: Spikes] --> G4{Unknowns resolved?}
        G4 -->|no| S4
        G4 -->|yes| S5

        S5[Stage 5: Plan Pack] --> G5{No guessing needed?}
        G5 -->|no| S5
    end

    subgraph BREAK[Stages 6-7: Breakdown]
        S6[Stage 6: Phases] --> G6{Context bounded?}
        G6 -->|no| S6
        G6 -->|yes| S7

        S7[Stage 7: Beads] --> G7{Tests defined first?}
        G7 -->|no| S7
    end

    subgraph EXEC[Stage 8: Execution]
        COORD[Coordinator] --> SPAWN[Spawn agents]

        subgraph WORK[Worker Loop]
            PRIME[/prime] --> RESERVE[Reserve files]
            RESERVE --> CLAIM[Announce CLAIMED]
            CLAIM --> TDD[Write tests FIRST]
            TDD --> IMPL[Implement]
            IMPL --> TEST{Pass?}
            TEST -->|no| RETRY{Try 3?}
            RETRY -->|yes| IMPL
            RETRY -->|no| DECOMP[ADaPT: decompose]
            TEST -->|yes| SEC[ubs --staged]
            SEC --> CLOSE[Close bead]
            CLOSE --> RELEASE[Release files]
        end

        SPAWN --> WORK
        WORK --> TRACK{Track done?}
        TRACK -->|no| PRIME
        TRACK -->|yes| PDONE{Phase done?}
        PDONE -->|no| SPAWN
    end

    subgraph CAL[Stage 9: Calibration]
        CALSTART[/calibrate] --> COV[Coverage check]
        COV --> DRIFT[Drift check]
        DRIFT --> CHALLENGE[Test-based resolution]
        CHALLENGE --> SYNTH[Synthesis]
        SYNTH --> REPORT[User report]
        REPORT --> CALGATE{Passed?}
    end

    subgraph REL[Stage 10: Release]
        VERIFY[Full verification] --> TRACE[Traceability complete]
        TRACE --> ACCEPT[Operator acceptance]
        ACCEPT --> RELGATE{Ready?}
        RELGATE -->|yes| SHIP[Ship it]
    end

    G5 -->|yes| S6
    G7 -->|yes| COORD
    PDONE -->|yes| CALSTART
    CALGATE -->|pass| VERIFY
    CALGATE -->|fail| S0
    RELGATE -->|no| COORD
    SHIP --> POST[Learning loop]
    POST --> DONE([Complete])

    classDef stage fill:#e7f1ff,stroke:#1e88e5
    classDef gate fill:#fff3cd,stroke:#d39e00
    classDef cmd fill:#f3e6ff,stroke:#6a1b9a
    classDef work fill:#ffebee,stroke:#c62828

    class S0,S1,S2,S3,S4,S5,S6,S7 stage
    class G0,G1,G2,G3,G4,G5,G6,G7,TEST,RETRY,TRACK,PDONE,CALGATE,RELGATE gate
    class AUTO,PRIME,CALSTART cmd
    class COORD,SPAWN,TDD,IMPL,DECOMP,COV,DRIFT,CHALLENGE,SYNTH,VERIFY work
```

---

## The Research Behind It

Every protocol is backed by research. 73 papers distilled into actionable practices:

- Why TDD produces better outcomes with AI
- Why long context degrades reasoning
- Why orchestrator-worker patterns outperform single agents
- Why extended self-correction makes things worse
- Why tests should adjudicate disagreements, not rhetoric

See the [Research summaries](./research/README.md) for the full collection.

---

## What's In This Repository

| Section | What You'll Find |
|:--------|:-----------------|
| [**Setup Guide**](./docs/guides/SETUP_GUIDE.md) | How to install and configure the toolchain |
| [**Evidence-Based Guide**](./docs/workflow/EVIDENCE_BASED_GUIDE.md) | The complete 10-stage pipeline |
| [**Protocols**](./docs/workflow/PROTOCOLS.md) | 19 repeatable procedures for common situations |
| [**Templates**](./TEMPLATES.md) | North Star cards, requirements, ADRs, and more |
| [**Glossary**](./GLOSSARY.md) | Every term defined |

Start with [**START_HERE.md**](./START_HERE.md) for the recommended reading order.

---

## About

This framework is the distillation of three years of building with AI, starting from zero.

When GPT-3.5 launched, I was in high finance. I couldn't write a line of code. But I recognized immediately that AI was going to fundamentally change how things get built, and I wanted to be part of it.

So I went all in. Not with tutorials or bootcamps, but with a first principles approach: *What can these models actually do? Where do they fail? How do you extract every ounce of capability while catching the inevitable mistakes?*

The early days were rough. The models were weaker. The tooling didn't exist. Every technique had to be discovered through trial and error. But I stayed in it, learning, building, refining, session after session, project after project.

Three years later, I'm shipping complex applications with real users and real revenue. Not because I became a traditional developer, but because I learned how to work *with* AI in a way that produces reliable results.

This framework is everything I've learned, systematized. The protocols that prevent the common failures. The tools that catch mistakes before they ship. The workflow that turns AI-assisted coding from a gamble into a repeatable process.

**If you're technical, this will make you faster. If you're not, this is proof that you can build real things anyway.**

---

<div align="center">

**[Follow my work →](https://x.com/YachtsmanCap)**

[![Twitter Follow](https://img.shields.io/twitter/follow/YachtsmanCap?style=social)](https://x.com/YachtsmanCap)

MIT License

</div>
