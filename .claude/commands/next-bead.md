---
description: Find and safely claim the next Beads task (conflict-checked)
argument-hint: [focus_area]
---

# Next Bead - Find and Claim Your Next Task

After completing a task, use this command to find the logically next available work while avoiding conflicts with other agents.

## Usage

```
/next-bead [focus_area]
```

## Arguments

- `$ARGUMENTS` - Optional. Focus area or preference (e.g., "api", "database", "tests"). If omitted, will show all available work.

---

## Configuration

**Before using, set these variables for your project:**

```
PROJECT_PATH="/abs/path/to/your/project"  # Agent Mail project_key / coordination root
PROJECT_NAME="Your Project Name"
```

If you’re working across multiple repos, keep `PROJECT_PATH` pointed at ONE canonical coordination root (the place you track Beads + Agent Mail), even if the code change is in another repo.

---

## Philosophy

**Parallel agents must coordinate to avoid stepping on each other.**

Before claiming work:
1. Know what's truly available (not just "ready")
2. Check what other agents are doing
3. Verify no file conflicts exist
4. Communicate intent before grabbing work

This command enforces rigorous verification to prevent wasted effort and merge conflicts.

---

## Instructions

**Complete ALL steps in order. Do not skip any.**

---

### Step 1: Re-Read AGENTS.md

Refresh your understanding of the workflow:

```
Read("AGENTS.md")
```

Note the key rules you must follow:
- File reservation requirements
- Cross-repo boundaries
- Communication expectations

---

### Step 2: Close Out Previous Work (If Applicable)

If you just finished a task, ensure it's properly closed:

```bash
bd list --status in_progress --json
```

If you have an in_progress task:
1. Did you run tests + `ubs --staged` (as appropriate)?
2. Did you commit/push your changes (including `.beads/issues.jsonl`)?
3. Close ALL sub-beads first, then the parent:
   ```bash
   bd close <task-id>.1 --reason "Completed: [summary]"
   bd close <task-id>.2 --reason "Completed: [summary]"
   # ... repeat for ALL sub-beads
   bd close <task-id> --reason "Completed: [summary]"
   ```
4. Release your file reservations:
   ```
release_file_reservations(
     project_key="PROJECT_PATH",
     agent_name="YOUR_AGENT_NAME"
   )
   ```

**Note:** Beads auto-exports to JSONL. Share via normal git commits (include `.beads/issues.jsonl` with code changes).

If you don't have an in_progress task, proceed to Step 3.

---

### Step 3: Get AI-Prioritized Recommendations

Use bv to understand what's logically next:

```bash
bv --robot-triage
```

This gives you a unified bundle:
- Ranked recommendations (with scores + reasons + unblock info)
- Quick wins + top blockers to clear
- Copy/paste next-step commands

Also check for parallel execution opportunities:

```bash
bv --robot-plan
```

Note which tasks are recommended and their priorities.

If multiple agents are active, group triage output to split work cleanly:

```bash
bv --robot-triage --robot-triage-by-track
# or: bv --robot-triage --robot-triage-by-label
```

---

### Step 4: Check Raw Ready Tasks

Get the full list of unblocked work:

```bash
bd ready --json
```

Cross-reference with bv recommendations. Identify candidates that:
- Match your focus area (`$ARGUMENTS`) if provided
- Are high priority
- Have clear acceptance criteria

---

### Step 5: Discover Active Agents

See who else is working:

```
ReadMcpResourceTool(server="mcp-agent-mail", uri="resource://agents/PROJECT_PATH")
```

