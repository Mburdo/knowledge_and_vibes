# IRIS: LLM-Assisted Static Analysis for Security

**Paper:** IRIS: LLM-Assisted Static Analysis for Detecting Security Vulnerabilities
**URL:** https://arxiv.org/abs/2405.17238
**Date:** May 2024 (updated April 2025)
**Venue:** arXiv preprint

---

## Summary

IRIS demonstrates that **combining LLMs with traditional static analysis** achieves better security vulnerability detection than either approach alone. The system uses LLMs to augment analysis specifications, interpret results, and reason about whole-repository security properties that traditional tools miss.

**Key innovation:** Hybrid architecture where LLMs provide context-aware reasoning while static analysis provides sound program analysis, creating a "best of both worlds" approach.

---

## The Security Detection Problem

### Why Traditional Tools Fail

```
┌─────────────────────────────────────────────────────────────────┐
│         TRADITIONAL STATIC ANALYSIS LIMITATIONS                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problem 1: Incomplete Specifications                            │
│      ├── Requires manual taint specs for each API                │
│      ├── Misses new/custom APIs                                  │
│      └── Can't infer implicit security properties                │
│                                                                  │
│  Problem 2: High False Positive Rate                             │
│      ├── Conservatively flags many safe patterns                 │
│      ├── Developers ignore warnings (alert fatigue)              │
│      └── Hard to prioritize real issues                          │
│                                                                  │
│  Problem 3: Context Insensitivity                                │
│      ├── Can't understand business logic                         │
│      ├── Misses semantic vulnerabilities                         │
│      └── Doesn't reason about multi-file flows                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Why Pure LLMs Fail

```
┌─────────────────────────────────────────────────────────────────┐
│              PURE LLM SECURITY ANALYSIS LIMITATIONS              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problem 1: Unsound Reasoning                                    │
│      ├── May miss subtle data flows                              │
│      ├── Can't prove absence of vulnerabilities                  │
│      └── Unreliable on complex control flow                      │
│                                                                  │
│  Problem 2: Scale Limitations                                    │
│      ├── Can't analyze entire large codebases                    │
│      ├── Context window limits                                   │
│      └── Computational cost prohibitive                          │
│                                                                  │
│  Problem 3: Hallucinated Vulnerabilities                         │
│      ├── May report non-existent issues                          │
│      ├── Inconsistent between runs                               │
│      └── Over-confident false positives                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## IRIS Architecture

### Three-Stage Hybrid Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    IRIS ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Stage 1: LLM-Assisted Specification                             │
│      ├── Input: Codebase + API documentation                     │
│      ├── LLM: Infer taint specifications                         │
│      │   ├── Identify sources (user input, files, network)       │
│      │   ├── Identify sinks (SQL, eval, file ops)                │
│      │   └── Identify sanitizers (validation functions)          │
│      └── Output: Enhanced specification for static analyzer      │
│                                                                  │
│  Stage 2: Sound Static Analysis                                  │
│      ├── Input: Code + LLM-enhanced specs                        │
│      ├── Tool: CodeQL / Semmle / Datalog engine                  │
│      │   ├── Compute precise data flows                          │
│      │   ├── Track taint propagation                             │
│      │   └── Identify potential vulnerabilities                  │
│      └── Output: Candidate vulnerabilities with traces           │
│                                                                  │
│  Stage 3: LLM-Assisted Triage                                    │
│      ├── Input: Candidates + code context                        │
│      ├── LLM: Validate and prioritize findings                   │
│      │   ├── Eliminate false positives (business logic)          │
│      │   ├── Assess exploitability                               │
│      │   ├── Suggest fixes                                       │
│      │   └── Rank by severity                                    │
│      └── Output: Validated vulnerabilities + remediation         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Data Flow

```
Codebase
    │
    ├──→ LLM (Stage 1): Extract API specs
    │        └──→ Taint specifications
    │
    ├──→ Static Analyzer (Stage 2): Find data flows
    │        └──→ Vulnerability candidates + traces
    │
    └──→ LLM (Stage 3): Validate + prioritize
             └──→ Confirmed vulnerabilities + fixes
```

---

## Mathematical Model

### Precision and Recall Trade-off

