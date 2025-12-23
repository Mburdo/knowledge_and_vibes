# LLM Planning for Code - Research Summaries

Research papers read in full and summarized with practical implications for the Knowledge & Vibes planning system.

---

## Index

| # | Paper | Key Finding | Implication |
|---|-------|-------------|-------------|
| 001 | [Live-SWE-agent](001-live-swe-agent.md) | Dynamic tool creation during execution → 77.4% on SWE-bench | Let agents create helper scripts; core workflow stays fixed |
| 002 | [Planning-Driven Programming](002-planning-driven-programming.md) | Plan-then-implement → 2-10% improvement; plan aids debugging more than initial gen | Plans are most valuable for debugging, not just generation |
| 003 | [Debate or Vote](003-debate-or-vote.md) | Voting alone beats unstructured debate; debate is a random walk | Require evidence in challenges; lock in verified claims |
| 004 | [Context Length Hurts](004-context-length-hurts.md) | 30-50% degradation on real tasks as length increases | Keep relevant context together; retrieval + short-context works |
| 005 | [Lost-in-Middle Code](005-lost-in-middle-code.md) | Semantic understanding suffers more than lexical retrieval | Put critical info at section boundaries; position matters |
| 006 | [Dark Side Self-Correction](006-dark-side-self-correction.md) | "Are you sure?" without new info → 58% correct answers overturned | Only ask for revision with new information (test results, evidence) |
| 007 | [SWE-Gym](007-swe-gym.md) | Training on test-verified trajectories → +12-13% on SWE-bench | Test-based verification is the key training signal |
| 008 | [API Hallucination](008-api-hallucination.md) | Low-freq APIs: 38% valid; doc retrieval helps but can hurt known APIs | Selective grounding: fetch docs for rare APIs, trust model for common ones |
| 009 | [SWE-bench](009-swe-bench.md) | Repo-level issue benchmark exposes localization + execution bottlenecks | Plan/localize/validate beats one-shot patching |
| 010 | [SWE-agent](010-swe-agent.md) | Agent-computer interface design drives large performance gains | Treat tooling + slash commands like an agent IDE |
| 011 | [Agentless](011-agentless.md) | Fixed localization→repair→validate pipeline can beat complex agents | Scaffold design + validation often matter more than debate |
| 012 | [AutoCodeRover](012-autocoderover.md) | Structure-aware code search + iterative retrieval improves SWE fixing | Invest in symbol/AST-aware navigation primitives |
| 013 | [SpecRover](013-specrover.md) | Explicit spec inference + reviewer improves patching & trust | Make “intent/spec” an artifact; add reviewer-style calibration |
| 014 | [CodePlan](014-codeplan.md) | Repo work is a chain of edits; dependencies create derived obligations | Use conditional beads triggered by oracle/build/test feedback |
| 015 | [Coeditor](015-coeditor.md) | Repo-diff conditioning is a strong signal for code auto-editing | Treat diffs as first-class shared context across agents |
| 016 | [RES-Q](016-res-q.md) | Repo-scale edit benchmark differentiates models beyond HumanEval | Systematize “vague request → clarified requirement” as a step |
| 017 | [Context Retrieval (Repo Editing)](017-context-retrieval-repo-editing.md) | Reasoning boosts retrieval precision, not sufficiency detection | Add explicit “context sufficiency” checks before editing |
| 018 | [LiveCodeBench](018-livecodebench.md) | Contamination-resistant eval + self-repair as a core capability | Make repair loops explicit and measurable |
| 019 | [PlanSearch](019-plansearch.md) | Searching over NL plans yields big pass@k gains via diversity | Generate diverse plans; select with evidence and filters |
| 020 | [CodeTree](020-codetree.md) | Tree search guided by execution feedback improves hard code gen | Use calibration to prune/branch strategies, not just reflect |
| 021 | [SWE-Bench+](021-swe-bench-plus.md) | Leakage + weak tests inflate SWE-bench; decontamination drops scores | Don’t optimize against leaky signals; strengthen tests/criteria |
| 022 | [ChatRepair](022-chatrepair.md) | Conversation + test feedback improves APR cost/effectiveness | Encode “failure feedback → revision” loops as standard beads |
| 023 | [Toggle](023-toggle.md) | Token-level localization + constrained edits improves APR | Separate “where to edit” from “what to change” rigorously |
| 024 | [ThinkRepair](024-thinkrepair.md) | Build a validated exemplar pool and retrieve it for repair | Capture solved beads as reusable exemplars/memory |
| 025 | [APPATCH](025-appatch.md) | Vulnerability patching improves with slicing + adaptive exemplars + validation | For high-stakes work, mandate analysis+validation beads |
| 026 | [FLAMES](026-flames.md) | Semantic-guided search reduces VRAM and improves repair outcomes | Use tests as online reward; optimize search under constraints |
| 027 | [SWE-smith](027-swe-smith.md) | Synthetic bug task scaling enables strong open SWE agents | Tests are leverage: they enable scaling, training, and autonomy |
| 028 | [SWE-bench-Live](028-swe-bench-live.md) | Fresh tasks reveal brittleness/overfitting to static benchmarks | Plan for drift; recalibrate frequently against current reality |
| 029 | [SWE-rebench](029-swe-rebench.md) | Continuous task mining + standardized eval + variance reporting | Standardize scaffolds; run multiple trials; report uncertainty |
| 030 | [SWE-bench Multimodal](030-swe-bench-multimodal.md) | Visual + JS SWE tasks expose multimodal/generalization gaps | Add visual acceptance artifacts + multimodal tooling plans |
| 031 | [AlphaCodium](031-alphacodium.md) | Flow engineering + tests + repair loops dramatically improve code gen | Encode test-driven multi-stage flows into beads |
| 032 | [TDD for Code Generation](032-tdd-code-generation.md) | Supplying tests + remediation boosts solve rates and robustness | Treat tests as the operator-facing spec surface |
| 033 | [Requirements → Code (Progressive Prompting)](033-requirements-to-code.md) | Stepwise requirements→design→tests→code improves inspectability | Make requirements quality + progressive artifacts central |
| 034 | [LLMs in RE (Guideline)](034-llms-in-re-guideline.md) | Choose LLM methods by task type (understand vs generate) | Route RE work through structured sub-tasks, not freeform chat |
| 035 | [LLM vs Human RE](035-llm-vs-human-re.md) | LLMs can match/beat experts on alignment/completeness at far lower cost | Invest in a rigorous REQ/AC workflow for operators |
| 036 | [Requirements QA (ISO 29148)](036-requirements-qa-iso-29148.md) | LLMs catch requirement issues well; explanations improve agreement | Add a requirements QA gate before planning/execution |
| 037 | [Requirements→Code Practices](037-requirements-to-code-practices.md) | Practitioners decompose requirements into concrete tasks with constraints | Make task packaging a formal stage before bead execution |
| 038 | [ADaPT](038-adapt.md) | Decompose only when execution fails | Prefer as-needed decomposition over rigid trees |
| 039 | [TDAG](039-tdag.md) | Dynamic decomposition + generated subagents | Treat decomposition as adaptive; store reusable skills |
| 040 | [MapCoder](040-mapcoder.md) | Retrieval→Planning→Coding→Debugging pipeline helps | Make stage separation a default bead workflow |
| 041 | [DebateCoder](041-debatecoder.md) | Debate via adversarial tests, not rhetoric | Use tests as disagreement medium in calibration |
| 042 | [RankEF](042-rankef.md) | Execution feedback improves candidate reranking | Diversity needs strong selection signals |
| 043 | [RLEF](043-rlef.md) | Models learn to use execution feedback iteratively | Encode disciplined feedback→patch loops |
| 044 | [IRIS](044-iris.md) | LLM+static analysis can beat static tools alone | Add security verification gates for high-rigor work |
| 045 | [RepairAgent](045-repairagent.md) | Tool-using repair loop works for program repair | Make repair mode explicit and stateful |
| 046 | [D4C](046-d4c.md) | Direct debugging can beat fault-localize+infill | Choose edit strategy: surgical vs bounded rewrite |
| 047 | [HumanEval Pro / MBPP Pro](047-humaneval-pro.md) | Strong drops on progressive “reuse” tasks | Justify between-phase calibration + stronger invariants |
| 048 | [EG‑CFG](048-eg-cfg.md) | Execution-guided decoding improves code gen | Favor small validated steps + frequent execution |
| 049 | [Code‑Form Planning](049-codeform-planning.md) | Structured pseudocode plans improve reasoning | Require structured plans for complex logic beads |
| 050 | [SWE-Bench Pro](050-swe-bench-pro.md) | Benchmark for realistic multi-file tasks | Verification requirements scale with task complexity |
| 051 | [METR RCT](051-metr-rct.md) | Productivity depends on domain familiarity | AI helps most when human understands the domain; verify P0 requirements |
| 052 | [LLM Security Vulnerabilities](052-llm-security-vulnerabilities.md) | ~40% of LLM-generated code has vulnerabilities | `ubs --staged` is mandatory, not optional |
| 053 | [Feedback Loop Security](053-feedback-loop-security.md) | Security degrades with repeated self-correction | Max 3 iterations; escalate after failures |
| 054 | [TDD for AI Code Gen](054-tdd-ai-code-gen.md) | TDD yields 45.97% pass@1 improvement | Tests must be written FIRST, not after |
| 055 | [Test Intentions](055-test-intentions.md) | 94% coverage with intentions vs 59% with raw prompts | Include test intentions in requirements |
| 056 | [Multi-Agent Orchestrator](056-multi-agent-orchestrator.md) | Orchestrator-worker pattern outperforms by 90.2% | Use structured coordination, not free-form debate |
| 057 | [Anthropic Context Engineering](057-anthropic-context-engineering.md) | "Smallest set of high-signal tokens" | Minimize context; maximize relevance |
| 058 | [Lita: Lightweight Agents](058-lita-lightweight-agents.md) | Simple agents reveal model capability; complex scaffolding obscures | Start simple; add complexity only when measured improvement |
| 059 | [Multi-Agent Orchestrator 2025](059-multi-agent-orchestrator-2025.md) | Orchestrator-worker pattern enables 10+ parallel agents | Track-based partitioning + file reservations prevent conflicts |
| 060 | [Debugging Decay Index](060-debugging-decay-index.md) | 60-80% capability loss within 2-3 attempts (DDI) | Hard stop at 3 iterations; fresh start or ADaPT decompose |
| 061 | [LLM Security 2025](061-llm-security-2025.md) | 45% of AI code fails OWASP Top 10; Java 72% failure rate | `ubs --staged` mandatory; never trust LLM code implicitly |
| 062 | [RAG for Repository Code](062-rag-repository-code.md) | Dual granularity (file + chunk) retrieval for codebase context | Use CASS/cm context for pattern retrieval before implementation |
| 063 | [Agentic SE 3.0](063-agentic-se-3.md) | SE 3.0: From code generation to autonomous goal-directed agents | Trust via verification (TDD + security + calibration) |
| 064 | [DeepCode: Open Agentic](064-deepcode-open-agentic.md) | Blueprint distillation + stateful memory for document→code | Distill specs before generation; maintain code memory |
| 065 | [Confucius Code Agent](065-confucius-code-agent.md) | Scalable agents for massive codebases via checkpoint + resume | Hierarchical navigation; session checkpoints at phase boundaries |
| 066 | [SWE-Bench Pro 2025](066-swe-bench-pro-2025.md) | Harder benchmarks reveal more about real capability | Don't trust benchmarks; verify everything on complex tasks |
| 067 | [Live-SWE-agent Evolution](067-live-swe-agent-evolution.md) | Agents that self-evolve scaffolds during runtime (+15% gain) | Tool creation + strategy adaptation; complement ADaPT |
| 068 | [GraphCodeAgent](068-graphcodeagent.md) | Dual graph (structure + dependency) for repo-level RAG | Graph navigation beats embedding search for multi-file |
| 069 | [AdaCoder](069-adacoder.md) | Adaptive planning based on model capability profiling | Weaker models need finer decomposition; profile before execute |
| 070 | [Structured CoT for Code](070-structured-cot-code.md) | Use program structures (SEQUENCE/BRANCH/LOOP) for reasoning | +9.7% on HumanEval; biggest gains on complex control flow |
| 071 | [Self-Correction via RL](071-self-correction-rl.md) | RL-trained correction > prompting; distribution mismatch is the key problem | External feedback (tests), not self-assessment; 3 tries then fresh start |
| 072 | [AI Code Review 2025](072-ai-code-review-2025.md) | 14.9% of PRs involve AI agents (14x growth); concise + code snippets = adoption | Two-stage (detect + verify); AI + human hybrid review |
| 073 | [MCP Tool Calling](073-mcp-tool-calling.md) | MCP standardizes tool calling; Code Mode reduces context 98.7% | Standardize tools via MCP; use Code Mode for complex workflows |