Note each agent's:
- Name
- Task description (what they're working on)
- Last active timestamp

---

### Step 6: Check Current File Reservations

**CRITICAL: Do this before claiming any task.**

Check what files are already reserved:

```
call_extended_tool(
  tool_name="file_reservation_status",
  arguments={"project_key": "PROJECT_PATH"}
)
```

Build a mental map:
- Which paths are reserved?
- By whom?
- For what reason?

---

### Step 7: Check In-Progress Beads

See what tasks other agents have claimed:

```bash
bd list --status in_progress --json
```

Cross-reference with active agents. Avoid:
- Tasks already claimed by another agent
- Tasks that touch files reserved by others
- Tasks that depend on in-progress work

---

### Step 8: Check Your Inbox

```
fetch_inbox(
  project_key="PROJECT_PATH",
  agent_name="YOUR_AGENT_NAME",
  include_bodies=true,
  limit=10
)
```

Look for:
- Requests from other agents
- Questions that need answers
- Coordination messages about task allocation

**If there are urgent messages, address them before claiming new work.**

---

### Step 9: Select a Task (With Conflict Verification)

Pick a task from the ready list that:
1. Is NOT in_progress by another agent
2. Does NOT conflict with existing file reservations
3. Matches your focus area (if specified)
4. Has high priority or is logically next per bv

**Verification checklist before claiming:**

| Check | Status |
|-------|--------|
| Task status is `ready` (not `in_progress`) | [ ] |
| No other agent has claimed this task | [ ] |
| Required files are not reserved by others | [ ] |
| Task dependencies are satisfied | [ ] |
| No blocking messages in inbox | [ ] |

If ANY check fails, pick a different task or coordinate first.

---

### Step 10: Coordinate If Uncertain

If you're unsure whether a task is safe to claim, **ask first**:

```
send_message(
  project_key="PROJECT_PATH",
  sender_name="YOUR_AGENT_NAME",
  to=["OtherAgent1", "OtherAgent2"],
  subject="Claiming task: <task-id>",
  body_md="I'm planning to pick up **<task-id>: <title>**.\n\nThis will involve files in:\n- `app/services/...`\n- `tests/...`\n\nAny conflicts or concerns before I claim it?",
  importance="normal"
)
```

Wait for responses if:
- The task touches shared infrastructure
- Multiple tasks seem related
- Another agent might be about to claim it

---

### Step 11: Claim the Task (INCLUDING ALL SUB-BEADS)

Once verified safe, **claim the parent AND all sub-beads together**:

```bash
# CRITICAL: Claim ALL sub-beads to prevent other agents from grabbing them
bd update <task-id> --status in_progress --assignee YOUR_AGENT_NAME
bd update <task-id>.1 --status in_progress --assignee YOUR_AGENT_NAME
bd update <task-id>.2 --status in_progress --assignee YOUR_AGENT_NAME
bd update <task-id>.3 --status in_progress --assignee YOUR_AGENT_NAME
# ... repeat for ALL sub-beads
```

**WHY:** If you only claim the parent, another agent sees sub-beads as "ready" and starts working on them → CONFLICT.

**Important:** Always set yourself as the assignee so other agents know who owns the task.

Reserve your files:

```
file_reservation_paths(
  project_key="PROJECT_PATH",
  agent_name="YOUR_AGENT_NAME",
  paths=["app/path/to/files/**", "tests/relevant/**"],
  ttl_seconds=3600,
  exclusive=true,
  reason="<task-id>: brief description"
)
```

### Step 12: Announce to ALL Agents

**MANDATORY: Send a `[CLAIMED]` message so other agents know this bead is taken:**

```
send_message(
  project_key="PROJECT_PATH",
  sender_name="YOUR_AGENT_NAME",
  to=["all"],  # Or list all active agents
  subject="[CLAIMED] <task-id> - <title>",
  body_md="I am starting work on **<task-id>: <title>** (plus sub-beads .1, .2, .3, etc.).\n\nFile reservations:\n- `app/path/...`\n- `tests/...`\n\nExpected scope: [brief description]",
  importance="normal",
  thread_id="<task-id>"
)
```

This prevents other agents from claiming the same bead or its sub-beads.

---

### Step 13: Output Summary

```
## Next Task Claimed

### Task
**ID:** <task-id>
**Title:** <task title>
**Priority:** <priority>

### Why This Task
- [Explanation of why this was the logical next choice]
- [bv recommendation rank, if applicable]

### Verification Performed
| Check | Result |
|-------|--------|
| Task status | ready -> in_progress |
| Assignee set | YOUR_AGENT_NAME |
| File conflicts | None |
| Agent conflicts | None |
| Dependencies | Satisfied |

### Files Reserved
- `app/path/to/files/**`
- `tests/relevant/**`

### Other Active Agents
- [Agent1]: Working on <their-task>
- [Agent2]: Working on <their-task>

### Ready to Begin
[Brief description of your approach to this task]
```

---

## When to Use This Command

- **After completing a task** - Find your next piece of work
- **After context compaction** - Re-orient and verify task is still valid
- **When starting a session (after /prime)** - If no task was claimed during prime
- **When your current task is blocked** - Find alternative work

## Anti-Patterns

- **Don't skip file reservation checks** - This causes merge conflicts
- **Don't claim tasks another agent is actively discussing** - Communicate first
- **Don't ignore inbox messages** - They may affect task availability
- **Don't hoard tasks** - Claim one at a time, finish it, then claim another
- **Don't claim only the parent bead** - ALWAYS claim ALL sub-beads together
- **Don't skip the `[CLAIMED]` announcement** - Other agents need to know what's taken
- **Don't forget to send `[CLOSED]` when done** - Other agents need to know work is complete

---

## Quick Reference

```bash
# See what's recommended
bv --robot-triage

# Minimal "what next?"
bv --robot-next

# See what's unblocked
bd ready --json

# See what's claimed
bd list --status in_progress --json

# Claim a task AND ALL SUB-BEADS (always set assignee!)
bd update <task-id> --status in_progress --assignee YOUR_AGENT_NAME
bd update <task-id>.1 --status in_progress --assignee YOUR_AGENT_NAME
bd update <task-id>.2 --status in_progress --assignee YOUR_AGENT_NAME
# ... repeat for ALL sub-beads

# Check file reservations
call_extended_tool(tool_name="file_reservation_status", ...)

# Check other agents
ReadMcpResourceTool(server="mcp-agent-mail", uri="resource://agents/PROJECT_PATH")

# Send [CLAIMED] announcement
send_message(..., subject="[CLAIMED] <task-id> - Title", ...)

# When done, send [CLOSED] announcement
send_message(..., subject="[CLOSED] <task-id> - Title", ...)
```

---

## Dependencies

This command assumes you have the following tools configured:
- **bd (beads)** - Issue tracking CLI
- **bv** - AI sidecar for bead prioritization
- **mcp-agent-mail** - MCP server for agent coordination
- **cass** - Session history search (optional but recommended)
