# SpecRover: Code Intent Extraction via LLMs

**Paper:** SpecRover: Code Intent Extraction via LLMs
**URL:** https://arxiv.org/html/2408.02232v4
**Date:** August 2024 (latest revisions November 2024)
**Venue:** ICSE 2025

---

## Summary

Critical research making **specification inference** an explicit, first-class component of software engineering agents. SpecRover extends AutoCodeRover with multi-stage validation: reproducer generation, function intent summaries, patch review, and evidence-based selection.

**Key insight:** Agents need derived requirements/specifications, not just raw retrieval. Every fix must tie back to explicit evidence.

---

## The SpecRover Architecture

### Multi-Agent Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                      SPECROVER PIPELINE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Stage 1: REPRODUCER AGENT                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Input:  Issue description                                 │ │
│  │  Goal:   Generate minimal failing test                     │ │
│  │  Output: Reproducer test (when possible)                   │ │
│  │                                                             │ │
│  │  Success Rate: ~60% of issues                              │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  Stage 2: CONTEXT RETRIEVAL + SPEC INFERENCE                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ├─ Retrieve candidate locations (AutoCodeRover-style)     │ │
│  │  │                                                          │ │
│  │  ├─ For each function/class:                               │ │
│  │  │  ├─ Analyze intended behavior                           │ │
│  │  │  ├─ Document invariants                                 │ │
│  │  │  └─ Write natural language spec                         │ │
│  │  │                                                          │ │
│  │  └─ Output: Code + inferred specifications                 │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  Stage 3: PATCHING AGENT                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Input:  Localized code + specifications + reproducer      │ │
│  │  Goal:   Generate candidate patches                        │ │
│  │  Output: N candidate patches (N=3-5)                       │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  Stage 4: REVIEWER AGENT                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  For each patch:                                           │ │
│  │  ├─ Verify alignment with issue                            │ │
│  │  ├─ Check adherence to specifications                      │ │
│  │  ├─ Test against reproducer                                │ │
│  │  ├─ Assess side effects                                    │ │
│  │  └─ Generate evidence-based feedback                       │ │
│  │                                                             │ │
│  │  Output: Ranked patches with justifications                │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  Stage 5: SELECTION AGENT (When Needed)                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  When tests are weak:                                      │ │
│  │  ├─ Compare patches against inferred specs                 │ │
│  │  ├─ Weight reviewer feedback                               │ │
│  │  └─ Select highest-confidence patch                        │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Specification Inference

### What Is "Specification Inference"?

Instead of just retrieving code, SpecRover explicitly extracts **intended behavior**:

```python
# Retrieved Code
def authenticate(username, password):
    user = db.get_user(username)
    if user:
        return True
    return False

# Inferred Specification
"""
Function: authenticate(username, password)

INTENDED BEHAVIOR:
- Purpose: Verify user credentials and grant access
- Preconditions:
  * username must be non-null string
  * password must be non-null string
  * database connection must be established
- Postconditions:
  * Returns True if user exists AND password is correct
  * Returns False if user doesn't exist OR password is wrong
  * No side effects (read-only operation)

INVARIANTS:
- Security: Must verify password, not just user existence
- Performance: Single database query expected
- Error handling: Should not raise exceptions for invalid input

CURRENT VIOLATION:
- Missing password verification (security issue)
- Only checks if user exists, not if password matches
"""
```

### Specification Inference Algorithm

