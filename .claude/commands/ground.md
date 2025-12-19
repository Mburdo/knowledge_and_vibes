---
description: Choose the right grounding tool (Warp-Grep vs Exa vs CASS/cm)
argument-hint: [question-or-task]
---

# Ground — Grounding Protocol

Use this command when you're about to answer a question or start a task and you're not sure whether the truth is **in the repo**, **on the web**, or **in prior sessions**.

---

## Why Grounding Matters

**AI-generated code follows training data, which may be outdated, deprecated, or wrong.**

LLMs are trained on snapshots of documentation, tutorials, and code samples from months or years ago. This means:

- **Deprecated APIs**: The AI might use `library.oldMethod()` when `library.newMethod()` replaced it 6 months ago
- **Outdated patterns**: The "recommended approach" from 2023 might be an anti-pattern in 2024
- **Hallucinated methods**: For niche libraries, the AI may confidently invent methods that never existed
- **Wrong defaults**: Configuration options, flags, and parameters change across versions

**Grounding pulls your code back to reality.**

Before implementing anything that touches external libraries, APIs, or frameworks, use Exa to fetch the **current** documentation. This sanity-checks your implementation against what actually exists today.

**This is critical for non-technical users** who can't spot when AI is hallucinating outdated patterns. If you can't read the code and know "that method was deprecated in v3.0," grounding is your safety net.

---

## Usage

```
/ground [question-or-task]
```

## One-line decision rule

- **Repo truth** → **Warp-Grep** (local codebase discovery)
- **Web truth** → **Exa** (current external docs/APIs/examples)
- **History truth** → **CASS** / **cass-memory (`cm`)** (what we did/learned before)
- **Task/graph truth** → **Beads Viewer (`bv`)** (what to do next, what blocks what, what changed)

## When to use what

### Warp-Grep (repo discovery)
Use when you believe the answer exists somewhere in the current repo, but you don’t know where:
- “Where is X implemented?”
- “What calls Y / where does this error originate?”
- “How does data flow from A → B across modules?”

Avoid:
- Using Exa to search for code that’s already in this repo.

### Exa (external grounding)
Use when the correct answer depends on **current** information outside the repo:
- Library/framework docs that change (APIs, flags, deprecations)
- Vendor integrations, auth flows, pricing/limits, “latest recommended pattern”
- Finding real-world OSS examples for a pattern

Avoid:
- Using Exa as a substitute for reading the code you already have.

### CASS + `cm` (history + distilled rules)
Use when you suspect we’ve solved it before or have a house style:
- “Have we seen this bug before?”
- “What conventions do we follow for X?”

Protocol:
1. `cm context "<task>" --json` (fast distilled rules + suggested searches)
2. If needed: `cass search "..." --robot --fields minimal --limit 5`
3. Then open the best match / expand around it.

## Defaults (safe)

- Prefer **Warp-Grep first** for repo questions.
- Use **Exa only** when you explicitly need external/current facts.
- Use **`cm` first** when looking for “how we do it”.
- Use **`bv` first** when the question is “what should we do next?” or “how does the task graph look?”.

### Beads Viewer (`bv`) (task graph truth)
Use when the question is about the *project’s work graph*, not the code or the web:
- “What should I work on next?”
- “Why is this bead blocked / what’s the blocker chain?”
- “What looks risky/stale right now?”
- “What changed in the task graph since yesterday / last commit?”

Opinionated defaults:
```bash
bv --robot-next                    # Fast: single best next task
bv --robot-triage                  # Full: blockers, quick wins, commands
bv --robot-blocker-chain bd-123    # If blocked: show the chain to clear
bv --robot-alerts                  # Risk/hygiene signals (stale, cascades, drift)
bv --robot-diff --diff-since HEAD~5  # Graph changes since a ref
```

## Output you should produce (when using this command)

State explicitly:
1. Which bucket your question falls into (repo/web/history)
2. Which tool you’re using and why
3. The next concrete action you’ll take (the exact search you’ll run)
