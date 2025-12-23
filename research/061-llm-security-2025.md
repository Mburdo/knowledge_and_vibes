---
title: LLM Security Vulnerabilities 2025
tags: [security, vulnerabilities, scanning, owasp, language-specific, ubs]
protocols: [P13]
---

# LLM Code Security Vulnerabilities 2025

**Sources:**
- Veracode 2025 GenAI Code Security Report (100+ LLMs tested)
- OWASP Top 10 for LLM Applications 2025
- "Popular LLMs Found to Produce Vulnerable Code by Default" (April 2025)
- Emergent Mind: "LLM-Generated Code Security" (December 2025)

---

## Summary

Comprehensive 2025 analysis of security vulnerabilities in LLM-generated code. Critical finding: **45% of AI-generated code fails security tests**.

**This is why `ubs --staged` is mandatory before every commit.**

---

## Key Statistics (2025)

### Overall Vulnerability Rates

| Metric | Value | Source |
|--------|-------|--------|
| Code failing OWASP Top 10 | **45%** | Veracode 2025 |
| Snippets with exploitable flaws | **12-65%** | Emergent Mind |
| Choosing insecure over secure | **45%** | Veracode 2025 |

### By Programming Language

| Language | Security Failure Rate |
|----------|----------------------|
| **Java** | **72%** (highest risk) |
| C# | 45% |
| JavaScript | 43% |
| Python | 38% |

### Specific Vulnerability Failures

| Vulnerability | Failure Rate |
|---------------|--------------|
| Cross-Site Scripting (CWE-80) | **86%** failed to secure |
| Log Injection (CWE-117) | **74%** failed to secure |
| SQL Injection (CWE-89) | High |
| Command Injection (CWE-78) | High |

---

## Most Common CWEs in LLM Code

From CWE Top 25 and MITRE analysis:

| CWE | Vulnerability | Prevalence |
|-----|---------------|------------|
| CWE-89 | SQL Injection | Very High |
| CWE-78 | OS Command Injection | Very High |
| CWE-94 | Code Injection | High |
| CWE-79/80 | Cross-Site Scripting | Very High |
| CWE-22 | Path Traversal | High |
| CWE-120/787 | Buffer Overflow | Medium |
| CWE-252/253 | Unchecked Return Value | High |
| CWE-259/798 | Hard-coded Credentials | Medium |
| CWE-780 | Cryptographic Misuse | Medium |
| CWE-117 | Log Injection | High |

---

## Why LLMs Generate Vulnerable Code

### Root Causes

1. **Training data contamination** — Trained on public code including insecure patterns
2. **Secure vs. simple tradeoff** — Simpler (insecure) patterns are more common in training
3. **Missing validation by default** — Input validation not automatic
4. **Naive prompt responses** — Simple prompts yield simple (insecure) code
5. **Context limitations** — Security considerations not in immediate context

### The "Naive Prompt" Problem

From Backslash Security research:

```
Naive Prompt: "Write a function to query the database for users"

LLM Response:
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"  # SQL INJECTION!
    return db.execute(query)

Secure Prompt: "Write a secure function to query the database for users,
using parameterized queries to prevent SQL injection"

LLM Response:
def get_user(user_id):
    query = "SELECT * FROM users WHERE id = ?"
    return db.execute(query, (user_id,))  # Parameterized - safe
```

---

## OWASP Top 10 for LLM Applications 2025

| Rank | Risk | Code Generation Impact |
|------|------|------------------------|
| 1 | **Prompt Injection** | Can cause LLM to generate malicious code |
| 2 | Sensitive Information Disclosure | Credentials in generated code |
| 3 | Supply Chain | Suggest vulnerable dependencies |
| 4 | Data and Model Poisoning | Training on malicious patterns |
| 5 | Improper Output Handling | Generated code trusts input |
| 6 | Excessive Agency | Generated code has too many permissions |
| 7 | System Prompt Leakage | Exposes security configurations |
| 8 | Vector/Embedding Attacks | Retrieves insecure code patterns |
| 9 | Misinformation | Suggests deprecated/insecure APIs |
| 10 | Unbounded Consumption | DoS through generated code |

---

## Mitigation Strategies

### Mandatory Security Scanning

```bash
# BEFORE EVERY COMMIT
ubs --staged

# If HIGH/CRITICAL found:
# 1. Fix immediately
# 2. Do not commit until clean

# If MEDIUM found:
# 1. Document in bead close reason
# 2. Create follow-up bead if needed
```

### Secure Prompt Patterns

| Pattern | Example |
|---------|---------|
| Explicit security requirement | "Use parameterized queries to prevent SQL injection" |
| Reference OWASP | "Following OWASP guidelines, implement..." |
| Specify validation | "Validate and sanitize all user input" |
| Request defense in depth | "Implement input validation, output encoding, and parameterized queries" |

### Code Review Checklist

Before accepting LLM-generated code:

- [ ] Input validation present
- [ ] Parameterized queries (no string concatenation)
- [ ] No hard-coded credentials
- [ ] Proper error handling (no stack traces to user)
- [ ] Output encoding for display
- [ ] Least privilege principles
- [ ] Secure defaults
- [ ] `ubs --staged` clean

---

## Language-Specific Risks

### Java (72% failure rate)

```java
// VULNERABLE (common LLM output)
String query = "SELECT * FROM users WHERE name = '" + name + "'";

// SECURE
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE name = ?");
stmt.setString(1, name);
```

### Python (38% failure rate)

```python
# VULNERABLE
os.system(f"ls {user_input}")  # Command injection

# SECURE
subprocess.run(["ls", user_input], shell=False)
```

### JavaScript (43% failure rate)

```javascript
// VULNERABLE
element.innerHTML = userInput;  // XSS

// SECURE
element.textContent = userInput;
```

---

## Integration with Knowledge & Vibes

### UBS Scanner Configuration

The `ubs --staged` tool catches these issues:

```yaml
# .ubs/config.yaml
severity_threshold: medium
fail_on: [high, critical]
rules:
  - sql-injection
  - command-injection
  - xss
  - path-traversal
  - hardcoded-secrets
```

### Bead Packaging Requirements

Every bead must:

1. Run `ubs --staged` before commit
2. Fix all HIGH/CRITICAL findings
3. Document accepted MEDIUM findings
4. Include security tests for risky operations

### Security Gate in Execution

```
┌─────────────────────────────────────────────────────────────┐
│                    BEAD EXECUTION FLOW                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Write tests (TDD)                                        │
│  2. Implement code                                           │
│  3. Tests pass                                               │
│  4. ubs --staged  ←──── MANDATORY SECURITY GATE             │
│      │                                                       │
│      ├── Clean? → Continue to commit                         │
│      └── Issues? → Fix before proceeding                     │
│                                                              │
│  5. Commit only after security clean                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Never trust LLM code implicitly** — 45% has security issues
2. **Always run security scans** — `ubs --staged` is mandatory
3. **Java is highest risk** — Extra scrutiny for Java code
4. **Prompts matter** — Explicit security requirements help
5. **Training data is contaminated** — LLMs learned insecure patterns
6. **Input validation missing by default** — Always add explicitly

---

## See Also

- `052-llm-security-vulnerabilities.md` — Earlier security research
- `053-feedback-loop-security.md` — Security in feedback loops
- `.claude/skills/ubs-scanner/SKILL.md` — UBS tool usage
