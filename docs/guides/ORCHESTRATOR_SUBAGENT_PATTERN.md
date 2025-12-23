# Orchestrator-Subagent Pattern

## TLDR

Instead of one agent holding everything in context, spawn specialized subagents for each phase. Each gets fresh context. Pass only **summaries and file paths** between phases, not full content.

**Result:** 90% improvement over single-agent approaches.

---

## What This Is

This is the **optimal pattern for complex multi-phase agentic workflows**. Instead of one agent trying to hold everything in context, an orchestrator spawns specialized subagents for each phase, with each subagent getting its own fresh context window.

**Why this matters:**
- Orchestrator-worker patterns beat single-agent by **90.2%** (`research/056-multi-agent-orchestrator.md`)
- Context degradation is **30-50%** as context grows (`research/004-context-length-hurts.md`)
- Each subagent gets **fresh context** = no "lost in middle" problem

---

## The Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR (Main Skill)                     │
│  - Creates session directory for artifacts                       │
│  - Manages TodoWrite state (progress tracking)                   │
│  - Spawns subagents with minimal, targeted context               │
│  - Collects and validates reports                                │
│  - Passes outputs between phases                                 │
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

---

## Skill Structure

```
.claude/skills/{skill-name}/
├── SKILL.md              # Orchestrator definition
├── agents/               # Subagent definitions
│   ├── phase1.md         # Phase 1 subagent prompt
│   ├── phase2.md         # Phase 2 subagent prompt
│   ├── phase3.md         # Phase 3 subagent prompt
│   └── ...
└── templates/            # Optional output templates
    └── report.md
```

---

## How It Works

### 1. Orchestrator Creates Session

```markdown
# In SKILL.md orchestrator section:

1. Create session directory: `sessions/{timestamp}/`
2. Initialize TodoWrite with phase tasks
3. Store inputs for subagent access
```

### 2. Orchestrator Spawns Subagent

The orchestrator uses the Task tool to spawn each subagent:

```markdown
Spawn Phase 1 agent with:
- Input: {extraction_type, session_dir, SKILL_DIR}
- Agent file: agents/phase1.md
- Expected output: report_path + architecture_summary
```

### 3. Subagent Executes with Fresh Context

Each subagent:
- Gets **only** the context it needs (not the whole conversation)
- Writes artifacts to disk (reports, code, etc.)
- Returns a summary to the orchestrator

### 4. Orchestrator Passes Context Forward

```
Phase 1 output: report_path + architecture_summary
         │
         ▼
Phase 2 input: {inputs + phase1_summary}
         │
         ▼
Phase 2 output: report_path + structure_summary
         │
         ▼
Phase 3 input: {inputs + phase2_report_path}
```

**Key insight:** Only pass **summaries and file paths**, not full content. The next subagent can read files if needed.

---

## Why This Beats Single-Agent

| Single Agent | Orchestrator-Subagent |
|--------------|----------------------|
| Context grows with each phase | Each phase gets fresh context |
| 30-50% degradation over time | No degradation |
| One failure can corrupt everything | Phases are isolated |
| Hard to parallelize | Can run independent phases in parallel |
| No natural checkpoints | Each phase is a checkpoint |

**Research backing:**
- `research/056-multi-agent-orchestrator.md`: +90.2% over single-agent
- `research/004-context-length-hurts.md`: Context degradation is real
- `research/057-anthropic-context-engineering.md`: "Smallest set of high-signal tokens"

---

## Example: Calibrate Skill with Subagents

### Orchestrator (`SKILL.md`)

```markdown
---
name: calibrate
description: Phase boundary calibration with subagent phases
---

# Calibrate - Orchestrator

## Execution Flow

1. **Setup**
   - Create session: `sessions/calibrate-{timestamp}/`
   - Initialize TodoWrite with phases
   - Gather inputs (phase name, beads, requirements)

2. **Phase 1: Coverage Analysis** (spawn agents/coverage.md)
   - Input: {phase_name, requirements, beads}
   - Output: coverage_report_path + gaps_summary

3. **Phase 2: Drift Detection** (spawn agents/drift.md)
   - Input: {phase_name, north_star, coverage_report_path}
   - Output: drift_report_path + drift_summary

4. **Phase 3: Risk Assessment** (spawn agents/risk.md)
   - Input: {drift_report_path, gaps_summary}
   - Output: risk_report_path + action_items

5. **Phase 4: Test Adjudication** (spawn agents/adjudicate.md)
   - Input: {action_items, any_disagreements}
   - Output: decisions_path + test_results

6. **Phase 5: Synthesis** (spawn agents/synthesize.md)
   - Input: {all_report_paths, decisions_path}
   - Output: calibration_memo_path + plan_updates

7. **Finalize**
   - Update TodoWrite (all complete)
   - Output calibration memo to user
```

