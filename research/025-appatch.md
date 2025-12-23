# APPATCH: Automated Adaptive Prompting for Vulnerability Patching

**Paper:** APPATCH: Automated Adaptive Prompting Large Language Models for Real-World Software Vulnerability Patching
**URL:** https://arxiv.org/html/2408.13597v1
**Date:** August 2024
**Venue:** arXiv preprint

---

## Summary

Security-focused program repair system combining **program analysis** for context reduction, **adaptive few-shot prompting** based on root cause similarity, and **multi-faceted validation** to reduce hallucinated fixes. Patches **7/11 zero-day vulnerabilities** in evaluation.

**Key innovation:** Don't just ask for a patch—constrain context with analysis, guide with root-cause-matched exemplars, then validate aggressively. Security requires higher rigor than functional bugs.

---

## The Security Patching Challenge

### Why Vulnerability Patching is Harder Than Bug Fixing

```
┌─────────────────────────────────────────────────────────────────┐
│         FUNCTIONAL BUG vs SECURITY VULNERABILITY                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FUNCTIONAL BUG:                                                 │
│  ├── Has reproducer (failing test)                              │
│  ├── Clear error message                                        │
│  ├── Test suite validates fix                                   │
│  └── Behavioral change is goal                                  │
│                                                                  │
│  SECURITY VULNERABILITY:                                         │
│  ├── Often NO reproducer (exploit not public)                   │
│  ├── No error message (silent security hole)                    │
│  ├── Tests may not cover vulnerability                          │
│  ├── Must preserve ALL existing behavior                        │
│  └── False fix can be worse than no fix                         │
│                                                                  │
│  Result: Much harder validation signal                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Validation Challenge

| Aspect | Bug Fix | Vulnerability Patch |
|--------|---------|-------------------|
| **Test signal** | Strong (failing test) | Weak (no exploit test) |
| **Correctness** | Tests pass | Must prove security property |
| **Behavioral change** | Acceptable | Minimal (preserve functionality) |
| **False positive cost** | Wasted effort | Security theater (dangerous) |
| **Validation** | Run tests | Analysis + tests + review |

---

## APPATCH Architecture

### Three-Stage Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    APPATCH PIPELINE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Stage 1: CONTEXT REDUCTION (Program Analysis)                  │
│  ┌────────────────────────────────────────────────┐             │
│  │ Input: Full codebase + CVE description         │             │
│  │                                                │             │
│  │ Analysis:                                      │             │
│  │ ├── Build program dependence graph (PDG)      │             │
│  │ ├── Compute vulnerability slice                │             │
│  │ └── Extract semantic context                   │             │
│  │                                                │             │
│  │ Output: Minimal vulnerability-relevant code    │             │
│  └─────────────────┬──────────────────────────────┘             │
│                    │                                             │
│                    ↓                                             │
│  Stage 2: ADAPTIVE PROMPTING (Root Cause Matching)               │
│  ┌────────────────────────────────────────────────┐             │
│  │ Input: Vulnerability slice + CVE                │             │
│  │                                                │             │
│  │ Steps:                                         │             │
│  │ 1. Infer root cause category                  │             │
│  │    (buffer overflow, injection, race, etc.)   │             │
│  │ 2. Retrieve similar vulnerabilities from pool │             │
│  │ 3. Build few-shot prompt with CoT              │             │
│  │ 4. Generate patch                              │             │
│  │                                                │             │
│  │ Output: Candidate patch + reasoning            │             │
│  └─────────────────┬──────────────────────────────┘             │
│                    │                                             │
│                    ↓                                             │
│  Stage 3: MULTI-FACETED VALIDATION                               │
│  ┌────────────────────────────────────────────────┐             │
│  │ Validation layers:                             │             │
│  │ ├── Syntax check                               │             │
│  │ ├── Test suite (if available)                  │             │
│  │ ├── Semantic equivalence check                 │             │
│  │ ├── Security property verification             │             │
│  │ └── LLM-based review (multi-model)             │             │
│  │                                                │             │
│  │ Output: Validated patch or rejection           │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Stage 1: Semantics-Aware Context Scoping

### Program Dependence Graph (PDG)

```
Original Code (100s of lines):
├── Module imports
├── Helper functions
├── Vulnerable function
├── Utility functions
└── Tests

         ↓ PDG analysis + slicing

Vulnerability Slice (10s of lines):
├── Vulnerable function
├── Direct data dependencies
├── Control flow dependencies
└── Key type definitions