```
Detection Metrics:

Precision = True Positives / (True Positives + False Positives)
Recall    = True Positives / (True Positives + False Negatives)

Traditional Static Analysis:
  High Recall (finds most issues)
  Low Precision (many false alarms)
  → P ≈ 0.3, R ≈ 0.9

Pure LLM:
  Medium Precision (some hallucinations)
  Medium Recall (misses subtle flows)
  → P ≈ 0.6, R ≈ 0.5

IRIS Hybrid:
  High Precision (LLM filters false positives)
  High Recall (static analysis soundness)
  → P ≈ 0.75, R ≈ 0.85
```

### F1 Score Improvement

```
F1 = 2 × (Precision × Recall) / (Precision + Recall)

Baseline CodeQL: F1 = 0.45
Pure LLM:        F1 = 0.54
IRIS:            F1 = 0.80  (78% improvement over baseline)
```

---

## Benchmark Performance

### CWE-Bench-Java Results (Reported)

| System | Vulnerabilities Found | False Positives | False Discovery Rate |
|--------|----------------------|-----------------|---------------------|
| CodeQL (baseline) | 27 | 18 | 40% |
| Pure GPT-4 | 38 | 22 | 37% |
| **IRIS** | **55** | **12** | **18%** |

### Real-World Impact

```
Previously Unknown Vulnerabilities Discovered:

Project A (Web Framework):
  ├── SQL Injection in auth middleware
  ├── XSS in template engine
  └── Path traversal in file upload

Project B (API Server):
  ├── Command injection in backup handler
  └── Sensitive data leak in logging

Project C (Mobile Backend):
  ├── Authentication bypass
  └── Race condition in session management

Total: 4 new CVEs reported (paper claims)
```

---

## Common Vulnerability Patterns

### Taint Analysis Examples

| CWE | Vulnerability Type | Source | Sink | IRIS Detection |
|-----|-------------------|--------|------|----------------|
| CWE-89 | SQL Injection | HTTP param | executeQuery() | Yes (with context) |
| CWE-79 | XSS | User input | innerHTML | Yes (tracks DOM) |
| CWE-78 | Command Injection | File upload | Runtime.exec() | Yes (with sanitizer check) |
| CWE-22 | Path Traversal | URL param | File.open() | Yes (validates normalization) |
| CWE-502 | Insecure Deserialization | Network | ObjectInputStream | Yes (type analysis) |

### Code Example: SQL Injection Detection

```java
// Vulnerable Code
public User login(String username, String password) {
    // SOURCE: user-controlled input
    String query = "SELECT * FROM users WHERE username='"
                   + username + "' AND password='" + password + "'";

    // SINK: SQL execution
    return db.executeQuery(query);
}

// IRIS Analysis:

// Stage 1 (LLM): Infer specifications
// - username: tainted (HTTP parameter)
// - password: tainted (HTTP parameter)
// - executeQuery(): sink (SQL execution)
// - No sanitizer detected

// Stage 2 (Static Analysis): Trace data flow
// username → concatenation → query → executeQuery()
//   ^SOURCE                            ^SINK
// Flow: TAINTED → UNSANITIZED → SINK

// Stage 3 (LLM): Validate and suggest fix
// Vulnerability: SQL Injection (CWE-89)
// Severity: CRITICAL
// Exploitability: Easy (standard SQLi)
// Suggested Fix: Use PreparedStatement

// Fixed Code (LLM-suggested)
public User login(String username, String password) {
    String query = "SELECT * FROM users WHERE username=? AND password=?";
    PreparedStatement stmt = db.prepareStatement(query);
    stmt.setString(1, username);  // Sanitizer: parameterized query
    stmt.setString(2, password);
    return stmt.executeQuery();
}
```

---

## Integration with Knowledge & Vibes

### Security Verification Bead

