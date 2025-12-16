# knowledge_and_vibes: The Complete Guide

> **For humans**: Read this to understand what these tools do and why they matter.
> **For agents**: See [AGENTS_TEMPLATE.md](./AGENTS_TEMPLATE.md) for distilled commands.

---

## Why This Exists

You're using AI coding agents (Claude, Codex, Cursor, etc.) to write code. That's great. But you've probably noticed some problems:

**The agent forgets what it was doing.** You come back the next day and it has no memory of yesterday's work. You have to re-explain everything.

**You lose track of what's done.** There's no clear record of what was completed, what's in progress, what's blocked.

**Multiple agents step on each other.** You run two agents on the same project and they both edit the same file. Merge conflicts. Lost work.

**Past solutions disappear.** You solved this exact problem last month with a different agent. But that conversation is gone. You start from scratch.

**AI-generated code has bugs.** Unclosed file handles. SQL injection. Missing error handling. You don't catch them until production.

**This toolkit solves all of these problems.**

---

## What Problem Does This Solve?

AI coding agents are powerful, but they have problems:

| Problem | What Happens | Our Solution |
|---------|--------------|--------------|
| **Amnesia** | Agent forgets tasks mid-session | **Beads** - persistent task memory |
| **No visibility** | Can't see dependencies or impact | **Beads Viewer** - graph intelligence |
| **Conflicts** | Multiple agents overwrite each other | **Agent Mail** - file locks + messaging |
| **Lost knowledge** | Past solutions disappear | **CASS** - search all past sessions |
| **No learning** | Same mistakes repeated across sessions | **cass-memory** - distill lessons into rules |
| **Bugs slip through** | AI-generated code has issues | **UBS** - 1000+ bug pattern scanner |
| **Slow search** | Serial search pollutes context | **Warp-Grep** - 8× parallel search |
| **Outdated knowledge** | Agent doesn't know current APIs/docs | **Exa MCP** - AI web & code search |

These tools work together as a complete system. Let's learn how.

---

## Part 1: Understanding the Tools

### Beads (`bd`) - Your Task Memory

**What it is**: A git-backed issue tracker designed for AI agents. Tasks persist across sessions and track dependencies.

**Why you need it**: Without Beads, an agent might forget what it was working on, lose track of discovered bugs, or not know what's ready to work on.

**Key insight**: Beads treats tasks as a dependency graph, not a flat list. Task A might block Task B. When you complete A, B becomes "ready."

**For humans**: Think of it as a todo list that understands "I can't do X until Y is done."

**For agents**: Always start sessions with `bd ready --json` to get unblocked work. End sessions by committing/pushing `.beads/issues.jsonl` (optionally run `bd sync` first to flush immediately).

**Advanced features**:
```bash
# Dependencies
bd dep add bd-child bd-blocker --type blocks      # Task blocked by another
bd dep add bd-new bd-current --type discovered-from  # Found during work
bd dep tree bd-42                                 # Visualize dependency tree
bd blocked                                        # What's waiting on dependencies?

# Maintenance
bd doctor --fix                                   # Health check and auto-repair
bd show bd-42                                     # Full task details
```

### Beads Viewer (`bv`) - Graph Intelligence

**What it is**: Analyzes your task graph to find the highest-impact work.

**Why you need it**: With 50+ tasks, how do you know which one matters most? Beads Viewer uses graph algorithms (PageRank, betweenness centrality) to find critical tasks.

**Key insight**: A task that unblocks 5 other tasks is more valuable than one that unblocks none.

**For humans**: It's like having a project manager who can instantly tell you "this task is blocking everything else."

**For agents**:
```
CRITICAL: Never run `bv` without --robot-* flags!
The interactive TUI will hang your session permanently.
Always use: bv --robot-priority, bv --robot-insights, etc.
```

**All robot commands**:
```bash
bv --robot-priority                # Ranked recommendations with confidence scores
bv --robot-plan                    # Parallel execution tracks
bv --robot-insights                # Graph metrics (PageRank, betweenness, HITS)
bv --robot-recipes                 # Available filter presets
bv --robot-diff --diff-since "1 hour ago"  # What changed recently
bv --robot-diff --diff-since HEAD~5        # Changes in last 5 commits
```

### Agent Mail - Multi-Agent Coordination

**What it is**: A messaging and file-locking system for multiple AI agents working on the same project.

**Why you need it**: If two agents edit the same file simultaneously, you get merge conflicts and lost work. Agent Mail prevents this.