Result: 90% context reduction, preserving all relevant semantics
```

### Slicing Algorithm

```python
def extract_vulnerability_slice(code, vulnerability_location):
    """
    Extract minimal code slice relevant to vulnerability.
    """
    # Build program dependence graph
    pdg = build_pdg(code)

    # Start from vulnerability location
    vuln_node = pdg.get_node(vulnerability_location)

    # Backward slice: all nodes that affect vulnerable code
    backward_deps = pdg.backward_slice(vuln_node)

    # Forward slice: all nodes affected by vulnerable code
    forward_deps = pdg.forward_slice(vuln_node)

    # Combine to get vulnerability semantics
    vuln_slice = backward_deps.union(forward_deps)

    # Extract code for slice
    return extract_code(vuln_slice)
```

### Example: SQL Injection

```python
# FULL CONTEXT (200 lines, overwhelming)
class UserController:
    def __init__(self, db):
        self.db = db
        self.logger = Logger()
        self.cache = Cache()
        # ... 50 more lines of setup

    def get_user(self, username):
        query = f"SELECT * FROM users WHERE name = '{username}'"  # Vuln!
        return self.db.execute(query)

    def get_all_users(self):
        # ... 30 lines

    def update_user(self, user_id, data):
        # ... 40 lines

    # ... 10 more methods, 100+ lines

# VULNERABILITY SLICE (15 lines, focused)
class UserController:
    def __init__(self, db):
        self.db = db

    def get_user(self, username):
        query = f"SELECT * FROM users WHERE name = '{username}'"  # Vuln!
        return self.db.execute(query)

# Slicing removed irrelevant methods while preserving:
# - db dependency
# - query construction
# - execution context
```

---

## Stage 2: Root Cause-Based Exemplar Retrieval

### Vulnerability Taxonomy

APPATCH categorizes vulnerabilities by root cause:

| Category | Example | Fix Pattern |
|----------|---------|-------------|
| **Input validation** | SQL injection, XSS | Sanitize/validate input |
| **Buffer management** | Buffer overflow | Bounds checking |
| **Resource management** | Use-after-free | Lifetime tracking |
| **Concurrency** | Race condition | Synchronization |
| **Cryptography** | Weak hashing | Use secure primitives |
| **Access control** | Auth bypass | Permission checks |

### Adaptive Exemplar Selection

```python
def select_exemplars(vulnerability, exemplar_pool):
    """
    Select exemplars based on root cause similarity.
    """
    # Infer root cause category
    root_cause = infer_root_cause(
        vulnerability.description,
        vulnerability.code_slice
    )

    # Filter pool by category
    category_matches = [
        ex for ex in exemplar_pool
        if ex.root_cause == root_cause
    ]

    # Rank by similarity within category
    similarities = compute_similarity(
        vulnerability,
        category_matches
    )

    # Select top-k diverse examples
    return select_diverse(similarities, k=3)
```

### Example Prompt with Exemplars

```
EXEMPLAR 1: SQL Injection
Root Cause: Unvalidated user input in query
Vulnerable Code:
  query = "SELECT * FROM users WHERE id = " + user_id
Fix Strategy: Use parameterized queries
Fixed Code:
  query = "SELECT * FROM users WHERE id = ?"
  cursor.execute(query, (user_id,))
Validation: Prevented injection in security scan

EXEMPLAR 2: Command Injection
Root Cause: Unvalidated user input in system call
Vulnerable Code:
  os.system("ls " + user_dir)
Fix Strategy: Input validation + safe API
Fixed Code:
  if re.match(r'^[a-zA-Z0-9_/]+$', user_dir):
      os.listdir(user_dir)
Validation: Prevented injection in fuzzing

EXEMPLAR 3: XSS
Root Cause: Unvalidated user input in HTML
Vulnerable Code:
  html = "<div>" + user_content + "</div>"
Fix Strategy: HTML escaping
Fixed Code:
  html = "<div>" + html.escape(user_content) + "</div>"
Validation: XSS scanner clean