```
┌─────────────────────────────────────────────────────────────────┐
│              IRIS-STYLE SECURITY VERIFICATION                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Automated Static Analysis                               │
│      ├── Run CodeQL / Semgrep / Bandit (language-specific)       │
│      ├── Collect vulnerability candidates                        │
│      └── Generate data flow traces                               │
│                                                                  │
│  Step 2: LLM-Enhanced Specification (IRIS Stage 1)               │
│      ├── Analyze code for custom APIs                            │
│      ├── Infer additional taint sources/sinks                    │
│      ├── Identify project-specific sanitizers                    │
│      └── Re-run static analysis with enhanced specs              │
│                                                                  │
│  Step 3: LLM-Assisted Triage (IRIS Stage 3)                      │
│      ├── For each candidate vulnerability:                       │
│      │   ├── Validate with business logic context                │
│      │   ├── Check if sanitizer exists in flow                   │
│      │   ├── Assess real-world exploitability                    │
│      │   └── Filter false positives                              │
│      └── Generate ranked findings report                         │
│                                                                  │
│  Step 4: Generate Fixes                                          │
│      ├── LLM proposes remediation for each issue                 │
│      ├── Create fix bead for each vulnerability                  │
│      └── Verify fixes don't introduce new issues                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### North Star Integration

| North Star Tier | Security Requirements | IRIS Application |
|-----------------|----------------------|------------------|
| **Prototype** | Basic awareness | None (move fast) |
| **Team Internal** | No critical vulnerabilities | Quick scan, major issues only |
| **Production** | Security scan + review | Full IRIS pipeline, all findings |
| **Enterprise** | Comprehensive security audit | IRIS + manual audit + pentesting |
| **Regulatory** | Certified secure | IRIS + compliance tools + certification |

---

## Practical Implementation

### Code Example: IRIS-Style Workflow

```python
from dataclasses import dataclass
from typing import List, Optional
import subprocess
import json

@dataclass
class TaintSpec:
    """Taint specification inferred by LLM."""
    sources: List[str]      # Functions returning tainted data
    sinks: List[str]        # Functions requiring sanitized data
    sanitizers: List[str]   # Functions that sanitize data

@dataclass
class Vulnerability:
    """Detected vulnerability."""
    cwe_id: str
    severity: str           # CRITICAL, HIGH, MEDIUM, LOW
    location: str           # file:line
    description: str
    data_flow: List[str]    # Source → ... → Sink
    exploitable: bool
    suggested_fix: str

class IRISStyleSecurityAnalysis:
    """IRIS-inspired security analysis for K&V."""

    def analyze(self, codebase_path: str) -> List[Vulnerability]:
        """
        Run IRIS-style three-stage analysis.

        Returns: List of validated vulnerabilities
        """
        # Stage 1: LLM-assisted specification
        taint_specs = self.infer_taint_specs(codebase_path)

        # Stage 2: Static analysis with enhanced specs
        candidates = self.run_static_analysis(codebase_path, taint_specs)

        # Stage 3: LLM-assisted triage
        validated = self.validate_and_prioritize(candidates)

        return validated

    def infer_taint_specs(self, codebase_path: str) -> TaintSpec:
        """
        Stage 1: Use LLM to infer taint specifications.
        """
        # Read codebase structure
        api_functions = self.extract_api_functions(codebase_path)

        # Ask LLM to categorize functions
        prompt = f"""
Analyze these API functions and categorize them:

{json.dumps(api_functions, indent=2)}

Identify:
1. SOURCES: Functions that return user-controlled/untrusted data
   (e.g., HTTP params, file reads, network input)

2. SINKS: Functions that perform security-sensitive operations
   (e.g., SQL execution, command execution, file operations)

3. SANITIZERS: Functions that validate/escape/sanitize data
   (e.g., input validation, escaping, encoding)

Return JSON: {{"sources": [...], "sinks": [...], "sanitizers": [...]}}
"""

        llm_response = call_llm(prompt)
        return TaintSpec(**json.loads(llm_response))

    def run_static_analysis(self,
                          codebase_path: str,
                          specs: TaintSpec) -> List[dict]:
        """
        Stage 2: Run static analyzer with LLM-enhanced specs.
        """
        # Generate CodeQL/Semgrep config from specs
        config = self.generate_analysis_config(specs)

        # Run static analysis
        result = subprocess.run(
            ["codeql", "database", "analyze",
             codebase_path,
             "--format=json",
             "--config", config],
            capture_output=True,
            text=True
        )

        candidates = json.loads(result.stdout)
        return candidates

    def validate_and_prioritize(self,
                               candidates: List[dict]) -> List[Vulnerability]:
        """
        Stage 3: Use LLM to validate findings and filter false positives.
        """
        validated = []

        for candidate in candidates:
            # Get code context for this finding
            context = self.extract_context(
                candidate['file'],
                candidate['line'],
                context_lines=20
            )

            # Ask LLM to validate
            validation_prompt = f"""
A static analyzer flagged a potential vulnerability:

Type: {candidate['type']}
Location: {candidate['file']}:{candidate['line']}
Data Flow: {' → '.join(candidate['flow'])}

Code Context:
{context}

Questions:
1. Is this a real vulnerability or false positive?
2. Can it be exploited in practice?
3. What is the severity (CRITICAL/HIGH/MEDIUM/LOW)?
4. What is the fix?

Return JSON: {{"real": bool, "exploitable": bool,
              "severity": str, "fix": str, "reasoning": str}}
"""

            llm_validation = json.loads(call_llm(validation_prompt))

            if llm_validation['real']:
                validated.append(Vulnerability(
                    cwe_id=candidate['cwe'],
                    severity=llm_validation['severity'],
                    location=f"{candidate['file']}:{candidate['line']}",
                    description=candidate['description'],
                    data_flow=candidate['flow'],
                    exploitable=llm_validation['exploitable'],
                    suggested_fix=llm_validation['fix']
                ))

        # Sort by severity
        severity_order = {'CRITICAL': 0, 'HIGH': 1, 'MEDIUM': 2, 'LOW': 3}
        validated.sort(key=lambda v: severity_order[v.severity])

        return validated