```
┌─────────────────────────────────────────────────────────────────┐
│              SPECIFICATION INFERENCE PROCESS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Static Analysis                                        │
│  ├── Extract function signature                                 │
│  ├── Identify parameters and return type                        │
│  ├── Parse docstring (if exists)                                │
│  └── Analyze AST for control flow                               │
│                                                                  │
│  Step 2: Dynamic Evidence                                       │
│  ├── Examine test cases (what's expected?)                      │
│  ├── Check call sites (how is it used?)                         │
│  ├── Review related code (similar functions)                    │
│  └── Analyze error messages (what can go wrong?)                │
│                                                                  │
│  Step 3: LLM Synthesis                                          │
│  ├── Combine all evidence                                       │
│  ├── Generate natural language specification                    │
│  ├── Identify preconditions/postconditions                      │
│  └── Document invariants                                        │
│                                                                  │
│  Step 4: Validation                                             │
│  ├── Check consistency with issue description                   │
│  ├── Verify against test expectations                           │
│  └── Flag uncertainty/assumptions                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Reproducer Test Generation

### The Power of Reproducers

| Without Reproducer | With Reproducer |
|-------------------|-----------------|
| Rely on weak existing tests | Targeted test that exposes bug |
| Hard to validate fix | Clear pass/fail signal |
| Ambiguous requirements | Concrete behavioral expectation |
| May miss regression | Captures exact issue |

### Reproducer Generation Process

```
Input: Issue Description
"User authentication succeeds even with wrong password"

↓

Reproducer Agent Analysis:
1. Identify bug type: Logic error (security)
2. Determine minimal test case
3. Generate test that SHOULD fail but currently passes

↓

Generated Reproducer Test:

```python
def test_authentication_rejects_wrong_password():
    """
    Reproducer for issue #1234: Authentication bypassed with wrong password.

    Expected behavior: authenticate() should return False when password is incorrect.
    Current behavior: Returns True for any existing username (BUG).
    """
    # Setup
    db.create_user("alice", password_hash("correct_password"))

    # The bug: This currently returns True but should return False
    result = authenticate("alice", "wrong_password")

    assert result == False, \
        "Authentication should fail with incorrect password"

    # Additional test: Verify correct password still works
    result = authenticate("alice", "correct_password")
    assert result == True, \
        "Authentication should succeed with correct password"
```

Success Rate: Generated reproducers in ~60% of cases
Impact: +15% pass rate when reproducer available
```

---

## Reviewer Agent: Evidence-Based Validation

### Review Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    PATCH REVIEW PROTOCOL                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  For each candidate patch:                                      │
│                                                                  │
│  Question 1: Does it address the issue?                         │
│  ├─ Evidence: Compare patch to issue description                │
│  ├─ Evidence: Check if reproducer now passes                    │
│  └─ Verdict: [YES/NO/PARTIAL] with explanation                  │
│                                                                  │
│  Question 2: Does it maintain specifications?                   │
│  ├─ Evidence: Compare against inferred specs                    │
│  ├─ Evidence: Check invariants preserved                        │
│  └─ Verdict: [YES/NO/PARTIAL] with explanation                  │
│                                                                  │
│  Question 3: Are there side effects?                            │
│  ├─ Evidence: Analyze scope of changes                          │
│  ├─ Evidence: Check for API contract changes                    │
│  └─ Verdict: [SAFE/RISKY/UNSAFE] with explanation               │
│                                                                  │
│  Question 4: Does it pass all tests?                            │
│  ├─ Evidence: Run existing test suite                           │
│  ├─ Evidence: Run reproducer test                               │
│  └─ Verdict: [PASS/FAIL] with test results                      │
│                                                                  │
│  Final Scoring:                                                  │
│  score = (addresses_issue × 10) +                               │
│          (maintains_specs × 8) +                                │
│          (no_side_effects × 6) +                                │
│          (passes_tests × 10)                                    │
│                                                                  │
│  Output: Structured review with evidence citations              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example Review Output

```markdown
## Patch Review: Candidate A

### 1. Addresses Issue? YES (10/10)
Evidence:
- Issue states: "Authentication succeeds with wrong password"
- Patch adds: `and verify_password(user, password)` check
- Reproducer test now PASSES
- Root cause directly addressed

### 2. Maintains Specifications? YES (8/8)
Evidence:
- Inferred spec requires password verification ✓
- Postcondition "Returns True if password correct" now met ✓
- No invariants violated ✓
- Security requirement addressed ✓

### 3. Side Effects? SAFE (6/6)
Evidence:
- Change limited to single function
- No API contract changes
- Backward compatible (stricter, not looser)
- Call sites unaffected (return type unchanged)