**Key insight**: Before editing files, you "reserve" them. Other agents see your reservation and work elsewhere. When done, you release the reservation.

**For humans**: It's like a whiteboard where developers write "I'm working on auth.ts - don't touch!"

**For agents**:
```python
# Always register first
ensure_project(human_key="/abs/path")
register_agent(project_key="/abs/path", program="claude-code", model="opus-4.5")

# Reserve before editing
file_reservation_paths(project_key, agent_name, paths=["src/auth/**"], exclusive=True)

# Release when done
release_file_reservations(project_key, agent_name)
```

**Advanced features**:
```python
# Extend reservation if work takes longer
renew_file_reservations(project_key, agent_name, extend_seconds=1800)

# Search all messages
search_messages(project_key, query="authentication", limit=20)

# Summarize a thread
summarize_thread(project_key, thread_id="bd-123")

# Build coordination (prevent concurrent builds)
acquire_build_slot(project_key, agent_name, slot="main", exclusive=True)
release_build_slot(project_key, agent_name, slot="main")

# Quick start macro (register + reserve + check inbox in one call)
macro_start_session(
    human_key="/abs/path",
    program="claude-code",
    model="opus-4.5",
    file_reservation_paths=["src/**"],
    inbox_limit=10
)

# Who is this agent?
whois(project_key, agent_name="BlueLake")
```

### CASS - Session Search

**What it is**: Searches across ALL your past AI coding sessions - Claude Code, Codex, Cursor, Aider, Gemini, and more.

**Why you need it**: You (or another agent) might have solved a similar problem last week. Why start from scratch?

**Key insight**: Your past sessions are a knowledge base. CASS makes them searchable.

**For humans**: It's like having a searchable history of every conversation you've had with every AI coding tool.

**For agents**:
```bash
# Search before starting work
cass search "authentication error" --robot --fields minimal --limit 5

# Token-efficient output
cass search "query" --robot --max-tokens 2000

# Wildcard search
cass search "auth*" --robot --limit 5
```

**Additional commands**:
```bash
# View session activity
cass timeline --today --json                       # Today's sessions
cass timeline --since 7d --json                    # Last week

# Export session to markdown
cass export /path/session.jsonl --format markdown -o output.md

# Expand context around a match
cass expand /path/session.jsonl -n 42 -C 3 --json  # Line 42 with 3 lines context

# Output formats
cass search "query" --robot-format jsonl           # Streaming output
cass search "query" --robot-format compact         # Minimal single-line JSON
```

### cass-memory (`cm`) - Cross-Agent Learning

**What it is**: A procedural memory system that learns from your past sessions. It maintains a playbook of rules and patterns, automatically extracting lessons from successful (and failed) approaches.

**Why you need it**: CASS finds raw sessions. cass-memory goes further - it distills those sessions into actionable rules: "When doing X, always do Y" or "Never do Z because it causes W."

**Key insight**: This is the difference between remembering a conversation and learning from it. cass-memory builds a knowledge base that improves over time.

**For humans**: It's like having a team knowledge base that automatically captures best practices from every coding session.

**For agents**:
```bash
# Start EVERY non-trivial task with this
cm context "implement user authentication" --json

# Returns:
# - Relevant rules from the playbook
# - Historical context from similar past work
# - Anti-patterns to avoid
# - Suggested CASS searches for more context

# Health check
cm doctor
```

**What you get back**:
```json
{
  "rules": [
    {"pattern": "authentication", "rule": "Always use bcrypt with cost factor >= 12"},
    {"pattern": "JWT", "rule": "Store refresh tokens in httpOnly cookies, not localStorage"}
  ],
  "history": [...],
  "antipatterns": ["Don't store passwords in plaintext - session abc123 failed"],
  "suggested_searches": ["cass search 'bcrypt implementation' --robot"]
}
```

**Automation note**: You don't need to run separate reflection/learning steps — the system learns automatically from your sessions.

### UBS - Bug Scanner

**What it is**: Static analysis tool that catches bugs in AI-generated code.

**Why you need it**: AI-generated code often has subtle issues - unclosed resources, missing error handling, security vulnerabilities. UBS catches 1000+ patterns.

**Key insight**: Run it before every commit. It's your safety net.

**For humans**: It's like having a code reviewer who never gets tired and knows every common bug pattern.

**For agents**:
```bash
# Before committing
ubs --staged

# If issues found, fix them, then verify
ubs --staged --fail-on-warning   # Exit 0 = safe
```

