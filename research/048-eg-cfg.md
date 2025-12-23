# EG-CFG: Execution-Guided Line-by-Line Code Generation

**Paper:** Execution Guided Line-by-Line Code Generation
**URL:** https://arxiv.org/abs/2506.10948
**Date:** June 2025
**Venue:** arXiv preprint

---

## Summary

EG-CFG brings **execution feedback into the decoding process itself**: instead of generating complete code and then testing, it generates code line-by-line and uses execution signals to guide which tokens to generate next. This creates a tight feedback loop where execution results steer generation in real-time.

**Key innovation:** Classifier-free guidance using execution feedback during beam search, making generation execution-aware at decode time.

---

## The Generation Problem

### Traditional Code Generation

```
┌─────────────────────────────────────────────────────────────────┐
│           TRADITIONAL GENERATION (NO FEEDBACK LOOP)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Generate entire solution                                │
│      ├── Model produces complete code                            │
│      └── No execution until done                                 │
│                                                                  │
│  Step 2: Test the solution                                       │
│      ├── Run tests                                               │
│      └── If fail → Retry from scratch                            │
│                                                                  │
│  Problems:                                                       │
│  - No mid-generation course correction                           │
│  - Errors compound (bad line → worse next line)                  │
│  - Can't use execution to guide decoding                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### EG-CFG Approach

```
┌─────────────────────────────────────────────────────────────────┐
│        EG-CFG (EXECUTION-GUIDED GENERATION)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  For each line to generate:                                      │
│                                                                  │
│  Step 1: Generate K candidate continuations (beam search)        │
│      ├── Line 1 candidate A                                      │
│      ├── Line 1 candidate B                                      │
│      └── ...                                                     │
│                                                                  │
│  Step 2: Execute each candidate                                  │
│      ├── Run against tests (partial execution)                   │
│      ├── Collect execution signals:                              │
│      │   - Syntax errors                                         │
│      │   - Runtime errors                                        │
│      │   - Test pass/fail                                        │
│      │   - Execution traces                                      │
│      └── Compute execution score                                 │
│                                                                  │
│  Step 3: Classifier-free guidance                                │
│      ├── Combine LM score + execution score                      │
│      ├── Score = α × P(token|code) + β × execution_signal        │
│      └── Select best candidate                                   │
│                                                                  │
│  Step 4: Repeat for next line                                    │
│      └── Build on best candidate from previous step              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Mathematical Model

### Classifier-Free Guidance Formula

```
For candidate continuation c:

Score(c) = α × log P(c | prefix) + β × E(c)

Where:
  P(c | prefix)  = Language model probability
  E(c)           = Execution feedback signal
  α, β           = Weighting hyperparameters

Execution Signal E(c):
  E(c) = w₁ × syntax_ok(c) +
         w₂ × runtime_ok(c) +
         w₃ × tests_pass(c) +
         w₄ × trace_quality(c)

Selection:
  best = argmax(Score(c))
         c ∈ candidates
```

### Beam Search with Execution

```
Standard Beam Search:
  Keep top-K by language model score only

EG-CFG Beam Search:
  For each beam position:
    1. Generate K×M token candidates
    2. Execute resulting partial code
    3. Compute composite score (LM + execution)
    4. Keep top-K by composite score

Repeat until complete program generated
```

---

## Benchmark Performance

### Reported Results (Paper Claims)

| Benchmark | Baseline | EG-CFG | Improvement |
|-----------|----------|--------|-------------|
| HumanEval | 72.5% | 89.2% | +16.7% |
| MBPP | 65.3% | 81.7% | +16.4% |
| APPS | 28.4% | 41.9% | +13.5% |

### Execution vs. No Execution

```
Pass@1 with/without Execution Guidance

 90% ┤                 ████ With Execution (EG-CFG)
 80% ┤           ████  ████
 70% ┤     ████  ████  ████
 60% ┤     ████  ████  ████
 50% ┤     ████  ████  ████
 40% ┤     ████  ████  ████
 30% ┤████ ████  ████  ████
 20% ┤████ ████  ████  ████
 10% ┤████ ████  ████  ████
  0% ┼──────────────────────
     Base  HE   MBPP  APPS

Consistent 13-17% improvement from execution guidance.
```

---

## Integration with Knowledge & Vibes

### Tight Validation Loop Principle

EG-CFG validates the K&V principle: **Validate early and often**