---

## Workflow Impact Analysis

For a comprehensive analysis of how this research should update our workflow, see:

**[2025-RESEARCH-UPDATE-ANALYSIS.md](../docs/workflow/2025-RESEARCH-UPDATE-ANALYSIS.md)**

This document maps each research paper to specific protocol updates with priority levels and cross-references.

---

## Key Takeaways

### What Works
1. **Plan-then-implement** - Consistent 2-10% improvements
2. **Test-based verification** - Both for training and debugging
3. **Retrieval + short context** - Better than stuffing long context
4. **Evidence-grounded debate** - Not unstructured back-and-forth
5. **Selective doc retrieval** - For rare APIs, not common ones
6. **TDD-first** (2025) - 45.97% pass@1 improvement; tests BEFORE implementation
7. **Iteration caps** (2025) - Max 3 repair attempts; security degrades with more
8. **Security scanning** (2025) - `ubs --staged` mandatory; ~40% of LLM code has vulnerabilities

### What Doesn't Work
1. **"Are you sure?" without new info** - Makes things worse
2. **Unstructured multi-agent debate** - Random walk, no convergence
3. **Long context for reasoning** - Semantic tasks suffer most
4. **Always-on doc retrieval** - Hurts performance on well-known APIs
5. **Unlimited repair loops** - Security degrades with each iteration
6. **Skipping verification** - Mistakes compound silently
7. **Working in unfamiliar domains without oversight** - Human understanding is leverage