### 4. Passes Tests? PASS (10/10)
Evidence:
- Existing tests: 487/487 pass ✓
- Reproducer test: PASS ✓
- No regressions introduced

**TOTAL SCORE: 34/34**

Recommendation: ACCEPT
Confidence: HIGH
```

---

## Performance Results

### SWE-bench Results

| System | SWE-bench (full) | SWE-bench Lite | Avg Cost |
|--------|------------------|----------------|----------|
| **SpecRover** | **19.0%** | **31.2%** | ~$2.50 |
| AutoCodeRover | 10.5% | 19.8% | ~$1.00 |
| Agentless | 14.2% | 27.3% | ~$0.80 |
| SWE-agent | 12.5% | 18.5% | ~$4.00 |

**Key insight:** +50% improvement over AutoCodeRover by adding specification inference and review.

### Reviewer Precision

```
Reviewer Prediction vs Actual Correctness
(SWE-bench Lite, N=300)

                Predicted    Predicted
                Correct      Incorrect
    ┌──────────┬───────────┬──────────┐
Actually   │          │           │          │
Correct    │    47    │     6     │   53   │
           │          │           │          │
    ├──────────┼───────────┼──────────┤
Actually   │          │           │          │
Incorrect  │    17    │    230    │  247   │
           │          │           │          │
    └──────────┴───────────┴──────────┘
              64        236       300

Precision: 47/64 = 73.4%
Recall:    47/53 = 88.7%
Accuracy:  277/300 = 92.3%
```

**Interpretation:** When reviewer says "accept," it's correct 73% of the time. Strong filter for bad patches (94% rejected when incorrect).

---

## Cost-Benefit Analysis

### Why More Stages Can Mean Lower Total Cost

```
Scenario: Fix authentication bug

Approach 1: Direct Patch (No Spec Inference)
├─ Attempt 1: Wrong location           ($0.50)
├─ Attempt 2: Incomplete fix           ($0.50)
├─ Attempt 3: Breaks other tests       ($0.50)
├─ Attempt 4: Manual debugging         ($0.50)
└─ Total: $2.00, 4 iterations, STILL BROKEN

Approach 2: SpecRover (With Spec Inference)
├─ Reproducer generation               ($0.40)
├─ Spec inference                      ($0.60)
├─ Patch generation (3 candidates)     ($0.90)
├─ Reviewer validation                 ($0.40)
├─ Selection                            ($0.20)
└─ Total: $2.50, 1 iteration, FIXED

