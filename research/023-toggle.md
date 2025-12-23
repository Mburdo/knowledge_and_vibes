---
title: Toggle - Token-Level Bug Localization and Repair
tags: [localization, token-level, repair, constrained-editing, surgical-edits, encoder-decoder]
protocols: [P2, P8]
---

# Toggle: Token-Level Bug Localization and Repair

**Paper:** A Deep Dive into Large Language Models for Automated Bug Localization and Repair (Toggle framework)
**URL:** https://arxiv.org/html/2404.11595v1
**Date:** April 2024
**Venue:** arXiv preprint

---

## Summary

Revolutionary approach that pushes bug localization from **line-level** to **token-level** granularity. Separates localization (encoder) from fixing (decoder) to constrain edits and improve precision.

**Key innovation:** "Edit this specific token span, keep everything else" dramatically reduces hallucination and unnecessary code regeneration.

---

## The Granularity Problem

### Line-Level vs Token-Level Localization

```
┌─────────────────────────────────────────────────────────────────┐
│              LINE-LEVEL LOCALIZATION (Traditional)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Buggy code:                                                     │
│  41: def calculate_discount(price, rate):                        │
│  42:     discount = price * rate / 100  ← Bug in `/100`          │
│  43:     return price - discount                                 │
│                                                                  │
│  Line-level localization: "Line 42 is buggy"                     │
│                                                                  │
│  Problem: Model regenerates entire line                          │
│  → May introduce new bugs in correct parts                       │
│  → Wastes context on redundant code                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                TOKEN-LEVEL LOCALIZATION (Toggle)                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Buggy code:                                                     │
│  41: def calculate_discount(price, rate):                        │
│  42:     discount = price * rate / 100  ← Bug: tokens `/ 100`    │
│  43:     return price - discount                                 │
│                                                                  │
│  Token-level localization: "Tokens 7-8 on line 42"               │
│  Highlighted: [price] [*] [rate] [/] [100]                       │
│                                    ^^^ ^^^                       │
│                                                                  │
│  Fix model: "Edit ONLY these tokens, preserve rest"              │
│  Result: Surgical edit, no collateral changes                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Precision Comparison

| Granularity | What Gets Regenerated | Risk of New Bugs |
|-------------|----------------------|------------------|
| **Function-level** | Entire function | Very High |
| **Line-level** | Entire line | High |
| **Token-level** | Specific tokens only | Low |

---

## Toggle Architecture

### Two-Model Design

```
┌─────────────────────────────────────────────────────────────────┐
│                     TOGGLE ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Input: Buggy Code + Bug Report                                 │
│         │                                                        │
│         ↓                                                        │
│  ┌──────────────────────────────────┐                           │
│  │   LOCALIZATION MODEL             │                           │
│  │   (Encoder-style, e.g., CodeT5)  │                           │
│  │                                  │                           │
│  │   Task: Predict buggy token span │                           │
│  │   Output: start_token, end_token │                           │
│  └──────────────┬───────────────────┘                           │
│                 │                                                │
│                 ↓                                                │
│         Token span: [7, 8]                                       │
│                 │                                                │
│                 ↓                                                │
│  ┌──────────────────────────────────┐                           │
│  │    FIXING MODEL                  │                           │
│  │    (Decoder/generative)          │                           │
│  │                                  │                           │
│  │    Prompt: "Edit tokens 7-8,     │                           │
│  │             keep everything else" │                           │
│  │    Output: Fixed code            │                           │
│  └──────────────┬───────────────────┘                           │
│                 │                                                │
│                 ↓                                                │
│         Fixed code with surgical edit                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Why Separation Matters

| Aspect | Combined Model | Separated (Toggle) |
|--------|----------------|-------------------|
| Localization | Implicit | Explicit, trainable |
| Edit scope | Entire context | Constrained span |
| Failure mode | Hallucination | Localization error |
| Debugging | Opaque | Clear: loc vs fix |

---

## Key Components

### 1. Token-Level Localization Model

```python
# Conceptual implementation

class TokenLocalizer(Encoder):
    def __init__(self, model="codet5-base"):
        self.encoder = load_model(model)

    def localize(self, code, bug_report):
        """
        Predict start and end token indices for buggy span.
        """
        # Encode code + bug report
        representation = self.encoder.encode(code, bug_report)

        # Predict token span
        start_logits = self.predict_start(representation)
        end_logits = self.predict_end(representation)

        start_token = argmax(start_logits)
        end_token = argmax(end_logits)

        return (start_token, end_token)
```

### 2. Constrained Fix Generation