```

### Integration with Bead Workflow

```markdown
## Security Verification Bead (bd-security-001)

### Input
- Codebase at current state
- Changed files from this phase

### Steps

1. **Run Static Analysis**
   ```bash
   # Run language-specific tools
   semgrep --config=auto --json > findings.json
   ```

2. **LLM Enhancement** (IRIS Stage 1 + 3)
   ```bash
   # Infer specs and validate findings
   python iris_analysis.py --codebase . --findings findings.json
   ```

3. **Review Findings**
   - Filter to CRITICAL and HIGH only
   - Create fix beads for each real vulnerability
   - Document false positives (update specs)

4. **Acceptance Criteria**
   - Zero CRITICAL vulnerabilities
   - Zero HIGH vulnerabilities in production code
   - All MEDIUM reviewed and documented
   - Security scan passes in CI

### Output
- Security report (JSON)
- Fix beads created (if needed)
- Updated taint specifications
```

---

## Caveats and Limitations

### IRIS is Research, Not Product

| IRIS (Research) | K&V (Practical) | Adaptation |
|----------------|----------------|------------|
| Custom CodeQL integration | Use existing tools | Semgrep, Bandit, ESLint plugins |
| GPT-4 API (their setup) | Any LLM | Use available model |
| Java focus | Polyglot repos | Language-specific tools |
| Whole-repo analysis | Incremental analysis | Focus on changed files |

### When Hybrid Analysis Helps Most

- **Complex data flows:** Multi-file taint propagation
- **Custom frameworks:** Non-standard APIs and patterns
- **False positive filtering:** Business logic context needed
- **Prioritization:** Many findings need triage

### When It's Overkill

- **Prototype phase:** Security later, speed now
- **Internal tools:** Lower security requirements
- **Static analysis sufficient:** Clear-cut vulnerabilities

---

## Key Takeaways

1. **Hybrid > Pure:** Combining LLMs + static analysis beats either alone
2. **Three stages matter:** Spec inference → Sound analysis → LLM triage
3. **Context is key:** LLMs add business logic understanding to static analysis
4. **False positive reduction:** Major benefit is filtering, not just finding
5. **Scale appropriately:** Security rigor should match North Star tier
6. **Fixes are actionable:** LLM-generated remediation accelerates response
7. **Continuous integration:** Security checks belong in CI, not one-time audits

---

## See Also

- `050-swe-bench-pro.md` — Realistic expectations for AI capabilities
- `042-rankef.md` — Using multiple signals for better decisions
- `041-debatecoder.md` — Test-based validation approaches
- `045-repairagent.md` — Automated bug fixing workflows
- `065-confucius-code-agent.md` — Large-scale codebase analysis
