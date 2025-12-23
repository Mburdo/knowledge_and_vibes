---
title: Agentless
tags: [agents, swe-bench, pipeline, localization, simplicity]
protocols: [P8]
---

# Agentless: Demystifying SWE Agents

**Paper:** Agentless: Demystifying LLM-based Software Engineering Agents
**URL:** https://arxiv.org/html/2407.01489v2
**Date:** July 2024
**Venue:** arXiv preprint

---

## Summary

Groundbreaking research demonstrating that **complex autonomous agents aren't necessary** for strong SWE-bench performance. Instead, a simple, interpretable three-stage pipeline achieves state-of-the-art results at <$1 per issue.

**Key insight:** Fixed scaffolding with hierarchical localization outperforms open-ended tool-using agents.

---

## The Agentless Pipeline

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      AGENTLESS PIPELINE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: LOCALIZATION (Hierarchical Narrowing)                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Level 1: File Localization                                │ │
│  │  ├── Input: Issue description + repository structure       │ │
│  │  ├── Output: Ranked list of suspicious files               │ │
│  │  └── Method: Semantic similarity + keyword matching        │ │
│  │                                                             │ │
│  │  Level 2: Class/Function Localization                      │ │
│  │  ├── Input: Suspicious files + issue description           │ │
│  │  ├── Output: Specific symbols (classes/functions)          │ │
│  │  └── Method: AST-based analysis + relevance scoring        │ │
│  │                                                             │ │
│  │  Level 3: Edit Location Selection                          │ │
│  │  ├── Input: Target symbols + surrounding context           │ │
│  │  ├── Output: Precise line ranges for editing               │ │
│  │  └── Method: Contextual analysis + edit span prediction    │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Phase 2: REPAIR (Constrained Patch Generation)                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ├── Generate N candidate patches (N=3-5 typical)          │ │
│  │  ├── Use Search/Replace diff format (reduces errors)       │ │
│  │  ├── Constrain edits to localized regions only             │ │
│  │  └── Multiple strategies per issue (coverage)              │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Phase 3: VALIDATION (Test-Based Selection)                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ├── Generate reproducer test from issue (when possible)   │ │
│  │  ├── Apply each candidate patch                            │ │
│  │  ├── Run test suite + reproducer                           │ │
│  │  ├── Filter: Keep only patches that pass                   │ │
│  │  └── Rerank: Use test coverage + confidence scores         │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Why Agentless Works

### Attacking Key Failure Modes

| Failure Mode | Traditional Agent | Agentless Solution |
|--------------|-------------------|-------------------|
| **Wrong location** | Agent searches entire repo, gets lost | Hierarchical narrowing guarantees focus |
| **Patch application fails** | Free-form diffs don't apply cleanly | Search/Replace format validates automatically |
| **Overcorrection** | Agent makes broad, risky changes | Edits constrained to localized spans |
| **Low patch quality** | Single attempt, no alternatives | Multiple candidates + test-based selection |
| **Tool confusion** | Agent picks wrong tools/order | Fixed pipeline, no decisions needed |

### The Hierarchical Localization Advantage

```
Traditional Approach:
Repository (100,000 lines)
    ↓
LLM searches everything
    ↓
Overwhelmed/lost
    ↓
Wrong file or gives up

Agentless Approach:
Repository (100,000 lines)
    ↓
Level 1: Narrow to 5-10 files
    ↓
Level 2: Narrow to 2-3 functions
    ↓
Level 3: Narrow to specific line ranges
    ↓
Focused, high-confidence edit
```

---

## Performance Results

### SWE-bench Lite Performance

| System | Pass@1 | Avg Cost | Architecture |
|--------|--------|----------|--------------|
| Agentless | **27.3%** | **<$1** | Fixed pipeline |
| SWE-agent | 18.5% | ~$4 | Autonomous agent |
| AutoCodeRover | 19.8% | ~$1 | Semi-autonomous |
| Baseline (GPT-4) | 8.2% | ~$2 | Direct patching |

### Cost-Effectiveness Curve

```
Performance
vs Cost
  │
30├─────────●  Agentless ($0.80)
  │
25├─────────
  │
20├──────────────●  AutoCodeRover ($0.95)
  │      ●  SWE-agent ($4.20)
15├──────
  │
10├────────────────●  Baseline ($2.10)
  │
 0└────────────────────────────────────
  $0   $1   $2   $3   $4   $5
              Average Cost per Issue
```

---

## The Search/Replace Diff Format

### Why It Matters

Traditional unified diffs fail when:
- Line numbers change (concurrent edits)
- Whitespace differs
- Context lines mismatch

Search/Replace format:
```python
<<<<<<< SEARCH
def authenticate(user, password):
    if user in database:
        return True
=======
def authenticate(user, password):
    if user in database and verify_password(user, password):
        return True
>>>>>>> REPLACE
```