```python
def generate_fix(code_tokens, buggy_span, fix_model):
    """
    Generate fix for specific token span.
    """
    start, end = buggy_span

    # Construct prompt with constraints
    prompt = f"""
Code:
{' '.join(code_tokens)}

Bug location: tokens {start} to {end}
Buggy tokens: {' '.join(code_tokens[start:end+1])}

Generate a fix that:
1. Replaces ONLY tokens {start}-{end}
2. Keeps all other code unchanged
3. Maintains syntax validity

Fixed tokens:
"""

    fixed_span = fix_model.generate(prompt)

    # Reconstruct code
    fixed_code = (
        code_tokens[:start] +
        fixed_span +
        code_tokens[end+1:]
    )

    return fixed_code
```

### 3. Tokenization Alignment Module

A critical engineering component:

```python
class TokenizerAdapter:
    """
    Handles mismatch between localization and fixing model tokenizers.
    """
    def __init__(self, loc_tokenizer, fix_tokenizer):
        self.loc_tok = loc_tokenizer
        self.fix_tok = fix_tokenizer

    def align_span(self, code, loc_span):
        """
        Convert token span from localization tokenizer
        to fixing tokenizer coordinates.
        """
        start_loc, end_loc = loc_span

        # Get character offsets from loc tokenizer
        char_start = self.loc_tok.token_to_char(start_loc)
        char_end = self.loc_tok.token_to_char(end_loc)

        # Convert to fix tokenizer spans
        start_fix = self.fix_tok.char_to_token(char_start)
        end_fix = self.fix_tok.char_to_token(char_end)

        return (start_fix, end_fix)
```

---

## Prompt Engineering for Constrained Editing

### Effective Prompt Structure

```
GOOD PROMPT (Constrained):

Code:
1: def authenticate(user, password):
2:     hash = md5(password)
3:     return hash == user.password_hash

Bug: Line 2, tokens 3-4 (`md5(password)`)
Issue: MD5 is insecure for passwords

Edit ONLY tokens 3-4. Preserve all other code exactly.
Keep the same variable name and structure.

Fixed tokens 3-4:

---

BAD PROMPT (Unconstrained):

Fix the authentication bug in this code:
[Shows full code]

Problem: Model regenerates entire function, may introduce:
- Different variable names
- Reordered lines
- Style changes
- New bugs
```

---

## Reported Results

### Performance Metrics

```
CodeXGLUE Code Refinement Tasks:

Toggle Results:
┌─────────────────────────────────────┐
│ New SOTA on refinement benchmarks  │
│ Competitive on Defects4J            │
│ 15-20% improvement over baselines   │
└─────────────────────────────────────┘

Key insight:
- Localization accuracy is the bottleneck
- When location is correct, fix success rate is high
- When location is wrong, fix is confidently wrong
```

### Ablation Study Results

| Configuration | Success Rate | Notes |
|---------------|--------------|-------|
| Line-level loc + unconstrained fix | 35% | Baseline |
| Token-level loc + unconstrained fix | 42% | Better localization |
| Token-level loc + constrained fix | **58%** | Best: precise + constrained |

---

## Mathematical Model

### Localization-Fix Decomposition

```
P(correct fix) = P(correct localization) × P(correct fix | correct loc)

Traditional (implicit localization):
  Both probabilities are coupled
  Hard to improve independently

Toggle (explicit localization):
  Can train/tune localization separately
  Can improve fix model independently
  Can measure each component's contribution

If P(loc) = 0.7 and P(fix | loc) = 0.8:
  P(success) = 0.7 × 0.8 = 0.56

To improve to 0.7:
  Option A: Improve P(loc) to 0.875 (challenging)
  Option B: Improve P(fix|loc) to 1.0 (constrained editing helps)
```

---

## Critical Insight: Separation of Concerns

### Why "Where" and "What" Are Different Problems

```
┌─────────────────────────────────────────────────────────────────┐
│               LOCALIZATION VS FIXING ARE DIFFERENT               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LOCALIZATION ("Where is the bug?")                              │
│  ├── Input: Code + bug report + maybe test failure              │
│  ├── Task: Classification/detection                             │
│  ├── Best model: Encoder (understand context)                   │
│  ├── Training data: Bug location labels                         │
│  └── Metric: Precision@k for buggy spans                        │
│                                                                  │
│  FIXING ("What should it be?")                                   │
│  ├── Input: Code + buggy span + constraints                     │
│  ├── Task: Generation/transformation                            │
│  ├── Best model: Decoder (generate code)                        │
│  ├── Training data: Bug → fix pairs                             │
│  └── Metric: Exact match or test pass rate                      │
│                                                                  │
│  Conflating them leads to:                                       │
│  • Diffuse gradients                                             │
│  • Unclear failure modes                                         │
│  • Harder to improve                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Integration with K&V Workflow

### Planning Phase Implications

Toggle's architecture maps to K&V planning phases:

| Toggle Component | K&V Phase | Artifact |
|------------------|-----------|----------|
| Localization | Analysis/Investigation | "Bug is in auth.py:42, tokens 7-8" |
| Fix generation | Implementation | Constrained patch |
| Validation | Calibration | Tests must pass |

### Encoding Localization as Distinct Artifact

```markdown
## Localization Artifact Template