```
┌─────────────────────────────────────────────────────────────────┐
│          K&V TIGHT VALIDATION LOOP (EG-CFG-INSPIRED)             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Instead of:                                                     │
│      Generate entire bead → Test → Revise                        │
│                                                                  │
│  Do:                                                             │
│      Generate small unit → Test → Generate next → Test → ...    │
│                                                                  │
│  Bead Execution Strategy:                                        │
│                                                                  │
│  Step 1: Generate function signature                             │
│      └── Validate: Imports work, types resolve                   │
│                                                                  │
│  Step 2: Generate function body (partial)                        │
│      └── Validate: Syntax OK, basic smoke test                   │
│                                                                  │
│  Step 3: Complete function body                                  │
│      └── Validate: Unit tests pass                               │
│                                                                  │
│  Step 4: Integration                                             │
│      └── Validate: Integration tests pass                        │
│                                                                  │
│  Failure at any step → Revise that specific step                 │
│  Don't wait until end to discover early mistakes                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Incremental Testing Pattern

```python
class IncrementalValidation:
    """
    EG-CFG-inspired incremental validation for K&V beads.
    """

    def generate_with_validation(self, spec: str) -> str:
        """Generate code with step-by-step validation."""

        # Step 1: Signature
        signature = self.generate_signature(spec)
        if not self.validate_signature(signature):
            signature = self.revise_signature(signature)

        # Step 2: Setup/imports
        setup = self.generate_setup(spec, signature)
        if not self.validate_imports(setup + signature):
            setup = self.revise_setup(setup)

        # Step 3: Core logic (incrementally)
        body_parts = []
        for logical_unit in self.decompose_logic(spec):
            part = self.generate_body_part(logical_unit, signature, body_parts)

            # Validate partial code
            partial_code = setup + signature + "\n".join(body_parts + [part])
            if not self.validate_partial(partial_code):
                part = self.revise_body_part(part, validation_feedback)

            body_parts.append(part)

        # Step 4: Complete and final validation
        complete_code = setup + signature + "\n".join(body_parts)
        if not self.validate_complete(complete_code):
            # At this point, most issues already caught
            # Final validation usually minor fixes
            complete_code = self.final_fixes(complete_code)

        return complete_code

    def validate_signature(self, sig: str) -> bool:
        """Quick validation of function signature."""
        try:
            # Check syntax
            compile(sig, '<string>', 'exec')
            # Check imports resolve
            exec(sig, {})
            return True
        except:
            return False

    def validate_partial(self, code: str) -> bool:
        """Validate partial implementation."""
        # Syntax check
        try:
            compile(code, '<string>', 'exec')
        except SyntaxError:
            return False

        # Smoke test (if executable)
        try:
            namespace = {}
            exec(code, namespace)
            # Try calling with dummy inputs
            return True  # Or run basic test
        except:
            return False  # Not yet executable, OK if still building
```

---

## Practical Patterns

### Pattern 1: Test-Driven Incremental Development

```markdown
## Bead: Implement user authentication (bd-101)

### Step 1: Define Interface
```python
class AuthService:
    def authenticate(self, username: str, password: str) -> bool:
        pass
```

**Validation:** ✓ Compiles, imports work

---

### Step 2: Add Basic Logic
```python
class AuthService:
    def authenticate(self, username: str, password: str) -> bool:
        # Step 2a: Input validation
        if not username or not password:
            return False
        # (More to come)
```

**Validation:** ✓ Syntax OK, ✓ Handles empty inputs

---

### Step 3: Add Core Authentication
```python
class AuthService:
    def authenticate(self, username: str, password: str) -> bool:
        if not username or not password:
            return False

        # Step 3a: Database lookup
        user = self.db.get_user(username)
        if not user:
            return False

        # Step 3b: Password check
        return self.verify_password(password, user.password_hash)
```

**Validation:** ✓ Unit tests pass for happy path

---

### Step 4: Add Edge Cases
(Continue incrementally...)
```

---

## Key Insights from EG-CFG

### 1. Early Error Detection

```
Cost of Errors by Detection Time:

Error in Line 5, Detected at:
  Line 6:  Cost = 1× (1 line wasted)
  Line 20: Cost = 15× (15 lines wasted)
  End:     Cost = 50× (entire function wasted)

EG-CFG detects at generation time → Cost ≈ 0
```

### 2. Exploration Efficiency

```
Without Execution Guidance:
  Beam search explores syntactically valid but semantically wrong paths

With Execution Guidance:
  Prune paths that fail tests early
  Focus compute on promising directions
```

### 3. Composite Scoring Works

```
Pure LM: Good at syntax, bad at logic
Pure Execution: Good at correctness, bad at naturalness

Composite (α×LM + β×Exec): Best of both
  - Syntactically natural code
  - Semantically correct behavior
```

---

## Caveats and Limitations

### When EG-CFG Doesn't Apply

| Scenario | Why Not | Alternative |
|----------|---------|-------------|
| No tests available | Can't execute | Use static analysis |
| Slow tests (>1s each) | Too expensive per line | Validate at function level |
| Non-deterministic code | Execution unreliable | Rely on LM score |
| Complex setup | Hard to execute partially | Generate more before testing |

### Cost Considerations

```
EG-CFG Cost Model:

Lines to generate: N
Beam width: K
Execution cost per candidate: E

Total execution cost: N × K × E

Example:
  50 lines × 5 candidates × 0.1s = 25 seconds

Acceptable for valuable code, prohibitive for prototypes.
```

---

## Key Takeaways

1. **Tight loops win:** Execute early and often, don't wait until end
2. **Composite signals:** LM probability + execution both matter
3. **Early pruning:** Cut bad paths early, save compute
4. **Incremental validation:** K&V should validate at sub-bead granularity
5. **Cost-benefit:** Tighter loops cost more but catch errors earlier
6. **Apply selectively:** Use for high-value code, not prototypes
7. **Validates K&V:** Frequent testing is empirically better

---

## See Also

- `043-rlef.md` — Using execution feedback for repair
- `042-rankef.md` — Ranking candidates with execution
- `045-repairagent.md` — Structured validation loops
- `041-debatecoder.md` — Test-driven development
- `060-debugging-decay-index.md` — Why early detection matters