**Advantages:**
- Exact match validation (fails fast if context changed)
- Self-documenting (shows what was replaced)
- Idempotent (safe to re-apply)
- Easy to validate before applying

---

## Multiple Candidate Strategy

### Why Multiple Patches?

```
Single Patch Approach:
┌──────────────┐
│ Generate     │
│ Best Patch   │──► Success: 27%
└──────────────┘    Failure: 73%

Multiple Candidates (N=3):
┌──────────────┐
│ Patch A      │──► Pass tests: 18%
├──────────────┤
│ Patch B      │──► Pass tests: 15%  } Combined: 45%
├──────────────┤
│ Patch C      │──► Pass tests: 12%
└──────────────┘
```

### Selection Strategy

```
For each candidate patch:
  1. Apply to clean checkout
  2. Run existing test suite
  3. Run generated reproducer test
  4. Measure test coverage change
  5. Score = (tests_pass × 10) + coverage_delta

Select highest scoring patch that:
  - Passes all tests
  - Increases or maintains coverage
  - Applies cleanly
```

---

## Reproducer Test Generation

### The Power of Reproducers

| Test Type | Discriminative Power | Availability |
|-----------|---------------------|--------------|
| Existing suite | Low (often weak) | 100% |
| Generated reproducer | High (targets bug) | ~60% |
| Combined | Very high | 100% |

### Example Reproducer

```python
# Issue: "User authentication fails for valid credentials"

# Generated reproducer test
def test_authentication_with_valid_credentials():
    """
    Tests that valid user credentials are accepted.

    Bug: System was only checking if user exists, not verifying password.
    """
    user = User(username="alice", password_hash=hash_password("secret123"))
    database.add(user)

    # This should pass but currently fails
    result = authenticate("alice", "secret123")
    assert result == True, "Valid credentials should authenticate"

    # This should fail but currently passes (the bug!)
    result = authenticate("alice", "wrong_password")
    assert result == False, "Invalid password should be rejected"
```

**Impact:** Reproducers catch bugs that weak test suites miss.

---

## Critical Limitations

### What Agentless Can't Do

| Limitation | Impact | Mitigation |
|------------|--------|------------|
| **Requires localization** | Fails if can't find bug location | Better search/retrieval needed |
| **Weak test suites** | Can't validate fixes properly | Generate reproducers |
| **Multi-step changes** | Pipeline assumes single edit site | Not addressed |
| **Requires clear issue** | Vague requests fail | Human clarification |
| **No learning/adaptation** | Same pipeline every time | Template customization |

### Benchmark Artifacts

Agentless paper itself identifies SWE-bench Lite problems:

```
Issues with Benchmark:
├── Solution leakage: 23% of tasks
├── Underspecified: 18% of tasks
├── Ambiguous success criteria: 15%
└── Infrastructure issues: 12%

Result: Apparent success may be inflated
```

---

## Integration with K&V Workflow

### Mapping to Bead Phases

| Agentless Phase | K&V Equivalent | Implementation |
|-----------------|----------------|----------------|
| Localization L1 | Investigation bead | File-level search |
| Localization L2 | Investigation bead | Symbol-level analysis |
| Localization L3 | Planning bead | Edit location selection |
| Repair | Execution bead | Generate candidate patches |
| Validation | Verification bead | Test-based selection |

### Example K&V Bead Chain

```yaml
# Investigation Bead (Localization)
- id: bd-101
  title: "Locate authentication bug"
  phase: investigation
  deliverable: "List of suspicious files and functions"

  # Planning Bead
- id: bd-102
  title: "Plan authentication fix"
  phase: planning
  depends: [bd-101]
  deliverable: "Edit locations + repair strategies"

  # Execution Bead
- id: bd-103
  title: "Generate fix candidates"
  phase: execution
  depends: [bd-102]
  deliverable: "3 candidate patches in Search/Replace format"

  # Verification Bead
- id: bd-104
  title: "Validate and select fix"
  phase: verification
  depends: [bd-103]
  deliverable: "Tested, validated patch"
```

---

## Practical Implementation Guide

### Code Example: Hierarchical Localization