**Additional options**:
```bash
# Scan working tree changes (not staged)
ubs --diff

# Profiles for different contexts
ubs --profile=strict .             # Fail on all warnings
ubs --profile=loose .              # Skip nits (for prototyping)

# Language filters
ubs --only=typescript .            # TypeScript only
ubs --only=python,go .             # Multiple languages

# Output formats
ubs . --format=json                # JSON output
ubs . --format=sarif               # GitHub Code Scanning format

# CI mode with regression detection
ubs --ci
ubs --comparison baseline.json .   # Compare against baseline

# Verbose mode (shows code examples)
ubs -v .
```

**Supported languages**: javascript, typescript, python, c, c++, rust, go, java, ruby

### Warp-Grep & FastApply - Fast Parallel Search

**What it is**: MCP server that runs 8 parallel searches across your codebase and provides fast code editing.

**Why you need it**: Normal grep is serial and slow. Warp-Grep is 8× faster and doesn't pollute your context window with search commands.

**Tools provided**:
- **Warp-Grep**: 8 concurrent searches per turn
- **FastApply**: Code edit merging in <1 second

**For humans**: It just makes Claude faster at finding and editing code.

**For agents**: It works automatically - Claude uses it instead of running grep commands.

**When to use Warp-Grep**:
- "How does X work?" discovery
- Data flow across multiple files
- Cross-cutting concerns

**When NOT to use**:
- You know the function name (use `rg`)
- You know the exact file (just open it)

### Exa MCP - AI-Powered Web & Code Search

**What it is**: MCP server that gives Claude access to Exa's AI search engine for web searches and code context retrieval.

**Why you need it**: Claude's knowledge has a cutoff date. Exa lets it search the live web for current documentation, search billions of GitHub repos for code examples, and crawl specific URLs for context.

**Key insight**: When Claude needs to know "how do I use library X" or "what's the latest API for Y", Exa finds real, current answers instead of Claude guessing from outdated training data.

**For humans**: It's like giving Claude access to Google + GitHub search, but smarter.

**For agents**:
```
Tools available:
- web_search_exa: Real-time web search
- get_code_context_exa: Search GitHub, docs, StackOverflow for code
- deep_search_exa: Deep research with smart query expansion
- crawling: Extract content from specific URLs
```