**File:** auth.py
**Line:** 42
**Token span:** 7-8
**Buggy code:** `md5(password)`
**Confidence:** 0.87
**Evidence:**
  - Bug report mentions "insecure hash"
  - Test failure in password validation
  - Known vulnerability pattern

**Constraints for fix:**
  - Replace only tokens 7-8
  - Preserve variable name `hash`
  - Maintain return statement structure
  - Use secure hashing (SHA256+ with salt)
```

### ADaPT + Toggle Pattern

```
Task Bead: "Fix authentication vulnerability"
    │
    ├── Sub-bead 1: LOCALIZATION
    │   ├── Input: Code + CVE report
    │   ├── Output: "Line 42, tokens 7-8: md5(password)"
    │   └── Validation: Manual review of location
    │
    ├── Sub-bead 2: FIXING
    │   ├── Input: Located span + constraints
    │   ├── Output: "bcrypt(password, salt)"
    │   └── Constraint: Edit ONLY the located span
    │
    └── Sub-bead 3: VALIDATION
        ├── Tests pass
        ├── Security scan passes
        └── Behavioral equivalence verified

Benefits:
- Clear failure attribution (loc vs fix)
- Can retry localization separately from fixing
- Evidence chain is explicit
```

---

## Critical Caveats

### What Toggle Doesn't Solve

1. **Localization Quality Dominates**
   - Wrong span → confident wrong fix
   - No fix model can overcome bad localization
   - Garbage in, garbage out

2. **Tokenizer Mismatch Complexity**
   - Alignment module is non-trivial engineering
   - Different tokenizers split differently
   - Character-level alignment has edge cases

3. **APR ≠ Repo Issue Resolution**
   - Toggle benchmarks are often single-function
   - Real issues may span multiple files
   - Environment/dependency bugs not addressed

4. **Multi-Location Bugs**
   - Assumes single buggy span
   - Real bugs may require coordinated edits
   - Token-level doesn't help with architectural issues

---

## Practical Implications

### When to Use Token-Level Localization

```
Use Toggle-style approach when:
✓ Bug is localized to specific code region
✓ You have good bug reports/test failures
✓ Fix should be surgical (minimal change)
✓ Localization model is available/trainable
✓ Precision matters more than recall

Don't use when:
✗ Bug requires multi-file coordination
✗ Architectural changes needed
✗ Bug location is completely unknown
✗ Generating new code (not fixing existing)
```

### Designing Constrained Edit Prompts

```python
# Template for constrained editing

CONSTRAINED_EDIT_TEMPLATE = """
Code context:
{code_before}
{buggy_span}  ← EDIT THIS
{code_after}

Bug location: Line {line}, tokens {start}-{end}
Buggy code: `{buggy_tokens}`

Requirements:
1. Replace ONLY tokens {start}-{end}
2. Keep all surrounding code EXACTLY as-is
3. Maintain syntax validity
4. Fix the bug: {bug_description}

Generate ONLY the replacement tokens:
"""
```

---

## Research Methodology

### Training Approach

1. **Localization Model:**
   - Pre-train on code understanding tasks
   - Fine-tune on bug localization datasets
   - Metric: Precision@k for token spans

2. **Fixing Model:**
   - Pre-train on code generation
   - Fine-tune on constrained editing
   - Metric: Test pass rate + exact match

3. **Alignment Module:**
   - Rule-based character offset mapping
   - Validated on diverse tokenizers

---

## Key Metrics to Track

When implementing token-level repair:

```json
{
  "task_id": "bd-123",
  "approach": "toggle",
  "localization": {
    "file": "auth.py",
    "line": 42,
    "tokens": [7, 8],
    "confidence": 0.87,
    "model": "codet5-localization"
  },
  "fix": {
    "original_span": "md5(password)",
    "fixed_span": "bcrypt(password, salt)",
    "tokens_changed": 2,
    "model": "codegen-fix"
  },
  "validation": {
    "tests_passed": true,
    "syntax_valid": true,
    "semantic_equivalent": true
  }
}
```

---

## Key Takeaways

1. **Localization and fixing are different problems** — Use different models/approaches
2. **Token-level beats line-level** — Surgical edits reduce hallucination
3. **Constrained generation is powerful** — "Edit only this span" improves precision
4. **Localization quality dominates** — Wrong location → confidently wrong fix
5. **Separation enables improvement** — Can optimize each component independently

---

## See Also

- `022-chatrepair.md` — Iterative repair with test feedback
- `025-appatch.md` — Program analysis for vulnerability patching
- `026-flames.md` — Semantic-guided search for repair
- `060-debugging-decay-index.md` — Why constrained edits reduce decay
- `057-anthropic-context-engineering.md` — Minimal context principle