ROI Analysis:
- 25% higher upfront cost
- 75% fewer iterations
- 4x higher success rate
- Net: 2x more cost-effective
```

---

## Implementation Guide

### Spec Inference Code

```python
class SpecificationInference:
    """
    Infers intended behavior from code, tests, and context.
    """

    def __init__(self, function_info, issue_text):
        self.func = function_info
        self.issue = issue_text

    def infer_specification(self):
        """Generate natural language specification."""

        # Gather evidence
        evidence = self._gather_evidence()

        # Synthesize specification
        spec = self._synthesize_spec(evidence)

        # Validate against issue
        spec = self._validate_spec(spec)

        return spec

    def _gather_evidence(self):
        """Collect all available evidence about intended behavior."""
        evidence = {
            'static': self._static_analysis(),
            'dynamic': self._dynamic_analysis(),
            'contextual': self._contextual_analysis()
        }
        return evidence

    def _static_analysis(self):
        """Analyze function signature and implementation."""
        return {
            'signature': {
                'name': self.func['name'],
                'params': self.func['params'],
                'return_type': self.func['return_type']
            },
            'docstring': self.func.get('docstring', ''),
            'control_flow': analyze_control_flow(self.func['ast']),
            'complexity': calculate_complexity(self.func['ast'])
        }

    def _dynamic_analysis(self):
        """Analyze how function is tested and used."""
        return {
            'test_cases': self._find_test_cases(),
            'call_sites': self._find_call_sites(),
            'error_handling': self._analyze_error_handling()
        }

    def _contextual_analysis(self):
        """Analyze related code and patterns."""
        return {
            'related_functions': self._find_related_functions(),
            'similar_implementations': self._find_similar_code(),
            'issue_mentions': self._extract_issue_requirements()
        }

    def _synthesize_spec(self, evidence):
        """Use LLM to synthesize specification from evidence."""
        prompt = f"""
        Infer the intended behavior specification for this function.

        Function: {evidence['static']['signature']}

        Static Evidence:
        - Docstring: {evidence['static']['docstring']}
        - Control flow: {evidence['static']['control_flow']}

        Dynamic Evidence:
        - Test cases: {evidence['dynamic']['test_cases']}
        - Usage: {evidence['dynamic']['call_sites']}

        Contextual Evidence:
        - Related functions: {evidence['contextual']['related_functions']}
        - Issue description: {self.issue}

        Generate a specification including:
        1. PURPOSE: What is this function supposed to do?
        2. PRECONDITIONS: What must be true before calling?
        3. POSTCONDITIONS: What must be true after calling?
        4. INVARIANTS: What must always hold?
        5. CURRENT_VIOLATIONS: Based on the issue, what's broken?

        Format as structured markdown.
        """

        spec = llm.call(prompt)
        return spec

    def _validate_spec(self, spec):
        """Check if specification is consistent with evidence."""
        prompt = f"""
        Review this inferred specification for consistency.

        Specification:
        {spec}

        Issue Description:
        {self.issue}

        Questions:
        1. Does the spec align with the issue description?
        2. Are there contradictions in the spec?
        3. Are preconditions/postconditions realistic?
        4. Is the violation correctly identified?

        Return: [VALID/INVALID] with corrections if needed.
        """

        validation = llm.call(prompt)

        if validation.startswith('INVALID'):
            # Refine spec based on feedback
            spec = self._refine_spec(spec, validation)

        return spec
```

### Reviewer Agent Code

```python
class ReviewerAgent:
    """
    Reviews patches against specifications and issue requirements.
    """

    def __init__(self, issue, spec, reproducer_test):
        self.issue = issue
        self.spec = spec
        self.reproducer = reproducer_test

    def review_patch(self, patch):
        """Conduct comprehensive evidence-based review."""

        review = {
            'addresses_issue': self._check_addresses_issue(patch),
            'maintains_specs': self._check_maintains_specs(patch),
            'side_effects': self._check_side_effects(patch),
            'test_results': self._run_tests(patch)
        }

        # Calculate score
        score = (
            review['addresses_issue']['score'] * 10 +
            review['maintains_specs']['score'] * 8 +
            review['side_effects']['score'] * 6 +
            review['test_results']['score'] * 10
        )

        review['total_score'] = score
        review['recommendation'] = 'ACCEPT' if score >= 30 else 'REJECT'
        review['confidence'] = self._calculate_confidence(review)

        return review

    def _check_addresses_issue(self, patch):
        """Verify patch addresses the reported issue."""
        prompt = f"""
        Issue: {self.issue}

        Patch:
        {patch}

        Does this patch address the issue?

        Provide:
        - Verdict: YES/NO/PARTIAL
        - Score: 0-1
        - Evidence: Specific lines/changes that address issue
        - Concerns: What's missing or problematic
        """

        response = llm.call(prompt)
        return self._parse_review_response(response)

    def _check_maintains_specs(self, patch):
        """Verify patch maintains inferred specifications."""
        prompt = f"""
        Specification:
        {self.spec}

        Patch:
        {patch}

        Does this patch maintain the specification?

        Check:
        1. Preconditions still enforced?
        2. Postconditions still guaranteed?
        3. Invariants preserved?
        4. Violations corrected?

        Provide verdict, score (0-1), and evidence.
        """

        response = llm.call(prompt)
        return self._parse_review_response(response)

    def _check_side_effects(self, patch):
        """Assess safety of patch changes."""
        analysis = analyze_patch_scope(patch)

        risk_factors = {
            'files_modified': len(analysis['files']),
            'functions_changed': len(analysis['functions']),
            'api_changes': analysis['has_api_changes'],
            'scope_expansion': analysis['changes_outside_localized_area']
        }

        # Risk scoring
        if risk_factors['api_changes']:
            risk = 'UNSAFE'
            score = 0.0
        elif risk_factors['scope_expansion']:
            risk = 'RISKY'
            score = 0.5
        else:
            risk = 'SAFE'
            score = 1.0

        return {
            'verdict': risk,
            'score': score,
            'evidence': risk_factors
        }

    def _run_tests(self, patch):
        """Execute tests against patched code."""
        with apply_patch_context(patch) as patched_repo:
            results = {
                'existing_tests': run_test_suite(patched_repo),
                'reproducer_test': run_test(patched_repo, self.reproducer)
            }

            if results['existing_tests']['pass'] and results['reproducer_test']['pass']:
                verdict = 'PASS'
                score = 1.0
            elif results['reproducer_test']['pass']:
                verdict = 'PARTIAL'  # Fixes issue but breaks other tests
                score = 0.5
            else:
                verdict = 'FAIL'
                score = 0.0

            return {
                'verdict': verdict,
                'score': score,
                'evidence': results
            }

    def _calculate_confidence(self, review):
        """Estimate confidence in recommendation."""
        # High confidence if:
        # - Strong evidence across all dimensions
        # - Clear test results
        # - No ambiguities

        ambiguity_count = sum([
            review['addresses_issue'].get('verdict') == 'PARTIAL',
            review['maintains_specs'].get('verdict') == 'PARTIAL',
            review['side_effects'].get('verdict') == 'RISKY'
        ])

        if ambiguity_count == 0 and review['test_results']['verdict'] == 'PASS':
            return 'HIGH'
        elif ambiguity_count <= 1:
            return 'MEDIUM'
        else:
            return 'LOW'