```python
class AgentlessLocalizer:
    def __init__(self, repo_path, issue_text):
        self.repo = Repository(repo_path)
        self.issue = issue_text

    def localize(self):
        """Three-level hierarchical localization."""

        # Level 1: File-level
        files = self._localize_files()
        print(f"Level 1: Found {len(files)} suspicious files")

        # Level 2: Symbol-level
        symbols = self._localize_symbols(files)
        print(f"Level 2: Found {len(symbols)} suspicious symbols")

        # Level 3: Edit spans
        edit_locations = self._localize_edit_spans(symbols)
        print(f"Level 3: Identified {len(edit_locations)} edit locations")

        return edit_locations

    def _localize_files(self):
        """Level 1: File localization using semantic similarity."""
        candidates = []

        for file_path in self.repo.get_all_files():
            # Get file summary
            summary = self._get_file_summary(file_path)

            # Compute relevance score
            score = semantic_similarity(self.issue, summary)
            score += keyword_overlap(self.issue, summary) * 0.3

            candidates.append((file_path, score))

        # Return top 10 files
        candidates.sort(key=lambda x: x[1], reverse=True)
        return [f for f, s in candidates[:10]]

    def _localize_symbols(self, files):
        """Level 2: Class/function localization."""
        symbols = []

        for file_path in files:
            # Parse AST
            tree = parse_ast(file_path)

            # Extract all classes and functions
            for node in ast.walk(tree):
                if isinstance(node, (ast.ClassDef, ast.FunctionDef)):
                    # Score this symbol
                    context = self._get_symbol_context(node, tree)
                    score = semantic_similarity(self.issue, context)

                    symbols.append({
                        'file': file_path,
                        'name': node.name,
                        'type': type(node).__name__,
                        'lineno': node.lineno,
                        'score': score
                    })

        # Return top 5 symbols
        symbols.sort(key=lambda x: x['score'], reverse=True)
        return symbols[:5]

    def _localize_edit_spans(self, symbols):
        """Level 3: Precise line range selection."""
        edit_locations = []

        for symbol in symbols:
            # Load full function/class
            code = self._load_symbol_code(symbol)

            # Analyze for edit location
            prompt = f"""
            Issue: {self.issue}

            Code:
            {code}

            Identify the exact line range that needs to be edited.
            Return start_line and end_line.
            """

            response = llm.call(prompt)
            edit_locations.append({
                **symbol,
                'edit_start': response.start_line,
                'edit_end': response.end_line
            })

        return edit_locations
```

### Code Example: Multi-Candidate Repair

```python
class AgentlessRepair:
    def __init__(self, edit_location):
        self.location = edit_location

    def generate_candidates(self, num_candidates=3):
        """Generate multiple candidate patches."""
        candidates = []

        strategies = [
            "minimal_fix",      # Smallest possible change
            "defensive_fix",    # Add error handling
            "comprehensive_fix" # Address root cause
        ]

        for strategy in strategies[:num_candidates]:
            prompt = f"""
            Strategy: {strategy}

            Issue: {self.location['issue']}

            Code to edit:
            {self.location['code']}

            Generate a patch using Search/Replace format.
            """

            patch = llm.call(prompt)
            candidates.append({
                'strategy': strategy,
                'patch': patch,
                'location': self.location
            })

        return candidates

    def validate_and_select(self, candidates):
        """Test candidates and select best."""
        results = []

        for candidate in candidates:
            # Apply patch to clean checkout
            with clean_checkout() as repo:
                try:
                    apply_patch(repo, candidate['patch'])

                    # Run tests
                    existing_pass = run_existing_tests(repo)
                    reproducer_pass = run_reproducer_test(repo)
                    coverage = measure_coverage(repo)

                    score = (
                        existing_pass * 10 +
                        reproducer_pass * 5 +
                        coverage
                    )

                    results.append({
                        **candidate,
                        'existing_pass': existing_pass,
                        'reproducer_pass': reproducer_pass,
                        'coverage': coverage,
                        'score': score
                    })

                except PatchApplicationError:
                    # Patch didn't apply cleanly
                    results.append({
                        **candidate,
                        'score': -1,
                        'error': 'patch_application_failed'
                    })

        # Select highest scoring valid patch
        valid = [r for r in results if r['score'] > 0]
        if not valid:
            return None

        valid.sort(key=lambda x: x['score'], reverse=True)
        return valid[0]
```

---

## Key Takeaways

1. **Fixed pipelines can beat autonomous agents** — When problem structure is known, hard-coded scaffolding wins
2. **Hierarchical localization is critical** — Don't search everything; narrow systematically
3. **Multiple candidates + selection > single attempt** — Diversity through strategy variation
4. **Constrained formats reduce errors** — Search/Replace diffs validate automatically
5. **Reproducers are powerful discriminators** — Generated tests catch bugs weak suites miss
6. **Simple is interpretable** — Fixed pipeline makes debugging/improvement easier
7. **Cost matters** — $0.80 vs $4.20 per issue is 5x difference at scale

---

## When to Use Agentless-Style Approach

### Good Fit

- Single-file or well-localized bugs
- Clear issue descriptions
- Reasonable test coverage exists
- Cost sensitivity (high volume)
- Need for interpretability

### Poor Fit

- Multi-step refactors
- Vague requirements
- No existing tests
- Novel problem types (no template)
- Learning/adaptation needed

---

## See Also

- `012-autocoderover.md` — Structure-aware code search and retrieval
- `013-specrover.md` — Specification inference for validation
- `014-codeplan.md` — Multi-step edit planning
- `022-chatrepair.md` — Iterative repair approaches
- `060-debugging-decay-index.md` — Why multiple candidates help (hedge against decay)
- `016-res-q.md` — Repository-scale editing benchmark