NOW FIX: [New vulnerability with similar pattern]
```

---

## Stage 3: Multi-Faceted Validation

### Validation Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                   VALIDATION PIPELINE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Layer 1: SYNTAX CHECK                                           │
│  ├── Parse patched code                                          │
│  ├── Type checking (if typed language)                           │
│  └── REJECT if syntax invalid                                    │
│                                                                  │
│  Layer 2: TEST SUITE                                             │
│  ├── Run existing tests                                          │
│  ├── REJECT if tests fail                                        │
│  └── NOTE: May not cover vulnerability                           │
│                                                                  │
│  Layer 3: SEMANTIC EQUIVALENCE                                   │
│  ├── Compare behavior on non-exploit inputs                      │
│  ├── REJECT if behavior changes                                  │
│  └── Ensures behavioral preservation                             │
│                                                                  │
│  Layer 4: SECURITY PROPERTY VERIFICATION                         │
│  ├── Static analysis for vulnerability patterns                  │
│  ├── Symbolic execution (if applicable)                          │
│  ├── Fuzzing (if time permits)                                   │
│  └── REJECT if vulnerability still present                       │
│                                                                  │
│  Layer 5: LLM-BASED REVIEW                                       │
│  ├── Multiple LLMs review patch                                  │
│  ├── Check for common pitfalls                                   │
│  ├── Verify fix addresses root cause                             │
│  └── REJECT if consensus is negative                             │
│                                                                  │
│  Result: ACCEPT only if all layers pass                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### LLM-Based Validation (Multi-Model)

```python
def llm_validation(vulnerability, patch, num_reviewers=3):
    """
    Use multiple LLMs to review patch quality.
    """
    review_prompt = f"""
You are a security expert reviewing a vulnerability patch.

Vulnerability: {vulnerability.description}
Original code: {vulnerability.code}
Proposed patch: {patch.code}

Evaluate:
1. Does the patch fix the root cause?
2. Does it preserve existing functionality?
3. Does it introduce new vulnerabilities?
4. Are there edge cases not handled?

Rate: ACCEPT or REJECT with reasoning.
"""

    reviews = []
    for model in ["gpt-4", "claude-3", "gemini-pro"]:
        review = model.generate(review_prompt)
        reviews.append(review)

    # Require consensus
    accepts = sum(1 for r in reviews if r.decision == "ACCEPT")
    return accepts >= (num_reviewers * 0.67)  # 2/3 majority
```

---

## Reported Results

### Performance Metrics

```
Zero-Day Vulnerabilities (11 total):
┌────────────────────────────────────┐
│ Successfully patched:   7/11       │
│ Success rate:          63.6%       │
│ False positives:       1/8 (12.5%) │
│ Average validation time: 15min     │
└────────────────────────────────────┘

CVE Database (100 historical vulnerabilities):
┌────────────────────────────────────┐
│ Successfully patched:   68/100     │
│ Success rate:          68%         │
│ Improvement over baseline: +35%    │
└────────────────────────────────────┘
```

### Ablation Studies

| Configuration | Success Rate | F1 Score |
|---------------|--------------|----------|
| Raw LLM (no analysis) | 33% | 0.45 |
| + Program slicing | 48% | 0.58 |
| + Adaptive exemplars | 59% | 0.66 |
| + Multi-faceted validation | **68%** | **0.74** |

---

## Mathematical Model

### Security Patching Success

```
P(secure patch) = P(correct localization) ×
                  P(correct fix | localization) ×
                  P(validation catches errors)

APPATCH improvements:
1. P(correct localization) ↑
   - PDG slicing reduces noise
   - Focus on vulnerability semantics

2. P(correct fix | localization) ↑
   - Root cause matching improves relevance
   - Security-specific exemplars guide fix

3. P(validation catches errors) ↑
   - Multi-layer validation
   - Independent reviews reduce false positives
```

---

## Integration with K&V Workflow

### Security-Grade Calibration

APPATCH validates K&V principle: high-stakes contexts require higher rigor.

| K&V Principle | APPATCH Evidence |
|---------------|------------------|
| Evidence standards vary by domain | Security requires proof, not just tests |
| Multi-layer validation | Each layer catches different error types |
| Independent review | Multiple LLMs reduce bias |
| Analysis before generation | PDG slicing provides ground truth |

### Security Bead Template

```markdown
## Security Vulnerability Bead

**Bead ID:** bd-sec-001
**Type:** Vulnerability patch
**Severity:** HIGH

**Vulnerability:**
- **CVE:** CVE-2024-XXXXX
- **Type:** SQL Injection
- **Root Cause:** Unvalidated user input in query construction
- **Location:** auth.py:142

**Analysis:**
- **PDG Slice:** [15 lines of relevant code]
- **Data flow:** user_input → query_builder → db.execute
- **Control flow:** No validation before query execution

**Fix Strategy:**
Based on similar vulnerabilities (bd-sec-012, bd-sec-034):
1. Use parameterized queries
2. Add input validation
3. Implement allow-list for special characters

**Implementation:**
[Patch code]

**Validation:**
- [ ] Syntax valid
- [ ] Tests pass
- [ ] Behavioral equivalence verified
- [ ] Static analysis clean
- [ ] Security scan passes
- [ ] Multi-LLM review: 3/3 ACCEPT
- [ ] Manual security review

**Evidence:**
- SQL injection scanner: CLEAN
- Fuzzing results: No exploit found
- Symbolic execution: No injection path
```

### ADaPT + APPATCH Pattern

```
Security Bead: "Fix CVE-2024-XXXXX"
    │
    ├── Sub-bead 1: ANALYSIS
    │   ├── Build PDG
    │   ├── Extract vulnerability slice
    │   └── Identify root cause
    │
    ├── Sub-bead 2: PATCH GENERATION
    │   ├── Retrieve similar vulnerabilities
    │   ├── Generate patch with exemplars
    │   └── Preliminary validation
    │
    ├── Sub-bead 3: RIGOROUS VALIDATION
    │   ├── Multi-layer validation pipeline
    │   ├── Security property verification
    │   └── Independent review
    │
    └── Sub-bead 4: VERIFICATION
        ├── Penetration testing
        ├── Formal proof (if critical)
        └── Security audit sign-off