### Context-Dependent (Read The Paper)
1. **Self-evolving agents** - Paper is about tool creation, not workflow changes
2. **The 85-94% degradation numbers** - Worst-case on synthetic tasks; real code is 30-50%
3. **Multi-agent coordination** - Paper's task type (QA) differs from planning
4. **Benchmark performance** - Static benchmarks don't predict real-world success; verify in context

---

## Key Structural Principles (Derived from Research)

These principles guide how we structure planning, decomposition, and coordination:

### 1. Adaptive Decomposition (ADaPT Pattern)
> "Decompose only when execution fails" — `research/038-adapt.md`

- **Do NOT** over-decompose upfront based on guesses about complexity
- **DO** start coarse, attempt execution, split only when reality demands it
- **Combined with:** 3-iteration cap (split after failure, don't keep retrying)

### 2. Test-Based Disagreement Resolution
> "Tests are the medium of disagreement, not rhetoric" — `research/041-debatecoder.md`
> "Voting alone beats extended debate" — `research/003-debate-or-vote.md`

- **Do NOT** engage in rhetorical cross-examination (it degrades outcomes)
- **DO** write discriminating tests when agents disagree
- **DO** let test results adjudicate conflicts

### 3. Minimal Viable Planning
> "The smallest set of high-signal tokens that enables correct behavior" — `research/057-anthropic-context-engineering.md`

- **Do NOT** create massive plans "just in case"
- **DO** match plan size to project complexity
- "Lossless" means no guessing, not "huge"

### 4. TDD-First Everywhere
> "TDD yields 45.97% pass@1 improvement" — `research/054-tdd-ai-code-gen.md`

- **Do NOT** write tests after implementation
- **DO** write tests in bead descriptions before any implementation code
- Tests define "done" and enable verification

---

## How To Read These Summaries

Each summary has:
- **Summary:** What the paper actually found, with numbers
- **Practical Implications:** What this means for building systems

The summaries avoid:
- Overstating findings based on abstracts
- Conflating different task types
- Treating worst-case numbers as typical

---

## Papers Not Yet Summarized

These were cited in initial searches but not read in full:

- Universal Self-Consistency (USC) for code generation selection
- HumanEval Pro / MBPP Pro follow-up analyses (if any)
- EG-CFG follow-up replications (if any)
- Various prompt engineering papers

Add to this folder as papers are read.