```

---

## Integration with K&V Workflow

### SpecRover Stages as Beads

```yaml
# Stage 1: Reproducer Generation
- id: bd-301
  title: "Generate reproducer test"
  phase: investigation
  deliverable: "Failing test that exposes bug (if possible)"
  success_criteria: "Test fails on current code, will pass when fixed"

# Stage 2: Specification Inference
- id: bd-302
  title: "Infer intended behavior specifications"
  phase: investigation
  depends: [bd-301]
  deliverable: "Natural language specs for affected functions"
  success_criteria: "Specs consistent with issue and test expectations"

# Stage 3: Patch Generation
- id: bd-303
  title: "Generate candidate patches"
  phase: execution
  depends: [bd-302]
  deliverable: "3-5 candidate patches with strategies"
  success_criteria: "Each patch targets root cause from spec analysis"

# Stage 4: Patch Review
- id: bd-304
  title: "Review patches against specs and issue"
  phase: verification
  depends: [bd-303]
  deliverable: "Evidence-based reviews for each patch"
  success_criteria: "Clear recommendation with supporting evidence"

# Stage 5: Selection
- id: bd-305
  title: "Select best patch"
  phase: verification
  depends: [bd-304]
  deliverable: "Single validated patch"
  success_criteria: "Passes all tests, addresses issue, maintains specs"
```

---

## Key Takeaways

1. **Make specifications explicit** — Don't assume models understand intent; derive it systematically
2. **Reproducers are force multipliers** — 60% coverage gives 15% performance boost
3. **Review with evidence, not vibes** — Every claim must cite code/test/doc
4. **Multi-stage validation pays off** — Higher upfront cost, much higher success rate
5. **Specs enable better debugging** — When patch fails, spec tells you why
6. **Reviewer precision matters** — 73% precision means 3:1 good:bad ratio
7. **Track violations explicitly** — Current behavior vs intended behavior diff

---

## See Also

- `011-agentless.md` — Pipeline architecture and validation
- `012-autocoderover.md` — Context retrieval foundation
- `014-codeplan.md` — Multi-step edit planning
- `018-livecodebench.md` — Self-repair evaluation
- `038-adapt.md` — Adaptive decomposition when specs are unclear