### Subagent (`agents/coverage.md`)

```markdown
---
name: calibrate-coverage
description: Coverage analysis subagent for calibration
---

# Coverage Analysis Subagent

You are a coverage analysis agent. Your job is to check requirement coverage.

## Inputs (provided by orchestrator)
- phase_name: The phase being calibrated
- requirements: REQ-* and AC-* items
- beads: Completed beads for this phase

## Task

1. For each REQ-*, check:
   - Is there a bead that implements it?
   - Is there a test that verifies it?
   - Is there evidence of acceptance?

2. Identify gaps:
   - REQ with no beads
   - REQ with no tests
   - AC with no evidence

3. Write report to: `{session_dir}/coverage_report.md`

## Output Format

Return to orchestrator:
```json
{
  "report_path": "{session_dir}/coverage_report.md",
  "gaps_summary": "3 REQs missing tests, 1 REQ missing bead"
}
```

## Constraints
- Do NOT implement fixes
- Do NOT read files outside the phase scope
- Do NOT debate findings—just report facts
```

---

## Context Optimization Strategies

### 1. Pass Summaries, Not Content

```markdown
# BAD: Passing full content
Phase 2 input: {full_coverage_report_content}  # 2000 tokens

# GOOD: Passing path + summary
Phase 2 input: {
  coverage_report_path: "sessions/.../coverage.md",
  gaps_summary: "3 REQs missing tests"  # 10 tokens
}
```

### 2. Let Subagents Read What They Need

Subagents can read files on demand. Don't pre-load everything.

### 3. Write Artifacts to Disk

Every subagent should write its work to disk:
- Reports go in `sessions/{id}/`
- Code goes in the codebase
- Summaries returned to orchestrator

### 4. Use TodoWrite for Progress

The orchestrator maintains TodoWrite state:
```markdown
- [completed] Phase 1: Coverage Analysis
- [in_progress] Phase 2: Drift Detection
- [pending] Phase 3: Risk Assessment
```

---

## When to Use This Pattern

| Use Orchestrator-Subagent | Use Single Agent |
|---------------------------|------------------|
| 3+ distinct phases | Simple, single-phase task |
| Each phase >500 lines of context | Total context <1000 lines |
| Phases can fail independently | All-or-nothing execution |
| Need checkpoints/recovery | No recovery needed |
| Complex multi-step workflows | Quick one-shot tasks |

---

## Integration with Knowledge & Vibes

This pattern maps directly to our workflow:

| Pipeline Stage | Orchestrator Use |
|----------------|------------------|
| `/decompose-task` | Subagents for: analyze, manifest, create beads, validate |
| `/calibrate` | Subagents for: coverage, drift, risk, adjudicate, synthesize |
| `/prime` | Could use subagents for: register, orient, discover, claim |
| Bead execution | Subagents for: TDD setup, implement, verify, close |

---

## Example: Wrapping Entire Skill in Subagent

For maximum context savings, the orchestrator can be minimal:

```markdown
# SKILL.md (minimal orchestrator)

1. Create session directory
2. Spawn main subagent with full skill logic
3. Collect results
4. Return to user

# agents/main.md (full skill logic)
[All the actual work happens here with fresh context]
```

**Tradeoff:** Less visibility into progress. Use TodoWrite in the subagent to maintain visibility.

---

## See Also

- `research/056-multi-agent-orchestrator.md` — Research on orchestrator patterns
- `research/004-context-length-hurts.md` — Context degradation evidence
- `docs/guides/TOOL_STACK_AND_OPERATIONS.md` — Tool stack overview
- `.claude/skills/calibrate/SKILL.md` — Example skill (can be refactored)
