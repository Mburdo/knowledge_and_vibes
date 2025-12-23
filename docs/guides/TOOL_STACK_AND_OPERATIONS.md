---
title: Tool Stack & Operations Guide
description: What you do vs what the agent does behind the scenes for each operation. Complete reference for slash commands and tool usage.
category: guides
---

# Tool Stack & Operations Guide

This guide explains what **you** do vs what **the agent** does behind the scenes for each operation.

---

## Table of Contents

**Overview**
- [The Tool Stack](#the-tool-stack)
- [Available Skills](#available-skills)
- [Orchestrator-Subagent Pattern](#orchestrator-subagent-pattern)

**Slash Commands**
- [/prime ,  Session Startup](#prime--session-startup)
- [/next-bead ,  Find and Claim Work](#next-bead--find-and-claim-work)
- [/execute ,  Parallel Execution](#execute--parallel-execution)
- [/decompose-task ,  Break Down Work](#decompose-task--break-down-work)
- [/calibrate ,  Hard Stop and Realign](#calibrate--hard-stop-and-realign)
- [/resolve ,  Disagreement Resolution](#resolve--disagreement-resolution)
- [/ground ,  Verify External Claims](#ground--verify-external-claims)

**Coordination**
- [The Agent Mail System](#the-agent-mail-system)
- [Multi-Agent Execution](#multi-agent-execution-whats-happening)
- [The Bead Lifecycle](#the-bead-lifecycle)

**Reference**
- [Security Gate: ubs --staged](#security-gate-ubs---staged)
- [Quick Reference](#quick-reference-what-you-type-vs-what-agent-does)
- [Troubleshooting](#troubleshooting)

---

## Key Distinction

| Term | Meaning |
|------|---------|
| **User actions** | What you type or decide |
| **Agent operations** | What the AI does automatically |
| **Skills** | The `.claude/skills/` files that define agent behavior |

---

## The Tool Stack

### Overview: What Each Tool Does

| Tool | Command/Access | Purpose | Failure Mode It Prevents |
|------|----------------|---------|--------------------------|
| **Beads** | `bd` | Task tracking, dependency graph | Lost work, unclear ownership |
| **BV** | `bv --robot-*` | Graph intelligence, execution planning | Wrong order, missed blockers |
| **Agent Mail** | MCP tools | Multi-agent coordination | File conflicts, duplicate work |
| **Warp-Grep** | MCP tool | Codebase search | Hallucinated code references |
| **Exa** | MCP tool | Web/docs search | Outdated API patterns |
| **UBS** | `ubs --staged` | Security scanning | Shipping vulnerable code |
| **CASS/cm** | `cass`, `cm` | History, learned patterns | Repeating past mistakes |

### Available Skills

Skills are located in `.claude/skills/`. Each defines agent behavior for specific operations:

| Skill | File | Triggers |
|-------|------|----------|
| **prime** | `.claude/skills/prime/SKILL.md` | `/prime`, "startup", new session |
| **next-bead** | `.claude/skills/next-bead/SKILL.md` | `/next-bead`, "next task", "what's next" |
| **execute** | `.claude/skills/execute/SKILL.md` | `/execute`, parallel execution |
| **bead-workflow** | `.claude/skills/bead-workflow/SKILL.md` | Claiming, closing, bead lifecycle |
| **decompose-task** | `.claude/skills/decompose-task/SKILL.md` | `/decompose-task`, phase breakdown |
| **calibrate** | `.claude/skills/calibrate/SKILL.md` | `/calibrate`, phase boundaries |
| **disagreement-resolution** | `.claude/skills/disagreement-resolution/SKILL.md` | Agent conflicts, test-based adjudication |
| **agent-mail** | `.claude/skills/agent-mail/SKILL.md` | Multi-agent messaging |
| **ubs-scanner** | `.claude/skills/ubs-scanner/SKILL.md` | Security scanning |
| **cass-memory** | `.claude/skills/cass-memory/SKILL.md` | Session history storage |
| **cass-search** | `.claude/skills/cass-search/SKILL.md` | History retrieval |
| **warp-grep** | `.claude/skills/warp-grep/SKILL.md` | Codebase search |
| **exa-search** | `.claude/skills/exa-search/SKILL.md` | Web documentation |

### How They Work Together

```
                    ┌─────────────────────────────────────────────┐
                    │              Agent Mail (MCP)               │
                    │   Registration, Messaging, File Reservations│
                    └─────────────────────────────────────────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        │                                │                                │
        ▼                                ▼                                ▼
┌───────────────┐              ┌───────────────┐              ┌───────────────┐
│   Agent A     │              │   Agent B     │              │   Agent C     │
│  (BlueLake)   │◄────────────►│  (GreenCastle)│◄────────────►│  (RedStone)   │
└───────────────┘   Messages   └───────────────┘   Messages   └───────────────┘
        │                                │                                │
        │ Claim                          │ Claim                          │ Claim
        ▼                                ▼                                ▼
┌───────────────┐              ┌───────────────┐              ┌───────────────┐
│  Beads (bd)   │              │  Beads (bd)   │              │  Beads (bd)   │
│  Task Graph   │◄─────────────│  Task Graph   │─────────────►│  Task Graph   │
└───────────────┘   Shared     └───────────────┘   Shared     └───────────────┘
        │          via Git              │          via Git             │
        ▼                                ▼                              ▼
┌───────────────┐              ┌───────────────┐              ┌───────────────┐
│   BV (bv)     │              │   BV (bv)     │              │   BV (bv)     │
│ Graph Intel   │              │ Graph Intel   │              │ Graph Intel   │
└───────────────┘              └───────────────┘              └───────────────┘
```

### Orchestrator-Subagent Pattern

For complex skills with multiple phases, we use the **orchestrator-subagent pattern**. This is how skills like `/calibrate` work:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SKILL ORCHESTRATOR                            │
│  - Creates session directory for artifacts                       │
│  - Manages TodoWrite state (progress tracking)                   │
│  - Spawns subagents with minimal, targeted context               │
│  - Passes report_path + summary between phases                   │
└─────────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Phase 1 Agent  │  │  Phase 2 Agent  │  │  Phase 3 Agent  │
│  Fresh context  │  │  Fresh context  │  │  Fresh context  │
│  Specific task  │  │  Specific task  │  │  Specific task  │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                    │
         ▼                    ▼                    ▼
    report_path +        report_path +        report_path +
    summary              summary              summary
```

**Why this works:**
- Each subagent gets **fresh context** (no degradation from previous phases)
- Only **paths and summaries** pass between phases (not full content)
- **90.2% improvement** over single-agent (`research/056-multi-agent-orchestrator.md`)

**Skills using this pattern:**
- `calibrate` → Coverage, Drift, Challenge, Synthesize, Report (5 subagents)

**Structure:**
```
.claude/skills/{skill-name}/
├── SKILL.md              # Orchestrator definition
├── agents/               # Subagent definitions
│   ├── phase1.md
│   ├── phase2.md
│   └── ...
└── templates/            # Optional output templates
```

See `docs/guides/ORCHESTRATOR_SUBAGENT_PATTERN.md` for full documentation.

---

---

## Slash Commands: User Actions vs Agent Operations

Each slash command triggers a skill. Here's exactly what **you** do and what **the agent** does.

---

### `/prime` ,  Session Startup

**Skill:** `.claude/skills/prime/SKILL.md`

#### What YOU Do

1. Type `/prime` (or "startup", "initialize")
2. Wait for the agent to complete startup
3. Review the summary the agent produces
4. Approve or modify the suggested task claim

#### What the AGENT Does (Behind the Scenes)

```
Step 1: ANNOUNCE
├── Sets terminal title with agent identity
└── Prints startup banner

Step 2: REGISTER (Agent Mail MCP calls)
├── ensure_project(human_key="/path/to/project")
├── register_agent(program="claude-code", model="opus-4.5")
│   └── Gets assigned name like "BlueLake"
└── set_contact_policy(policy="open")

Step 3: ORIENT
├── Reads AGENTS.md for project context
└── Runs: cass search "project" --days 7 --limit 5

Step 4: COORDINATE (Agent Mail MCP calls)
├── fetch_inbox(agent_name, limit=20)
├── Reads resource://agents/{project} to discover other agents
└── Sends greeting message to active agents

Step 5: DISCOVER
├── git status + git log --oneline -5
├── bd ready --json
└── bv --robot-triage

Step 6: CLAIM (if you approve)
├── bd update <id> --status in_progress --assignee BlueLake
├── file_reservation_paths(paths=[...], exclusive=true)
└── send_message(subject="[CLAIMED] <id>")

Step 7: SUMMARIZE
└── Outputs formatted startup summary
```

#### Output You'll See

```markdown
## Agent Initialized

**Name:** BlueLake
**Focus:** Authentication refactor
**Repo:** my-project

### Context
- Recent sessions: [summary from CASS]
- Inbox: 3 messages, 1 requiring response
- Active agents: GreenCastle, RedStone

### Task Claimed
- **ID:** bd-123
- **Title:** Implement JWT validation
- **Files reserved:** src/auth/**

### Ready to Work
[Agent's plan for the task]
```

---

### `/next-bead` ,  Find and Claim Work

**Skill:** `.claude/skills/next-bead/SKILL.md`
**Also uses:** `.claude/skills/bead-workflow/SKILL.md`

#### What YOU Do

1. Type `/next-bead` (or "next task", "what should I work on")
2. If you have in-progress work, confirm whether to close it
3. Review the agent's task recommendation
4. Approve the claim or request a different task

#### What the AGENT Does (Behind the Scenes)

```
Step 1: CLOSE OUT (if applicable)
├── bd list --status in_progress --json
├── If work exists:
│   ├── Runs tests + ubs --staged
│   ├── Commits (includes .beads/issues.jsonl)
│   ├── bd close <id>.1, <id>.2, ... (sub-beads first)
│   ├── bd close <id> (parent last)
│   ├── release_file_reservations()
│   └── send_message(subject="[CLOSED] <id>")

Step 2: DISCOVER
├── bv --robot-triage (recommendations)
├── bv --robot-plan (execution order)
├── bd ready --json (available tasks)
├── resource://agents/{project} (active agents)
├── bd list --status in_progress (already claimed)
└── fetch_inbox() (check for blocking messages)

Step 3: VERIFY (before claiming)
├── Task status == ready?
├── No other agent claimed it?
├── No file reservation conflicts?
├── Dependencies satisfied?
└── No blocking inbox messages?
    If ANY fails → picks different task or asks you

Step 4: CLAIM
├── bd update <id> --status in_progress --assignee BlueLake
├── bd update <id>.1 --status in_progress --assignee BlueLake
├── bd update <id>.2 --status in_progress --assignee BlueLake
│   └── (claims ALL sub-beads together)
├── file_reservation_paths(paths=[...], exclusive=true, ttl=3600)
└── send_message(subject="[CLAIMED] <id> - Title", to=[all_agents])
```

#### Output You'll See

```markdown
## Previous Work Closed
- **bd-122**: Completed, tests passing, reservations released

## Next Task Claimed
- **ID:** bd-123
- **Title:** Add user validation
- **Priority:** P0
- **Sub-beads:** .1 (tests), .2 (implementation), .3 (docs)
- **Files reserved:** src/validation/**, tests/test_validation.py
- **Dependencies:** bd-120 (satisfied), bd-121 (satisfied)

## Ready to Work
[Agent reads bead description and states plan]
```

---

### `/execute` ,  Parallel Execution

**Skill:** `.claude/skills/execute/SKILL.md`

#### What YOU Do

1. Type `/execute` (or "execute", "run the beads", "start working")
2. Review the discovered beads and computed tracks
3. Watch progress as parallel agents work
4. Review calibration results at phase boundaries
5. Handle any blocked beads that need attention

#### What the AGENT Does (Behind the Scenes)

```
Step 1: DISCOVERY
├── bd ready --json (find ready beads)
├── bv --robot-triage (graph analysis)
└── Identify phases from dependency structure

Step 2: TRACK COMPUTATION
├── Extract file scope from each bead
├── Group beads with non-overlapping files
└── Each group = one parallel track

Step 3: SPAWN WORKERS
├── For each track, spawn worker agent (Task tool, background)
├── Each worker gets: track name, bead IDs, file scope
└── Workers run /prime → /next-bead loop

Step 4: MONITOR
├── Watch Agent Mail for [TRACK COMPLETE], [BLOCKED]
├── Update progress display
└── Handle issues as they arise

Step 5: PHASE BOUNDARY
├── All tracks complete → run /calibrate
├── Calibration passed → spawn next phase workers
└── Calibration failed → create fix beads, re-run

Step 6: COMPLETION
├── All phases done
├── Send [EXECUTION COMPLETE]
└── Prompt for /release
```

#### Output You'll See

```markdown
## Execution Progress

**Phase:** 2 of 4
**Tracks:** 4 active

| Track | Agent | Beads | Progress |
|-------|-------|-------|----------|
| A: Auth | BlueLake | bd-101, bd-102 | 2/2 ✓ |
| B: API | GreenCastle | bd-103, bd-104 | 1/2 |
| C: Models | RedStone | bd-105 | 0/1 |

### Next
Waiting for Tracks B, C...
Then: Calibration → Phase 3
```

#### Options

| Flag | Description |
|------|-------------|
| `--dry-run` | Show plan without spawning agents |
| `--agents N` | Limit parallel agents |
| `--phase N` | Execute specific phase only |

---

### `/decompose-task` ,  Break Down Work

**Skill:** `.claude/skills/decompose-task/SKILL.md`

#### What YOU Do

1. Type `/decompose-task [phase_name]`
2. Point the agent to your phase document (or it finds it)
3. Review the proposed bead structure
4. Approve, modify, or request changes
5. Confirm the dependency graph makes sense

#### What the AGENT Does (Behind the Scenes)

```
Step 1: ANALYZE PHASE
├── Reads phase document
├── Extracts requirements (REQ-*, AC-*)
├── Identifies edit locus (files/directories)
└── Notes verification criteria

Step 2: CREATE CONTENT MANIFEST
├── Lists every detail that must be captured
├── Maps REQ/AC to proposed beads
└── Ensures nothing is "summarized away" (lossless)

Step 3: CREATE BEADS
├── Creates parent bead with:
│   ├── North Star reference
│   ├── REQ/AC links
│   ├── Edit locus
│   └── Verification plan
├── Creates sub-beads with:
│   ├── TESTS FIRST (TDD requirement)
│   ├── Implementation spec
│   └── Handoff safety info
└── Uses: bd create, bd dep add

Step 4: SET DEPENDENCIES
├── bd dep add <child> <blocker> --type blocks
├── Schema/migrations → implementation
├── Implementation → tests
└── Phase completion → calibration bead

Step 5: VALIDATE
├── bv --robot-suggest (missing deps?)
├── bv --robot-plan (sane order?)
└── bv --robot-alerts (risks?)
```

#### Key Rule

**LOSSLESS decomposition:** Every detail from the phase must appear in a bead. If something would be "summarized," it must become explicit text in a bead description.

---

### `/calibrate` ,  Hard Stop and Realign

**Skill:** `.claude/skills/calibrate/SKILL.md`
**Pattern:** Orchestrator-Subagent (5 phases, each with fresh context)

#### What YOU Do

1. Type `/calibrate [phase_or_milestone]`
2. Review the calibration summary the agent produces
3. If agents disagree, review the test results (not their arguments)
4. Make decisions on any unresolved value tradeoffs
5. Approve plan updates before next phase begins

#### What the AGENT Does (Behind the Scenes)

The orchestrator spawns 5 subagents in sequence, each with fresh context:

```
ORCHESTRATOR (manages session, TodoWrite, passes summaries)
        │
        ├──► Phase 1: Coverage Agent (agents/coverage.md)
        │    └── Checks REQ/AC → beads → tests coverage
        │    └── Output: 01_coverage_report.md + gaps_summary
        │
        ├──► Phase 2: Drift Agent (agents/drift.md)
        │    └── Compares implementation to North Star
        │    └── Output: 02_drift_report.md + drift_items
        │
        ├──► Phase 3: Challenge Agent (agents/challenge.md)
        │    └── Writes discriminating tests for disputes
        │    └── Output: 03_challenge_report.md + test_results
        │
        ├──► Phase 4: Synthesize Agent (agents/synthesize.md)
        │    └── Creates falsifiable decisions, preserves dissent
        │    └── Output: 04_synthesis_report.md + decisions
        │
        └──► Phase 5: Report Agent (agents/report.md)
             └── Produces user-facing summary
             └── Output: 05_user_report.md → displayed to you
```

**Key:** Each subagent gets ~500 tokens of input (paths + summaries), not the full 3000+ token context. This prevents "lost in middle" degradation.

#### Key Insight

**This is NOT a debate.** When agents disagree, the Challenge Agent writes discriminating tests. Test results adjudicate, rhetoric does not.

---

### `/resolve` ,  Disagreement Resolution

**Skill:** `.claude/skills/disagreement-resolution/SKILL.md`
**Pattern:** Orchestrator-Subagent (4 phases, test-based adjudication)

#### What YOU Do

1. Type `/resolve` when agents disagree or multiple approaches exist
2. Let the agent gather positions and generate discriminating tests
3. Review test results (not arguments)
4. If tests don't discriminate, you make the value tradeoff decision

#### What the AGENT Does (Behind the Scenes)

The orchestrator spawns 4 subagents:

```
ORCHESTRATOR (manages session, passes test results between phases)
        │
        ├──► Phase 1: Positions Agent (agents/positions.md)
        │    └── Gathers all positions neutrally
        │    └── Output: positions A, B, C with rationale
        │
        ├──► Phase 2: Tests Agent (agents/tests.md)
        │    └── Writes discriminating tests
        │    └── Each test should PASS one position, FAIL another
        │
        ├──► Phase 3: Execute Agent (agents/execute.md)
        │    └── Runs tests against each position
        │    └── Output: results matrix (A: PASS/FAIL, B: PASS/FAIL)
        │
        └──► Phase 4: Adjudicate Agent (agents/adjudicate.md)
             └── Declares winner based on test results
             └── Or preserves dissent if tests don't discriminate
```

#### Example Flow

```
Question: "JWT or session tokens for auth?"

Positions:
  A: JWT (stateless, scalable)
  B: Session tokens (revocable, simpler)

Discriminating Tests:
  T1: test_immediate_revocation   → A: FAIL, B: PASS
  T2: test_horizontal_scaling     → A: PASS, B: FAIL
  T3: test_offline_validation     → A: PASS, B: FAIL

Result: A wins 2-1 → Decision: Use JWT
Preserved dissent: "Revocation concern valid, use short expiry"
```

#### Key Insight

**Max 2 discussion rounds without tests.** If agents can't produce discriminating tests, escalate to user for value decision.

---

### `/ground` ,  Verify External Claims

**Skill:** Uses `.claude/skills/warp-grep/SKILL.md` and `.claude/skills/exa-search/SKILL.md`

#### What YOU Do

1. Type `/ground` when the agent makes claims about APIs, libraries, or patterns
2. Or: Agent invokes grounding automatically before architecture decisions
3. Review what was verified vs. what remains an assumption

#### What the AGENT Does (Behind the Scenes)

```
Step 1: IDENTIFY CLAIMS
├── Parses current context for external claims:
│   ├── API signatures
│   ├── Library usage patterns
│   ├── Framework conventions
│   └── Security best practices

Step 2: GROUND IN CODEBASE (Warp-Grep MCP)
├── Searches existing codebase for:
│   ├── How similar patterns are used
│   ├── Existing conventions
│   └── Related implementations
└── Returns low-token, high-relevance snippets

Step 3: GROUND IN WEB (Exa MCP)
├── Searches current documentation for:
│   ├── Official API signatures
│   ├── Latest library versions
│   └── Current best practices
└── Prioritizes official docs over blog posts

Step 4: MARK ASSUMPTIONS
├── Verified claims → proceed with confidence
└── Unverified claims → marked as assumptions
    └── User decides whether to accept or investigate
```

#### Why This Matters

**API hallucinations are common.** The agent confidently uses APIs that don't exist or have changed. Grounding catches this before it becomes debugging time.

---

## The Agent Mail System

**Skill:** `.claude/skills/agent-mail/SKILL.md`

Agent Mail is the coordination layer that prevents multi-agent chaos. **You don't interact with it directly**, the agent uses it behind the scenes when you run slash commands.

### What It Provides

| Feature | Purpose | Agent Uses It For |
|---------|---------|-------------------|
| **Registration** | Identity | Getting a unique name (e.g., "BlueLake") |
| **Messaging** | Communication | [CLAIMED], [CLOSED], coordination |
| **File Reservations** | Conflict prevention | Locking files before editing |
| **Discovery** | Awareness | Finding active agents |

### What YOU See vs What AGENT Does

#### Registration (happens during `/prime`)

**You see:** "Agent registered as BlueLake"

**Agent does:**
```python
ensure_project(human_key="/abs/path/to/project")
register_agent(project_key="...", program="claude-code", model="opus-4.5")
# → Returns name like "BlueLake"
set_contact_policy(agent_name="BlueLake", policy="open")
```

#### File Reservations (happens during `/next-bead`)

**You see:** "Files reserved: src/auth/**"

**Agent does:**
```python
file_reservation_paths(
    project_key="/abs/path",
    agent_name="BlueLake",
    paths=["src/auth/**", "tests/test_auth.py"],
    ttl_seconds=3600,      # 1 hour
    exclusive=True,        # No one else can edit
    reason="bd-123: implementing JWT validation"
)
```

**If conflict, you see:** "Conflict: GreenCastle has src/auth/** reserved. Pick different task?"

**Agent detected:**
```python
# Response: {"conflicts": [{"path": "src/auth/**", "holder": "GreenCastle"}]}
```

#### Messaging (happens automatically)

**You see:** Agent announces "[CLAIMED] bd-123" and "[CLOSED] bd-123"

**Agent does:**
```python
# When claiming:
send_message(
    sender_name="BlueLake",
    to=["GreenCastle", "RedStone"],
    subject="[CLAIMED] bd-123 - JWT Validation",
    body_md="Starting work on **bd-123**.\n\nFiles reserved: `src/auth/**`",
    thread_id="bd-123"
)

# When closing:
send_message(
    subject="[CLOSED] bd-123 - JWT Validation",
    body_md="Completed **bd-123**.\n\nTests: passing\nReservations: released",
    thread_id="bd-123"
)
```

### When YOU Need to Intervene

| Situation | What You See | What to Do |
|-----------|--------------|------------|
| File conflict | "GreenCastle has file reserved" | Pick different task or wait |
| Inbox message | "Message from GreenCastle: Need help with X" | Respond or acknowledge |
| Agent blocked | "Waiting for reply from RedStone" | Check if RedStone is active |

---

## Multi-Agent Execution: What's Happening

**Skills involved:** `prime`, `next-bead`, `bead-workflow`, `agent-mail`

When multiple agents run in parallel, all coordination happens automatically. Here's what's happening.

### Parallel Execution Diagram (What Agents Do)

```
Time →

Agent A (BlueLake)        Agent B (GreenCastle)      Agent C (RedStone)
        │                         │                         │
        │ /prime                  │ /prime                  │ /prime
        ▼                         ▼                         ▼
   Register ────────────────► Register ────────────────► Register
        │                         │                         │
        │ Check inbox             │ Check inbox             │ Check inbox
        ▼                         ▼                         ▼
   No messages               See A is active            See A, B active
        │                         │                         │
        │ bd ready                │ bd ready                │ bd ready
        ▼                         ▼                         ▼
   [bd-1, bd-2, bd-3]        [bd-1, bd-2, bd-3]        [bd-1, bd-2, bd-3]
        │                         │                         │
        │ Claim bd-1              │ See bd-1 claimed        │ See bd-1,2 claimed
        ▼                         ▼                         ▼
   Reserve src/auth/**       Claim bd-2                Claim bd-3
        │                    Reserve src/users/**      Reserve src/api/**
        │                         │                         │
   [CLAIMED] bd-1 ─────────► Inbox: bd-1 claimed       Inbox: bd-1,2 claimed
        │                         │                         │
        │ Work...                 │ Work...                 │ Work...
        ▼                         ▼                         ▼
   Tests pass                Tests pass                Tests pass
   ubs clean                 ubs clean                 ubs clean
        │                         │                         │
   [CLOSED] bd-1 ──────────► Inbox: bd-1 closed        Inbox: bd-1 closed
        │                         │                         │
        │ /next-bead              │                         │
        ▼                         │                         │
   Claim bd-4...                  │                         │
```

### What Prevents Conflicts

| Mechanism | How It Works |
|-----------|--------------|
| **File reservations** | Agent must reserve files before editing; conflicts reported |
| **[CLAIMED] messages** | All agents see what's been claimed |
| **Task status** | `bd update --status in_progress` marks task as taken |
| **Assignee tracking** | `--assignee BlueLake` records ownership |
| **Inbox checking** | `/prime` and `/next-bead` check for conflicts first |

### When Conflicts Occur

If Agent B tries to reserve files held by Agent A:

```python
# Agent B tries:
file_reservation_paths(
    paths=["src/auth/**"],
    agent_name="GreenCastle"
)

# Response:
{
    "granted": [],
    "conflicts": [{
        "path": "src/auth/**",
        "holders": ["BlueLake"],
        "expires_ts": "2025-12-22T21:00:00Z"
    }]
}
```

**Resolution options:**
1. Wait for BlueLake to finish (check `expires_ts`)
2. Message BlueLake to coordinate
3. Pick a different task with no conflicts

---

## The Bead Lifecycle

**Skill:** `.claude/skills/bead-workflow/SKILL.md`

### What YOU Do at Each Phase

| Phase | Your Actions |
|-------|--------------|
| **Claim** | Approve task selection, confirm file scope |
| **Work** | Review implementation, provide clarification if asked |
| **Close** | Confirm tests pass, approve commit message |

### Full Flow: What the Agent Does

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLAIM PHASE                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. CHECK                    2. CLAIM                   3. RESERVE          │
│  ─────────────               ──────────                 ─────────           │
│  • Check inbox               • bd update <id>           • Reserve files     │
│  • Check reservations          --status in_progress     • Set TTL           │
│  • Check dependencies          --assignee NAME          • Exclusive lock    │
│  • bv --robot-triage         • Claim ALL sub-beads                          │
│                                                                              │
│  4. ANNOUNCE                                                                 │
│  ──────────────                                                              │
│  • send_message [CLAIMED]                                                    │
│  • thread_id = bead ID                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              WORK PHASE                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. READ TESTS               2. IMPLEMENT              3. RUN TESTS         │
│  ──────────────              ─────────────             ──────────           │
│  • Tests in bead desc        • Minimal code to         • pytest / npm test  │
│  • TDD-first always            pass tests              • Expect pass        │
│                                                                              │
│  4. REPAIR (if needed)       5. SECURITY GATE                               │
│  ─────────────────────       ────────────────                               │
│  • Max 3 iterations          • ubs --staged                                 │
│  • After 3: STOP             • Zero high/critical                           │
│  • Spawn sub-bead or spike   • Medium needs justification                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLOSE PHASE                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. VERIFY                   2. COMMIT                  3. CLOSE            │
│  ─────────                   ────────                   ───────             │
│  • Tests passing             • git add -A               • Close sub-beads   │
│  • ubs --staged clean        • Include .beads/            FIRST             │
│                              • git commit               • Then close parent │
│                                                                              │
│  4. RELEASE                  5. ANNOUNCE                                     │
│  ─────────                   ──────────                                      │
│  • Release file              • send_message [CLOSED]                        │
│    reservations              • thread_id = bead ID                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Commands at Each Stage

```bash
# === CLAIM ===
bd ready --json                                    # See what's available
bv --robot-next                                    # Get recommendation
bd update <id> --status in_progress --assignee ME  # Claim it
# + file_reservation_paths() via MCP
# + send_message([CLAIMED]) via MCP

# === WORK ===
# Read tests from bead description
pytest                                             # Run tests (expect fail)
# Implement
pytest                                             # Run tests (expect pass)
ubs --staged                                       # Security scan

# === CLOSE ===
git add -A && git commit -m "..."                  # Commit
bd close <id>.1 --reason "Completed: ..."          # Close sub-beads first
bd close <id> --reason "Completed: ..."            # Then parent
# + release_file_reservations() via MCP
# + send_message([CLOSED]) via MCP
```

---

## Security Gate: `ubs --staged`

**Skill:** `.claude/skills/ubs-scanner/SKILL.md`

### What YOU Do

1. **Nothing during normal flow** ,  Agent runs `ubs --staged` automatically before commits
2. **If findings reported:** Review severity and decide whether to fix or document
3. **CRITICAL/HIGH findings:** Must approve the fix before proceeding

### What the AGENT Does

The agent runs `ubs --staged` automatically:
- Before any commit
- Before closing any bead
- After each repair iteration
- Before any PR

### Why It's Mandatory

~40% of LLM-generated code contains vulnerabilities. This gate catches them before commit.

### What You'll See

```bash
# Clean output (good):
✓ No security issues found in staged files

# With findings (agent will show you):
⚠ HIGH: Potential SQL injection in src/db/query.ts:42
  - User input directly interpolated into query
  - Agent proposes: Use parameterized queries

⚠ MEDIUM: Hardcoded API key in src/config.ts:15
  - Agent proposes: Move to environment variable
```

### Your Decision Points

| Severity | Agent Does | You Decide |
|----------|------------|------------|
| **CRITICAL/HIGH** | Stops and proposes fix | Approve fix (counts toward 3-iteration cap) |
| **MEDIUM** | Proposes fix or justification | Accept fix OR approve documented justification |
| **LOW** | Notes in output | Acknowledge or ignore |

---

## Calibration: Behind the Scenes

**Skill:** `.claude/skills/calibrate/SKILL.md`
**Pattern:** Orchestrator-Subagent

This section summarizes what `/calibrate` does. See the slash command section above for the full user/agent breakdown.

### Summary: What Happens

The orchestrator spawns 5 subagents sequentially:

| Phase | Subagent | Agent Does | You Do |
|-------|----------|------------|--------|
| 1 | Coverage | Checks REQ/AC → beads → tests | Wait |
| 2 | Drift | Compares implementation to North Star | Wait |
| 3 | Challenge | Writes discriminating tests for disputes | Review test results |
| 4 | Synthesize | Produces decisions, preserves dissent | Approve plan changes |
| 5 | Report | Creates user-facing summary | Read and decide |

### Why Subagents

| Monolithic Calibration | Subagent Pattern |
|------------------------|------------------|
| All 5 phases in one context | Each phase gets fresh 200k context |
| ~3000 token prompt | ~500 tokens per phase |
| "Lost in middle" risk | No degradation |
| One phase failure corrupts all | Phases are isolated |

### Key Point

**Tests adjudicate disagreements, not debate.** If the Coverage and Drift agents disagree, the Challenge Agent writes tests that would break each approach. Test results decide the winner. You only intervene if tests don't discriminate (value tradeoff).

---

## Quick Reference: What You Type vs What Agent Does

### Your Commands (What You Type)

| Command | What Happens |
|---------|--------------|
| `/prime` | Agent registers, orients, discovers tasks, recommends claim |
| `/next-bead` | Agent closes current work, finds next task, claims it |
| `/decompose-task [phase]` | Agent breaks phase into beads with tests |
| `/calibrate` | Agent runs hard stop, collects analyses, resolves disagreements |
| `/ground` | Agent verifies external claims against codebase/web |

### What Agent Does Automatically (You Don't Type These)

| Trigger | Agent Does |
|---------|------------|
| Claiming task | Reserves files, sends [CLAIMED] message |
| Before commit | Runs `ubs --staged` security scan |
| Closing task | Releases reservations, sends [CLOSED] message |
| Agents disagree | Writes discriminating tests, runs them |
| Needs coordination | Sends/reads Agent Mail messages |

### Manual Commands (If You Need Them)

```bash
# Task discovery (agent does this in /next-bead)
bd ready --json                  # Available tasks
bv --robot-triage                # Recommendations
bv --robot-next                  # Single best task

# Claiming (agent does this in /next-bead)
bd update <id> --status in_progress --assignee YOUR_NAME
bd update <id>.1 --status in_progress --assignee YOUR_NAME

# Closing (agent does this in /next-bead)
bd close <id>.1 --reason "..."   # Sub-beads FIRST
bd close <id> --reason "..."     # Then parent

# Security (agent runs automatically)
ubs --staged                     # Manual security check
```

### Between Phases
```bash
/calibrate                       # Hard stop and realign
```

---

## Troubleshooting

| Problem | What You See | What to Do |
|---------|--------------|------------|
| "from_agent not registered" | Agent can't send messages | Type `/prime` to register |
| File reservation conflict | "GreenCastle has files reserved" | Pick different task or wait |
| Task already claimed | "bd-123 is in_progress" | Ask agent to pick different task |
| `ubs` finding blocks commit | "HIGH: SQL injection..." | Approve agent's proposed fix |
| Tests won't pass after 3 tries | "Exceeded 3 repair iterations" | Agent will spawn spike bead |
| Agent seems stuck | No progress | Check if waiting for your input |
| Multiple agents conflict | "Merge conflict detected" | One agent should back off |

---

## See Also

- `START_HERE.md` ,  System overview
- `PROTOCOLS.md` ,  Protocol reference
- `EVIDENCE_BASED_GUIDE.md` ,  Full pipeline
- `.claude/skills/` ,  Individual skill documentation
