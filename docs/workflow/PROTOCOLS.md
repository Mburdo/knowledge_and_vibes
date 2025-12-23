# Evidence-Backed Protocols

19 short, repeatable procedures that turn AI-assisted work into verifiable progress. Each protocol exists because skipping it causes predictable failures.

**How to use:** Each protocol is a "card" with Why, Inputs, Outputs, Gate, and Evidence. Gates are mandatory.

---

## Table of Contents

**Quick Reference**
- [Protocol Summary Table](#quick-reference)
- [Why Protocols Matter](#why-protocols-matter)

**Planning Protocols (P0-P5)**
- [P0: North Star](#p0--north-star-ground-truth)
- [P1: REQ/AC](#p1--requirements-that-compile-req--ac)
- [P2: Requirements QA](#p2--requirements-qa-ambiguity-is-expensive)
- [P3: Grounding](#p3--grounding-where-does-the-truth-live)
- [P4: Decision Search](#p4--decision-search-abc-then-select)
- [P5: Risks & Spikes](#p5--risks--spikes-evidence-production-not-features)

**Decomposition Protocols (P6-P8)**
- [P6: Phase Chunking](#p6--phase-chunking-short-context-lossless)
- [P7: Bead Packaging](#p7--bead-packaging-tdd-first-reduce-guessing)
- [P8: Adaptive Decomposition](#p8--adaptive-decomposition-adapt-split-only-when-execution-fails)

**Execution Protocols (P9-P10)**
- [P9: Execution Loop](#p9--execution-loop-generate--run--repair-hard-limit-3-iterations)
- [P10: Calibration](#p10--calibration-between-phases-hard-stop)
- [P10a: Test-Based Disagreement](#p10a--test-based-disagreement-resolution-replaces-rhetorical-debate)

**Quality Protocols (P11-P17)**
- [P11: Traceability](#p11--traceability-coverage-map)
- [P12: Release Readiness](#p12--release-readiness-rigortier-gate)
- [P13: Security Gate](#p13--security-gate-mandatory-language-aware)
- [P14: Human Verification](#p14--human-verification-gate-operator-checkpoint)
- [P15: Task Complexity](#p15--task-complexity-assessment-before-execution)
- [P16: AI Code Review](#p16--ai-code-review-two-stage)
- [P17: Extended Thinking](#p17--extended-thinking-for-critical-tasks)

---

## Quick Reference

| Protocol | Purpose | When to Use |
|----------|---------|-------------|
| P0 | North Star | Project start |
| P1 | REQ/AC | After North Star |
| P2 | Requirements QA | Before planning |
| P3 | Grounding | Before any external claim |
| P4 | Decision Search | Architecture decisions |
| P5 | Risks & Spikes | Before committing |
| P6 | Phase Chunking | Plan breakdown |
| P7 | Bead Packaging | Before execution |
| P8 | Adaptive Decomposition | When beads fail |
| P9 | Execution Loop | During implementation |
| P10 | Calibration | Between phases |
| P10a | Test-Based Disagreement | When agents conflict |
| P11 | Traceability | Throughout |
| P12 | Release Readiness | Before ship |
| P13 | Security Gate | Every commit |
| P14 | Human Verification | P0 implementations |
| P15 | Task Complexity | Before bead execution |
| P16 | AI Code Review | MEDIUM+ complexity beads |
| P17 | Extended Thinking | Complex/critical tasks |

---

## Why Protocols Matter

Without protocols, AI-assisted development fails in predictable ways:
- **Drift:** AI solves problems you didn't ask for
- **Hallucination:** Confident claims that aren't true
- **Silent omission:** Requirements dropped in long contexts
- **Security holes:** Vulnerable code that "works"
- **Infinite loops:** Repeated fixes that make things worse

Protocols prevent these failures by enforcing gates that catch problems early.

---

## P0 ,  North Star (Ground Truth)

**Why:** drift is the default; you need an explicit tradeoff policy.  
**Inputs:** operator intent + context/stakes.  
**Output:** North Star Card pinned at the top of the plan.  
**Template:** `templates/NORTH_STAR_CARD_TEMPLATE.md`  
**Gate:** contains success metrics, non‑goals, invariants, stop/ask rule.

---

## P1 ,  Requirements That “Compile” (`REQ-*` / `AC-*`)

**Why:** “correct” must be checkable by tests, not vibe.  
**Inputs:** North Star Card.  
**Template:** `templates/REQUIREMENTS_TEMPLATE.md`  
**Output:** `PLAN/01_requirements.md` with `REQ-*` and measurable `AC-*`.  
**Gate:** every P0 `REQ-*` has ≥ 1 falsifiable `AC-*`.

---

## P2 ,  Requirements QA (Ambiguity is expensive)

**Why:** ambiguity becomes rework; QA early is leverage.  
**Template:** `templates/REQUIREMENTS_QA_TEMPLATE.md`  
**Output:** rewrites + operator confirmation questions + AC coverage check.  
**Gate:** no ambiguous/non‑verifiable P0 requirements remain.  
**Evidence base:** `research/036-requirements-qa-iso-29148.md`

---

## P3 ,  Grounding (Where does the truth live?)

**Why:** API hallucinations and outdated patterns are common.  
**Use:**
- repo truth → Warp‑Grep
- web truth → Exa
- history truth → `cm` / `cass`
**Command:** `/ground`  
**Gate:** architecture/security claims are grounded or explicitly marked as assumptions.  
**Evidence base:** `research/008-api-hallucination.md`, `research/017-context-retrieval-repo-editing.md`

---

## P4 ,  Decision Search (A/B/C, then select)

**Why:** single‑path planning is fragile; search improves outcomes.  
**Template:** `templates/DECISIONS_ADRS_TEMPLATE.md`  
**Output:** `PLAN/02_decisions_adrs.md` entries with:
- options A/B/C
- tradeoffs
- risks
- “what would change our mind” (reversal triggers)
**Gate:** every architecturally significant requirement is decided or explicitly “pending spike.”  
**Evidence base:** `research/019-plansearch.md`, `research/020-codetree.md`, `research/042-rankef.md`

---

## P5 ,  Risks & Spikes (Evidence production, not features)

**Why:** collapse uncertainty before committing to long sequences.  
**Template:** `templates/RISKS_AND_SPIKES_TEMPLATE.md`  
**Gate:** top risks become timeboxed spikes with explicit outputs and decision impact.  
**Evidence base:** `research/031-alphacodium.md`, `research/014-codeplan.md`

---

## P6 ,  Phase Chunking (Short context, lossless)

**Why:** long context causes silent omission.  
**Output:** phase docs sized for reliable agent execution (coarse heuristic guardrail: ~500–1000 lines).  
**Gate:** each phase is independently implementable and references the North Star + relevant REQ/AC.  
**Evidence base:** `research/004-context-length-hurts.md`, `research/005-lost-in-middle-code.md`

---

## P7 ,  Bead Packaging (TDD-First, Reduce Guessing)

**Why:** "task quality" determines execution quality. TDD yields 45.97% pass@1 improvement.
**Template:** `.claude/templates/beads/bead-structure.md`
**Required:**
- **Tests FIRST** (write test code in bead before implementation, not after)
- edit locus, context sufficiency check, REQ/AC links
- security checklist (inputs validated, no hardcoded secrets)

**Gate:** another agent could implement the bead without guessing. Tests exist before implementation begins.
**Evidence base:** `research/037-requirements-to-code-practices.md`, `research/054-tdd-ai-code-gen.md`

> **2025 update:** TDD is not optional. Tests in the bead description, written before implementation, yield 45.97% higher success rates.

---

## P8 ,  Adaptive Decomposition (ADaPT: Split Only When Execution Fails)

**Why:** rigid decomposition overfits to guesses; execution reveals reality. Over-decomposing upfront wastes effort on problems that don't exist.

**The ADaPT Pattern:**
```
1. START COARSE: Create beads at the level you think is atomic
2. ATTEMPT EXECUTION: Run tests + implement
3. IF SUCCESS: Close bead, done
4. IF FAILURE (after 3 iterations):
   a. STOP - don't keep trying
   b. ANALYZE: What specific sub-problem failed?
   c. SPAWN SUB-BEAD: For ONLY the failing part
   d. Continue from step 2
```

**Trigger:** Bead fails after 3 repair iterations.
**Action:** Spawn sub-bead for the specific failing component, not the whole bead.

**Anti-patterns:**
- Pre-decomposing "just in case"
- Creating sub-beads before attempting execution
- Decomposing based on guesses about complexity

**Gate:** Sub-beads created only in response to execution failure, not upfront planning.

**Evidence base:** `research/038-adapt.md` (decompose only when needed), `research/011-agentless.md` (simple pipelines beat complex agents)

---

## P9 ,  Execution Loop (Generate → Run → Repair, HARD LIMIT: 3 Iterations)

**Why:** disciplined execution feedback beats "think harder." But debugging capability decays exponentially.

### Mathematical Basis (DDI)

The Debugging Decay Index proves:
```
E(t) = E₀ × e^(-λt)

At iteration 1: 100% capability
At iteration 2: 40-60% capability
At iteration 3: 20-40% capability
At iteration 4+: ACTIVELY HARMFUL (introduces more bugs than it fixes)
```

### HARD LIMIT: 3 Iterations ,  NO EXCEPTIONS

**Rules:**
- Every iteration must cite **new external evidence** (failing test/log/trace)
- **Iteration tracking is MANDATORY** ,  log "Attempt {n}/3" after each try
- `ubs --staged` must pass before any bead closes
- **NEVER self-assess** ,  "are you sure?" patterns overturn 58% of correct answers

### Correction Flow (External Feedback Only)

```
CORRECT:
1. Generate code
2. Run tests → Get failure message (external)
3. Analyze failure message (external evidence)
4. Generate fix based on failure

WRONG (DO NOT DO):
1. Generate code
2. "Let me check if this looks right..." ← NO
3. Self-assess and modify ← NO
```

**Gate:** no bead closes without verification passing AND `ubs` clean.

**If still failing after 3 iterations:**
1. **STOP IMMEDIATELY** ,  do not try "one more time"
2. Spawn spike bead to investigate root cause (ADaPT)
3. Notify operator via Agent Mail with `importance: high`
4. Fresh context for new attempt

**Evidence base:** `research/060-debugging-decay-index.md` (DDI proves decay), `research/071-self-correction-rl.md` (external feedback essential), `research/022-chatrepair.md`, `research/053-feedback-loop-security.md`

> **2025 update:** Research proves debugging capability decays 60-80% within 2-3 attempts. The 3-iteration limit is not a guideline, it's a mathematical ceiling. Beyond it, you're making things worse.

---

## P10 ,  Calibration (Between phases, hard stop)

**Why:** drift compounds; calibration is the search controller (branch/prune/pivot).
**Command:** `/calibrate`
**Evidence standard:** code/test/doc/measurement, or a discriminating test.
**Gate:** no next phase begins until calibration produces falsifiable decisions + recorded plan changes.
**Evidence base:** `research/003-debate-or-vote.md`, `research/041-debatecoder.md`, `research/019-plansearch.md`, `research/020-codetree.md`

---

## P10a ,  Test-Based Disagreement Resolution (Replaces Rhetorical Debate)

**Why:** Rhetorical debate produces persuasive noise. Tests produce verifiable truth. Voting alone beats extended debate.

**When:** Agents disagree on approach, architecture, or correctness.

**Protocol:**
1. Each agent writes **discriminating tests** (tests that PASS for their approach, FAIL for alternatives)
2. Run all tests against all proposals
3. Test results adjudicate, not rhetoric
4. If tests don't discriminate: define what measurement/experiment would resolve it
5. If value-dependent: preserve both positions for user decision

**Anti-patterns:**
- Extended rhetorical debate (degrades outcomes)
- "Arguing" without code/tests
- Compromising to reach agreement (loses correctness)
- More than 2 rounds of discussion without test results

**Gate:** Disagreements resolved by test results or explicitly preserved for user decision.

**Evidence base:** `research/003-debate-or-vote.md` (voting beats debate), `research/041-debatecoder.md` (tests as medium), `research/056-multi-agent-orchestrator.md` (orchestrator-worker +90.2%)

---

## P11 ,  Traceability (Coverage map)

**Why:** agents work locally; you need a global coverage map to prevent gaps.  
**Template:** `templates/TRACEABILITY_TEMPLATE.md`  
**Rule:** this is agent‑maintained; operator should not have to update it.  
**Gate:** every P0 requirement maps to beads + tests + acceptance.

---

## P12 ,  Release Readiness (Rigor‑tier gate)

**Why:** "passes tests" is not always "safe to ship."
**Output:** release checklist tied to rigor tier (tests, observability, rollback, security posture).
**Evidence base:** `research/021-swe-bench-plus.md`, `research/047-humaneval-pro.md`, `research/044-iris.md`

> **2025 update:** "Tests pass" is necessary but not sufficient. Human review is required for production readiness, especially for P0 requirements and unfamiliar domains.

---

## P13 ,  Security Gate (Mandatory, Language-Aware)

**Why:** ~40% of LLM-generated code contains vulnerabilities. Security is not optional.
**Tool:** `ubs --staged` (1000+ vulnerability patterns)
**When:** Every commit, every bead close, every PR.

### Language-Specific Risk Levels

Research shows significant variance by language:

| Language | OWASP Failure Rate | Risk Level | Required Action |
|----------|-------------------|------------|-----------------|
| **Java** | **72%** | CRITICAL | Extra manual review required |
| C# | 45% | HIGH | Standard + SQL injection audit |
| JavaScript | 43% | HIGH | Standard + XSS review |
| Python | 38% | STANDARD | `ubs --staged` sufficient |

### XSS and Log Injection (Most Missed)

| Vulnerability | Detection Rate | Action |
|---------------|---------------|--------|
| XSS | Only 14% caught | Manual output encoding review for web code |
| Log Injection | Only 26% caught | Check all logging for user input |

**Rules:**
- Zero tolerance for high/critical findings, fix before proceeding
- Medium findings require documented justification in commit message
- Security-critical files (auth, crypto, input handling) require file reservation via Agent Mail
- **Java code:** Requires ADDITIONAL manual SQL injection review beyond ubs
- **Web code (JS/TS):** Requires manual XSS output encoding review

**Gate:** `ubs --staged` returns zero high/critical findings PLUS language-specific checks.

**Evidence base:** `research/061-llm-security-2025.md` (language-specific rates), `research/052-llm-security-vulnerabilities.md`, `research/053-feedback-loop-security.md`, `research/044-iris.md`

**Top 5 LLM vulnerability patterns:**
1. Command injection via unsanitized input
2. SQL injection via string formatting (especially Java: 72% fail)
3. Path traversal via user-controlled paths
4. Hardcoded secrets/credentials
5. Missing authentication checks

**XSS Prevention Checklist (for web code):**
- [ ] User input escaped before HTML output
- [ ] No `innerHTML` or `dangerouslySetInnerHTML` with user data
- [ ] Content-Security-Policy headers present
- [ ] JSON responses use proper content-type

> **2025 update:** Java has a 72% OWASP failure rate. XSS is missed 86% of the time. Language matters, adjust scrutiny accordingly.

---

## P14 ,  Human Verification Gate (Operator Checkpoint)

**Why:** AI helps most when the human already understands the domain. Verification catches mistakes before they compound.

**When:**
- All P0 requirement implementations
- Any code in unfamiliar domains (operator cannot explain what it does)
- Auth, payment, and data handling code
- Before merge to main/production branches

**Rules:**
- Operator must be able to explain what the AI-generated code does (not just that tests pass)
- If operator cannot explain, request `/ground` + explicit walkthrough
- Sign-off required in PR description or commit message

**Gate:** Operator sign-off on all P0 requirement implementations before merge.

**Evidence base:** `research/051-metr-rct.md`, `research/050-swe-bench-pro.md`

**Reality check prompt for operator:**
```
Before approving this code:
1. Can you explain what this code does in plain English?
2. Can you identify where user input enters the system?
3. Can you trace data flow through the critical path?

If "no" to any: request walkthrough before approval.
```

> **2025 update:** AI assistance improves productivity only when humans verify output in domains they understand. Blind trust in AI-generated code leads to slower, buggier results.

---

## P15 ,  Task Complexity Assessment (Before Execution)

**Why:** Task complexity determines verification requirements. Multi-file, cross-cutting changes need more scrutiny than single-file edits.

**When:** Before executing any bead.

### Complexity Classification

| Complexity | Characteristics | Expected AI Success | Verification Level |
|------------|-----------------|---------------------|-------------------|
| **Simple** | Single file, clear spec, isolated change | 70-80% | Standard (tests + ubs) |
| **Medium** | 2-3 files, some ambiguity, cross-module | 40-50% | Thorough (+ code review) |
| **Complex** | Multi-file, cross-cutting, architectural | 15-25% | Human oversight required |
| **Architectural** | System-wide changes, API contracts | <10% | Human-led with AI assistance |

### Complexity Indicators

```
FILES TOUCHED:
- 1 file → Simple
- 2-3 files → Medium
- 4+ files → Complex

CROSS-MODULE:
- Changes in one module → Simple
- Changes span modules → Complex

AMBIGUITY:
- Clear spec, one approach → Simple
- Multiple valid approaches → Medium
- Unclear requirements → Complex (clarify first)
```

### Verification Requirements by Complexity

**Simple:**
- Tests pass
- `ubs --staged` clean

**Medium:**
- Tests pass
- `ubs --staged` clean
- Code review by another agent or human

**Complex:**
- Tests pass
- `ubs --staged` clean
- Human review REQUIRED before merge
- Consider spawning multiple agents with ADaPT

**Architectural:**
- Human leads implementation
- AI provides assistance only
- Full P14 verification

**Gate:** Complexity assessed before execution; verification level matches complexity.

**Evidence base:** `research/066-swe-bench-pro-2025.md` (harder benchmarks reveal true capability), `research/069-adacoder.md` (model capability profiling)

> **2025 update:** Complex tasks require commensurate verification. Match scrutiny level to task complexity.

---

## P16 ,  AI Code Review (Two-Stage)

**Why:** 14.9% of PRs now involve AI review (14x growth since 2024). Two-stage review (detect + verify) is most effective.

**When:**
- Before closing any MEDIUM+ complexity bead
- Before calibration points
- Before release

### Two-Stage Process

**Stage 1: Detection (Automated)**
```bash
# Run all detection tools
ubs --staged              # Security
npm run lint              # Style
npm run typecheck         # Types
# AI pattern scan (if configured)
```

**Stage 2: Verification (Before Merge)**
- Are flagged issues real? (filter false positives)
- Did automated tools miss obvious issues?
- Human spot-check for complex changes

### Comment Format (Higher Adoption)

Research shows comments with code snippets are adopted more often:

```markdown
## GOOD (gets adopted)

**Issue:** SQL injection risk at line 42

**Before:**
```python
query = f"SELECT * FROM users WHERE id = {id}"
```

**After:**
```python
query = "SELECT * FROM users WHERE id = ?"
cursor.execute(query, (id,))
```

**Why:** User input directly in query enables attacks.
```

```markdown
## BAD (gets ignored)

"Consider improving database access code."
```

### Triggering

| Trigger | Adoption Rate | Use When |
|---------|--------------|----------|
| **Manual** | Higher | Focused review on specific changes |
| **Auto on all PRs** | Lower | Background safety net |

**Recommendation:** Use manual trigger for meaningful review, auto for baseline.

**Gate:** MEDIUM+ complexity beads receive Stage 1 before close, Stage 2 before merge.

**Evidence base:** `research/072-ai-code-review-2025.md` (14.9% adoption, comment format matters)

> **2025 update:** AI code review is mainstream. Use two-stage (detect + verify) and include code snippets in comments for higher adoption.

---

## P17 ,  Extended Thinking (For Critical Tasks)

**Why:** Extended thinking gives Claude internal reasoning space before responding. Research shows +9.7% improvement on complex control flow and reasoning tasks.

**When to Enable:**

| Task Type | Thinking Budget | Rationale |
|-----------|-----------------|-----------|
| Architectural decisions | 20,000 | Long-term consequences; need thorough analysis |
| Calibration challenges | 16,000 | Multi-report synthesis; discriminating tests |
| Complex bead implementation | 10,000-16,000 | Control flow, algorithms, multi-file |
| Debugging (attempt 1-2) | 10,000 | Better first attempts reduce iterations |
| Synthesis tasks | 16,000 | Cross-referencing multiple sources |

**When NOT to Enable:**

- Simple file edits
- Routine test writing
- Documentation updates
- Single-line fixes

### Thinking Budget Guidelines

```
10,000 tokens ,  Standard complex task
16,000 tokens ,  Multi-source synthesis, challenge resolution
20,000 tokens ,  Architectural decisions (highest stakes)
32,000 tokens ,  Maximum (rarely needed)
```

### Integration with DDI

Extended thinking on early iterations reduces total iterations needed:

```
WITHOUT Extended Thinking:
- Iteration 1: 60% success
- Iteration 2: 40% remaining × 50% = 20% more
- Iteration 3: 20% remaining × 30% = 6% more
- Total: ~86% after 3 iterations

WITH Extended Thinking (first attempt):
- Iteration 1: 75% success (+15% from reasoning)
- Fewer iterations needed → Less DDI decay
```

### Agents with Extended Thinking Enabled

| Agent | Budget | Reason |
|-------|--------|--------|
| `decisions.md` | 20,000 | ADRs, risk assessment, spikes |
| `challenge.md` | 16,000 | Discriminating test design |
| `synthesize.md` | 16,000 | Multi-report integration |
| `execution.md` (complex beads) | 10,000-16,000 | Based on P15 complexity |

**Gate:** Extended thinking allocation based on P15 complexity assessment.

**Evidence base:** `research/070-structured-cot-code.md` (+9.7% on complex control flow), Claude platform documentation

> **2025 update:** Extended thinking is now standard for critical tasks. Allocate budget based on task complexity and stakes. Better first attempts via thinking reduce DDI decay.
