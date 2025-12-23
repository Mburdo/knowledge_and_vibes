# Multi-Agent Orchestrator Patterns 2025

**Sources:**
- Anthropic: "How we built our multi-agent research system" (June 2025)
- "Conductors to Orchestrators: The Future of Agentic Coding" (November 2025)
- GitHub: multi-agent-coding-system (#13 on Terminal Bench)
- Microsoft: "Build Multi-Agent AI Systems" (September 2025)

---

## Summary

2025 research consolidates around the **orchestrator-worker pattern** for multi-agent coding systems. Key findings on running 10+ parallel agents without interference.

---

## The Orchestrator Pattern

### Core Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATOR                             │
│  - Analyzes requirements                                         │
│  - Breaks down into parallel tracks                              │
│  - Distributes to workers                                        │
│  - Monitors progress via shared state                            │
│  - Coordinates convergence                                       │
└─────────────────────────────────────────────────────────────────┘
                              │
     ┌────────────────────────┼────────────────────────────────┐
     │            │           │           │                    │
     ▼            ▼           ▼           ▼                    ▼
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         ┌─────────┐
│Worker 1 │ │Worker 2 │ │Worker 3 │ │Worker 4 │   ...   │Worker N │
│Track A  │ │Track B  │ │Track C  │ │Track D  │         │Track N  │
└─────────┘ └─────────┘ └─────────┘ └─────────┘         └─────────┘
     │            │           │           │                    │
     └────────────┴───────────┴───────────┴────────────────────┘
                              │
                    Shared Context Store
              (File reservations, messages, state)
```

### Anthropic's Design (June 2025)

From their multi-agent research system:

1. **Lead agent (orchestrator)** coordinates the process
2. **Subagents (workers)** operate **in parallel** for simultaneous search
3. **Significant performance improvement** over sequential processing
4. **Fresh context per subagent** prevents context bombing

---

## Key Patterns for Parallel Execution

### 1. Track-Based Partitioning

Partition work by subsystem to minimize conflicts:

```markdown
## Phase 1: Auth System (10 agents)

| Track | Agent | Scope | Files |
|-------|-------|-------|-------|
| A: JWT | BlueLake | Token handling | src/auth/jwt/** |
| B: Session | GreenCastle | Session mgmt | src/auth/session/** |
| C: Middleware | RedStone | Auth middleware | src/middleware/auth/** |
| D: Models | PurpleBear | User models | src/models/user/** |
| E: Tests-JWT | OrangeFox | JWT tests | tests/auth/jwt/** |
```

### 2. File Reservation Protocol

Prevent edit conflicts with exclusive locks:

```python
# Before editing
file_reservation_paths(
    paths=["src/auth/jwt/**"],
    exclusive=True,
    ttl_seconds=3600,
    reason="bd-101"
)

# After completing
release_file_reservations()
```

### 3. Message-Based Coordination

Agents communicate state changes via messaging:

| Message Type | When | Purpose |
|--------------|------|---------|
| `[CLAIMED] bd-XXX` | Starting work | Announce intent |
| `[CLOSED] bd-XXX` | Completed | Signal done |
| `[BLOCKED] Agent` | Stuck | Request help |
| `[TRACK COMPLETE]` | Track done | Ready for sync |

### 4. Synchronization Points

All agents must sync at phase boundaries:

```
Phase 1 Execution
├── Track A: ████████████ DONE
├── Track B: ████████████ DONE
├── Track C: ████████████ DONE
└── Track D: ████████████ DONE
                │
                ▼
         CALIBRATION
    (all tracks must complete)
                │
                ▼
Phase 2 Execution (new track assignments)
```

---

## Performance Findings

### From 2025 Research

| Metric | Single Agent | 10 Parallel Agents | Improvement |
|--------|--------------|-------------------|-------------|
| Wall clock time | 100% | 15-20% | **5-7x faster** |
| Total tokens | 100% | 110-130% | Slight overhead |
| Conflict rate | 0% | 2-5% (with reservations) | Acceptable |
| Success rate | Baseline | Similar | No degradation |

### Scaling Observations

- **Optimal parallelism:** 5-10 agents for most codebases
- **Diminishing returns:** Beyond 10 agents, coordination overhead increases
- **Conflict surface:** Larger codebase = more parallelism possible
- **Track granularity:** Match to natural subsystem boundaries

---

## Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| No file locks | Edit conflicts | Always use reservations |
| Shared mutable state | Race conditions | Message-based coordination |
| No sync points | Drift accumulation | Calibration between phases |
| Over-parallelization | Coordination overhead | Match to subsystem count |
| Silent workers | Invisible progress | Require announcements |

---

## Context Store Design

The orchestrator maintains shared state:

```json
{
  "tracks": {
    "A": {"agent": "BlueLake", "status": "in_progress", "beads": ["bd-101"]},
    "B": {"agent": "GreenCastle", "status": "complete", "beads": ["bd-102"]}
  },
  "file_reservations": {
    "src/auth/jwt/**": {"holder": "BlueLake", "expires": "2025-01-15T12:00:00Z"}
  },
  "messages": [
    {"from": "GreenCastle", "subject": "[CLOSED] bd-102", "ts": "..."}
  ]
}
```

---

## Practical Implications

### For Knowledge & Vibes

This validates our parallel execution architecture:
- **Parallel execution** — Multiple agents on tracks
- **Agent Mail coordination** — Message-based state
- **File reservations** — Conflict prevention
- **Calibration points** — Phase synchronization

### Implementation Checklist

- [ ] Identify natural subsystem boundaries
- [ ] Create tracks that partition files
- [ ] Assign one agent per track
- [ ] Require file reservations before edits
- [ ] Require announcements on claim/close
- [ ] Sync at phase boundaries
- [ ] Calibrate before next phase

---

## See Also

- `056-multi-agent-orchestrator.md` — Original orchestrator research
- `057-anthropic-context-engineering.md` — Context management
- `038-adapt.md` — Adaptive decomposition within agents