**Setup**: Requires an API key from [dashboard.exa.ai](https://dashboard.exa.ai/api-keys)

**When to use Exa**:
- Current documentation (APIs change after training cutoff)
- Code examples from GitHub/StackOverflow
- Latest library versions or deprecation notices
- Research on best practices

**When NOT to use Exa**:
- Information likely in your codebase (use CASS or Warp-Grep)
- Historical context from past sessions (use cass-memory)
- Task information (use Beads)

---

## Part 2: The Complete Workflow

Here's how everything fits together in a real coding session.

### Phase 1: Starting Your Session

**Goal**: Figure out what to work on.

#### Step 1: Check available work

```bash
bd ready --json
```

This returns tasks with no unresolved dependencies. Example output:
```json
{
  "ready": [
    {"id": "bd-a1b2", "title": "Fix login validation", "priority": "high"},
    {"id": "bd-c3d4", "title": "Add password reset", "priority": "medium"}
  ]
}
```

**Why this matters**: Don't start random work. Start with what's actually unblocked and ready.

#### Step 2: Get priority recommendations

```bash
bv --robot-priority
```

This analyzes the graph and returns:
```json
{
  "recommendations": [
    {
      "id": "bd-a1b2",
      "confidence": 0.92,
      "reasoning": "High betweenness centrality - unblocks 3 downstream tasks",
      "impact": ["bd-e5f6", "bd-g7h8", "bd-i9j0"]
    }
  ]
}
```

**Why this matters**: Not all tasks are equal. This tells you which one has the biggest impact.

#### Step 3: Get context from past learning

```bash
cm context "login validation" --json
```

This returns distilled knowledge from past sessions:
```json
{
  "rules": [
    {"rule": "Always validate email format before database lookup"},
    {"rule": "Use parameterized queries to prevent SQL injection"}
  ],
  "antipatterns": ["Don't use regex for email validation - use a library"],
  "suggested_searches": ["cass search 'login form validation' --robot"]
}
```

**Why this matters**: This isn't just past sessions - it's lessons learned. The system has already done the work of extracting what matters.

#### Step 4: Search for specific solutions (if needed)

```bash
cass search "login validation" --robot --fields summary --limit 5
```

**Why this matters**: If `cm context` suggested searches or you need more detail, dig into specific past sessions.

#### Step 5: Claim the task

```bash
bd update bd-a1b2 --status in_progress
```

**Why this matters**: This prevents duplicate work and tracks who's doing what.

---

### Phase 2: Multi-Agent Setup (If Applicable)

Skip this if you're working solo. Use this when multiple AI agents work on the same project simultaneously.

#### Step 1: Start the coordination server

```bash
am
```

This starts Agent Mail on port 8765.

#### Step 2: Register your identity

```python
ensure_project(human_key="/Users/you/project")

register_agent(
    project_key="/Users/you/project",
    program="claude-code",
    model="opus-4.5"
    # name is auto-generated like "GreenCastle"
)
```

**Why registration matters**: The system needs to know who you are before you can send messages or reserve files.

#### Step 3: Reserve your files

```python
file_reservation_paths(
    project_key="/Users/you/project",
    agent_name="GreenCastle",
    paths=["src/auth/**/*.ts", "src/login/**/*.ts"],
    ttl_seconds=3600,      # 1 hour lease
    exclusive=True,        # No one else can edit these
    reason="bd-a1b2"       # Link to your task
)
```

**Why reservations matter**: This prevents conflicts. If another agent tries to reserve these files, they'll see your lock and work on something else.

**Reservation types**:
- `exclusive=True` - Only you can edit (use for active work)
- `exclusive=False` - Others can observe but you have priority

#### Step 4: Announce your work

```python
send_message(
    project_key="/Users/you/project",
    sender_name="GreenCastle",
    to=["BlueLake", "RedForest"],  # Other agents
    subject="[bd-a1b2] Starting login validation fix",
    body_md="""
    I'm working on the login validation bug.

    Files I'm touching:
    - src/auth/validator.ts
    - src/login/form.ts

    ETA: ~1 hour
    """,
    thread_id="bd-a1b2"
)
```

**The thread_id pattern**: Always use the bead ID as the thread ID. This links everything together:
```
Bead ID:          bd-a1b2
Mail thread:      bd-a1b2
Mail subject:     [bd-a1b2] ...
File reservation: reason="bd-a1b2"
Commit message:   Fixes bd-a1b2
```

---

### Phase 3: During Work

This is where the real coding happens. Here's how to use the tools effectively.

#### Understanding the codebase with Warp-Grep

When you need to understand how something works, use Warp-Grep (it activates automatically for natural language questions):

```
"How does the authentication flow work?"
"Where is user input validated?"
"What functions call the database?"
"Show me all API endpoints that handle payments"
```

**Warp-Grep runs 8 parallel searches**, so asking broad questions is efficient. It returns focused, relevant code snippets.

**When to use it**:
- You're new to a codebase section
- You need to understand data flow
- You're looking for all usages of a pattern

**When NOT to use it** (use regular tools instead):
- You know the exact function name → use `rg` or Grep
- You know the exact file → just Read it

#### Looking up current documentation with Exa

When you need information that might have changed since training:

```
# Check current API documentation
web_search_exa("NextJS 14 app router middleware documentation 2024")

# Find code examples
get_code_context_exa("React server components data fetching patterns")

# Research best practices
deep_search_exa("OAuth 2.0 PKCE flow implementation security best practices")

# Get specific page content
crawling("https://docs.stripe.com/api/payment-intents")
```

**When to use Exa**:
- Library versions or deprecation notices (things change!)
- Current API documentation
- Security advisories
- Code examples from GitHub/StackOverflow
- "What's the recommended way to do X in 2024?"

**When NOT to use Exa**:
- The answer is in your codebase → use CASS or Warp-Grep
- Historical context from past sessions → use `cm context`
- Task information → use Beads

**Example workflow combining tools**:

```
1. Task: "Add OAuth login with Google"

2. Check past learning:
   cm context "OAuth Google authentication" --json
   → Returns: "Use PKCE flow, store tokens in httpOnly cookies"

3. Search codebase for existing auth:
   Warp-Grep: "How does the current login system work?"
   → Returns: existing auth flow in src/auth/

4. Get current Google OAuth docs:
   web_search_exa("Google OAuth 2.0 API documentation 2024")
   → Returns: current endpoints, required scopes

5. Find implementation examples:
   get_code_context_exa("Google OAuth PKCE NextJS implementation")
   → Returns: code snippets from GitHub
```

#### Check your inbox periodically (multi-agent)

```python
messages = fetch_inbox(
    project_key="/Users/you/project",
    agent_name="GreenCastle",
    limit=10
)
```

Other agents might send you important information:
- "Hey, I changed the auth API - update your imports"
- "Found a related bug you should know about"
- "Need your input on a design decision"

#### Acknowledge important messages

```python
acknowledge_message(
    project_key="/Users/you/project",
    agent_name="GreenCastle",
    message_id=123
)
```

#### File new issues as you discover them

You'll find bugs and tasks as you work. Don't lose them:

```bash
bd create "Found: special characters break login" -t bug
```

Link it to the parent task:

```bash
bd dep add bd-x1y2 bd-a1b2 --type discovered-from
```

**Why linking matters**: This creates a paper trail. You can see "this bug was discovered while working on that feature."

#### Send status updates

```python
reply_message(
    project_key="/Users/you/project",
    message_id=123,  # Original message ID
    sender_name="GreenCastle",
    body_md="50% done. Found a related issue, filed as bd-x1y2."
)
```

---

### Phase 4: Before Committing

**This phase is critical. Don't skip it.**

#### Step 1: Scan your changes

```bash
ubs --staged
```

This scans only your staged changes (fast, usually <1 second).

Example output:
```
src/auth/validator.ts:42  [HIGH] Possible SQL injection
src/login/form.ts:18      [MEDIUM] Unclosed file handle
```

#### Step 2: Fix the issues

UBS gives you file:line locations. Fix each issue.

For false positives, add an ignore comment:
```javascript
// ubs:ignore - validated input, safe
const query = `SELECT * FROM users WHERE id = ${userId}`;
```

#### Step 3: Verify clean

```bash
ubs --staged --fail-on-warning
```

- Exit code 0 = safe to commit
- Exit code 1 = still has issues

**Don't commit until this passes.**

#### Step 4: Commit with task reference

```bash
git add .
git commit -m "Fix login validation for special characters

- Added input sanitization
- Updated error messages
- Added test cases

Fixes bd-a1b2"
```

---

### Phase 5: Ending Your Session

#### Step 1: Complete the task

```bash
bd close bd-a1b2 --reason "Fixed: added input sanitization and tests"
```

#### Step 2: Release file reservations

```python
release_file_reservations(
    project_key="/Users/you/project",
    agent_name="GreenCastle"
)
```

**Why this matters**: Other agents are waiting for these files. Release them promptly.

#### Step 3: Send completion message

```python
send_message(
    project_key="/Users/you/project",
    sender_name="GreenCastle",
    to=["BlueLake", "RedForest"],
    subject="[bd-a1b2] Completed",
    body_md="""
    Login validation fix is done and merged.

    This unblocks:
    - bd-e5f6 (password reset)
    - bd-g7h8 (account settings)

    Files changed:
    - src/auth/validator.ts
    - src/login/form.ts
    """,
    thread_id="bd-a1b2"
)
```

#### Step 4: Sync to git

**Never skip this:**

```bash
bd sync  # optional: flush immediately
git add -A && git commit && git push
```

This persists your beads changes and pushes everything.

#### Step 5: See what you unblocked

```bash
bv --robot-diff --diff-since "1 hour ago"
```

This shows tasks that became "ready" because of your work.

---

## Part 3: Quick Reference

### For Agents: Copy-Paste Commands

**Session Start:**
```bash
bd ready --json
bv --robot-priority
cm context "your task description" --json    # Get distilled knowledge
cass search "problem" --robot --limit 5       # If cm suggests or you need more
bd update bd-XXX --status in_progress
```

**During Work:**
```
# Understand codebase (Warp-Grep - automatic for natural language)
"How does the payment processing work?"
"Where are API routes defined?"

# Look up current docs (Exa MCP tools)
web_search_exa("library-name latest API documentation 2024")
get_code_context_exa("pattern implementation examples")
crawling("https://docs.example.com/api")

# If stuck on implementation
cass search "similar problem" --robot
cm context "what I'm trying to do" --json
```

**Multi-Agent (if needed):**
```python
ensure_project(human_key="/abs/path")
register_agent(project_key="/abs/path", program="claude-code", model="opus-4.5")
file_reservation_paths(project_key, agent_name, paths=["src/**"], exclusive=True, reason="bd-XXX")
send_message(project_key, sender_name, to=["OtherAgent"], subject="[bd-XXX] Starting", body_md="...", thread_id="bd-XXX")
```

**Before Commit:**
```bash
ubs --staged
ubs --staged --fail-on-warning  # Must exit 0
```

**Session End:**
```bash
bd close bd-XXX --reason "Completed: summary"
bd sync  # optional: flush immediately
git add -A && git commit && git push
```
```python
release_file_reservations(project_key, agent_name)
```

### For Agents: Critical Rules

1. **NEVER run `bv` without `--robot-*` flags** - TUI will hang your session
2. **ALWAYS use `--robot` or `--json` with CASS/cm** - for machine-readable output
3. **ALWAYS run `cm context` before non-trivial tasks** - get distilled knowledge first
4. **ALWAYS register before messaging** - `ensure_project` + `register_agent` first
5. **ALWAYS commit/push `.beads/issues.jsonl` before ending** (run `bd sync` first if you want zero-lag flush)
6. **ALWAYS run `ubs --staged` before commits** - catch bugs early
7. **ALWAYS release file reservations when done** - don't block other agents

### For Humans: What to Remember

1. **Start with `bd ready`** - see what's actually ready to work on
2. **Use `bv` for priority** - it knows which task matters most
3. **Run `cm context` first** - get distilled lessons from past sessions
4. **Search with `cass` for details** - when you need specific past solutions
5. **Run `ubs` before commits** - catch bugs before they're merged
6. **Sync before you leave** - commit/push (run `bd sync` first if you want)

---

## Part 4: Troubleshooting

| Problem | Solution |
|---------|----------|
| `bd: command not found` | Add `~/.local/bin` to PATH or reinstall via agent-mail installer |
| `bv` hangs | You forgot `--robot-*` flag. Kill and restart with flag |
| Agent Mail errors | Run `am` to start server first |
| CASS finds nothing | Run `cass index --full` to rebuild |
| `cm context` returns empty | Check CASS is indexed, run `cm doctor` |
| UBS module errors | Run `ubs doctor --fix` |
| File reservation conflict | Wait for TTL expiry or coordinate with holder |
| Warp-Grep not working | Check `/mcp` shows morph-fast-tools, verify API key |
| Exa not working | Check `/mcp` shows exa, verify API key from dashboard.exa.ai |
| MCP server not listed | Run `claude mcp add ...` command from installation docs |

---

## Part 5: Example Scenarios

### Scenario 1: Solo Developer, Single Task

```bash
# Morning: What should I work on?
bd ready --json
bv --robot-priority

# Pick task bd-a1b2
bd update bd-a1b2 --status in_progress

# Get context - this is the key step!
cm context "fix login validation bug" --json

# If cm suggests more research or you need specifics
cass search "related problem" --robot --limit 3
```

```
# Understand existing code (Warp-Grep activates automatically)
"How does the current login validation work?"
"Where is user input sanitized in the codebase?"

# Check if there are new best practices (Exa)
web_search_exa("input validation security best practices 2024")
get_code_context_exa("XSS prevention input sanitization")
```

```bash
# ... do the work ...

# Before committing
ubs --staged
# Fix any issues
ubs --staged --fail-on-warning

# Commit
git add . && git commit -m "Feature X - Fixes bd-a1b2"

# End session
bd close bd-a1b2 --reason "Completed"
bd sync  # optional: flush immediately
git add -A && git commit && git push
```

### Scenario 2: Two Agents, Same Project

**Agent 1 (Backend):**
```python
ensure_project(human_key="/project")
register_agent(project_key="/project", program="claude-code", model="opus-4.5")  # Gets name "GreenCastle"

file_reservation_paths(project_key, "GreenCastle",
    paths=["src/api/**", "src/db/**"],
    exclusive=True, reason="bd-backend")

send_message(project_key, "GreenCastle",
    to=["BlueLake"],
    subject="[bd-backend] Working on API",
    body_md="I have the backend. You take frontend.",
    thread_id="bd-backend")
```

**Agent 2 (Frontend):**
```python
ensure_project(human_key="/project")
register_agent(project_key="/project", program="claude-code", model="opus-4.5")  # Gets name "BlueLake"

# Check inbox first
messages = fetch_inbox(project_key, "BlueLake")
# Sees: "I have the backend. You take frontend."

file_reservation_paths(project_key, "BlueLake",
    paths=["src/components/**", "src/pages/**"],
    exclusive=True, reason="bd-frontend")

send_message(project_key, "BlueLake",
    to=["GreenCastle"],
    subject="[bd-frontend] Working on UI",
    body_md="Got it. Taking frontend components.",
    thread_id="bd-frontend")
```

**Both agents work without conflicts because they reserved different files.**

---

*This tutorial is based on the official documentation from each tool's repository. For detailed reference, see the individual tool READMEs in the docs menu.*

*Press q to return to the menu*