```

---

## Critical Caveats

### What APPATCH Doesn't Solve

1. **Validation Without Tests is Hard**
   - Semantic equivalence checks are partial
   - Can't prove absence of vulnerabilities
   - May miss subtle behavioral changes
   - Need domain expertise for review

2. **Program Analysis Dependency**
   - PDG construction must be correct
   - Language-specific tooling required
   - Complex codebases challenge slicing
   - Dynamic behavior may be missed

3. **LLM-Based Validation Can Collude**
   - Multiple LLMs are not truly independent
   - May share training data
   - Can agree on wrong answer
   - Still need human expert review

4. **Zero-Day Evaluation is Limited**
   - 11 vulnerabilities is small sample
   - Selection bias in evaluation set
   - Real-world deployment unclear
   - Long-term security impact unknown

---

## Practical Implications

### When to Use APPATCH-Style Approach

```
Use security-grade patching when:
✓ Vulnerability has security impact
✓ Program analysis tools available
✓ Can afford rigorous validation
✓ Have security expertise for review
✓ False fixes are dangerous

Standard repair may suffice when:
○ Functional bug only
○ Good test coverage
○ Low security impact
○ Fast iteration needed
```

### Designing Security Validation

```python
SECURITY_VALIDATION_CHECKLIST = """
1. Static Analysis:
   - [ ] No new vulnerability patterns introduced
   - [ ] Common vulnerability database check
   - [ ] Code complexity within bounds

2. Dynamic Analysis:
   - [ ] Fuzzing passes (min 10k inputs)
   - [ ] Symbolic execution clean
   - [ ] Exploit attempts fail

3. Behavioral Preservation:
   - [ ] All existing tests pass
   - [ ] Property-based tests pass
   - [ ] Performance within 10% of original

4. Expert Review:
   - [ ] Security team sign-off
   - [ ] Multi-LLM consensus
   - [ ] Formal proof (if critical path)

5. Deployment Safeguards:
   - [ ] Canary deployment plan
   - [ ] Rollback procedure tested
   - [ ] Monitoring alerts configured
"""
```

---

## Research Methodology

### Experimental Design

- **Zero-day evaluation:** 11 vulnerabilities from bug bounty programs
- **Historical evaluation:** 100 CVEs from NVD database
- **Baselines:** Standard prompting, CodeBERT, repair baselines
- **Validation:** Manual security expert review of all patches
- **Metrics:** Precision, recall, F1 for correct+secure patches

### Exemplar Pool Construction

- **Size:** 500 vulnerability-fix pairs
- **Sources:** CVE database, security commits, bug bounty reports
- **Validation:** Each pair manually reviewed by security experts
- **Taxonomy:** Categorized by CWE (Common Weakness Enumeration)

---

## Key Metrics to Track

When implementing security patching:

```json
{
  "vulnerability_id": "CVE-2024-XXXXX",
  "severity": "HIGH",
  "approach": "appatch",
  "analysis": {
    "pdg_nodes": 45,
    "slice_size": 15,
    "root_cause": "sql_injection"
  },
  "exemplars_used": [
    {"id": "ex-012", "similarity": 0.89},
    {"id": "ex-034", "similarity": 0.82},
    {"id": "ex-091", "similarity": 0.78}
  ],
  "validation": {
    "syntax": "pass",
    "tests": "pass",
    "semantic_equiv": "pass",
    "static_analysis": "pass",
    "llm_reviews": {"accept": 3, "reject": 0},
    "manual_review": "approved"
  },
  "deployment": {
    "canary_passed": true,
    "monitoring": "active",
    "rollback_tested": true
  }
}
```

---

## Key Takeaways

1. **Security requires higher rigor** — Multi-layer validation, not just tests
2. **Analysis before generation** — PDG slicing provides ground truth
3. **Root cause matching improves exemplars** — Similarity within vulnerability class
4. **Multi-faceted validation catches errors** — Each layer addresses different failure mode
5. **Independent review reduces false confidence** — Don't rely on single validator

---

## See Also

- `021-swe-bench-plus.md` — Weak validation misleads even security fixes
- `023-toggle.md` — Token-level localization and constrained editing
- `024-thinkrepair.md` — Knowledge pool construction and retrieval
- `022-chatrepair.md` — Iterative refinement with feedback
- `060-debugging-decay-index.md` — Why security can't afford many iterations
