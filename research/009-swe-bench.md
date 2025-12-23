---
title: SWE-bench - GitHub Issue Resolution
tags: [benchmark, localization, multi-file, execution-based, github]
protocols: [P6, P8]
---

# SWE-bench: Real-World GitHub Issue Resolution

**Paper:** SWE-bench: Can Language Models Resolve Real-World GitHub Issues?
**URL:** https://ar5iv.labs.arxiv.org/html/2310.06770
**Date:** October 2023
**Venue:** ICLR 2024

---

## Summary

SWE-bench fundamentally reframes AI coding evaluation from "write a standalone function" to "resolve a real GitHub issue in a production codebase." It tests whether LLMs can navigate large repositories, localize bugs, coordinate multi-file changes, and produce patches that pass execution-based tests.

**Key innovation:** Each task is built from a real merged PR, ensuring problems are authentic and solutions are execution-validated.

**Key finding:** Even with strong models and retrieval assistance, solve rates remain low - exposing that **localization + correct repo-context editing** is the primary bottleneck, not just code generation ability.

---

## The Real-World Gap

### Why Standalone Benchmarks Don't Suffice

```
┌─────────────────────────────────────────────────────────────────┐
│           STANDALONE vs REAL-WORLD CODING                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Standalone Benchmarks (HumanEval, MBPP):                        │
│  ├── Write self-contained function                               │
│  ├── Clear input/output spec                                     │
│  ├── No external dependencies                                    │
│  └── Single file, ~10-50 lines                                   │
│                                                                  │
│  Real-World Tasks (SWE-bench):                                   │
│  ├── Navigate 100+ file repository                               │
│  ├── Understand issue from natural language                      │
│  ├── Localize bug across modules                                 │
│  ├── Coordinate multi-file edits                                 │
│  ├── Respect project conventions                                 │
│  ├── Work with existing APIs                                     │
│  └── Pass integration tests                                      │
│                                                                  │
│  The gap is MASSIVE.                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Skills Required Beyond Code Generation

| Skill | HumanEval Needs It? | SWE-bench Needs It? |
|-------|---------------------|---------------------|
| Code generation | Yes | Yes |
| **Repository navigation** | No | **Critical** |
| **Bug localization** | No | **Critical** |
| **Multi-file coordination** | No | **Critical** |
| **API understanding** | Minimal | **Essential** |
| **Test interpretation** | No | **Essential** |
| **Project conventions** | No | **Important** |

---

## Dataset Construction

### What Counts as a Task

Each SWE-bench instance is built from a real merged pull request:

```
┌─────────────────────────────────────────────────────────────────┐
│                     TASK CONSTRUCTION PIPELINE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Find merged PR in real repository                            │
│     └── Must link to one or more GitHub issues                   │
│                                                                  │
│  2. Extract components                                           │
│     ├── Codebase (C): Repository at PR's base commit             │
│     ├── Problem (P): Issue title + description                   │
│     ├── Tests (T): Test suite from fixed version                 │
│     └── Gold patch (δ): The PR's code changes                    │
│                                                                  │
│  3. Execution validation                                         │
│     ├── Tests without patch: At least one FAIL                   │
│     ├── Tests with patch: All FAIL_TO_PASS tests pass            │
│     └── Reject if validation fails                               │
│                                                                  │
│  4. Create task                                                  │
│     └── Agent receives C and P, must produce patch δ̂             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Dataset Statistics

**SWE-bench full:**
- 2,294 tasks from 12 popular Python repositories
- Real GitHub issues from 2012-2023
- Average repository size: 50,000+ lines of code
- Average fix: 5-20 lines across 1-3 files

```
Repository Distribution (SWE-bench full):

django        ████████████ 580 tasks (25%)
scikit-learn  ██████████ 450 tasks (20%)
matplotlib    ████████ 378 tasks (16%)
requests      ████ 185 tasks (8%)
pytest        ████ 172 tasks (8%)
sympy         ███ 142 tasks (6%)
flask         ██ 98 tasks (4%)
pandas        ██ 87 tasks (4%)
sphinx        ██ 76 tasks (3%)
astropy       █ 54 tasks (2%)
seaborn       █ 42 tasks (2%)
pylint        █ 30 tasks (1%)

Total: 12 repositories, 2,294 tasks
```

### Task Anatomy

```
Example Task: django-15789

Codebase (C):
├── django/
│   ├── core/
│   ├── db/
│   │   ├── models/
│   │   │   ├── query.py       ← Bug location
│   │   │   └── ...
│   │   └── ...
│   └── ...
└── tests/
    └── queries/
        └── test_q.py           ← Test that validates fix

Problem Statement (P):
"QuerySet.only() with select_related() doesn't work properly
on reverse OneToOneField relations. The query generates
incorrect SQL that fails with FieldError."

Gold Patch (δ):
File: django/db/models/query.py
Lines: 1287-1295
+        if field.remote_field:
+            field = field.remote_field
Changes: 8 lines modified across 1 file

Tests (T):
- tests/queries/test_q.py::TestOnlySelectRelated
  FAIL_TO_PASS: test_only_with_select_related_reverse_o2o
  PASS_TO_PASS: test_only_basic, test_select_related_basic
```

---

## Evaluation Protocol

### What "Solved" Means

```
┌─────────────────────────────────────────────────────────────────┐
│                    EVALUATION WORKFLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Agent produces patch δ̂                                       │
│                                                                  │
│  2. Apply patch to codebase C                                    │
│     └── If patch doesn't apply: FAIL                             │
│                                                                  │
│  3. Run test suite T                                             │
│     ├── Record which tests pass/fail                             │
│     └── Compare to gold patch behavior                           │
│                                                                  │
│  4. Task is RESOLVED iff:                                        │
│     ├── All FAIL_TO_PASS tests now pass                          │
│     └── All PASS_TO_PASS tests still pass                        │
│                                                                  │
│  5. Crucially: Don't judge by diff similarity                    │
│     └── Only behavior matters (tests pass/fail)                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Why Execution-Based Validation Matters

**Surface similarity vs behavioral correctness:**

```python
# Gold patch (from real PR)
if field.remote_field:
    field = field.remote_field

# Agent patch (different code, same behavior)
field = getattr(field, 'remote_field', field)

# Both pass tests → Both CORRECT
# Traditional diff-based eval would mark agent as WRONG
```

**Key principle:** Code is a means to an end. If tests pass, the issue is resolved.

---

## Baseline Results

### Original Paper Performance

**With oracle retrieval (perfect file localization):**

| Model | Pass@1 | Pass@10 |
|-------|--------|---------|
| GPT-4 | 12.5% | ~25% |
| GPT-3.5 | 4.3% | ~10% |
| Claude 2 | 8.1% | ~18% |

**Without retrieval (navigate yourself):**

| Model | Pass@1 |
|-------|--------|
| GPT-4 | 2.8% |
| GPT-3.5 | 0.9% |

```
Impact of File Localization

With oracle retrieval: ████████████ 12.5%
Without retrieval:      ███ 2.8%

Interpretation: Finding the right files is the primary bottleneck
```

### The Localization Bottleneck

**Where agents fail (from paper analysis):**

```
Failure Mode Distribution:

Wrong file location:    ████████████████████ 45%
Incorrect patch logic:  ████████████ 28%
Incomplete fix:         ███████ 15%
Patch doesn't apply:    ████ 8%
Breaks existing tests:  ██ 4%

Conclusion: Navigation and localization are harder than fixing
```

---

## Evolution of SWE-bench

### Variants Addressing Specific Issues

| Variant | Release | Purpose | Size |
|---------|---------|---------|------|
| **SWE-bench** | Oct 2023 | Original benchmark | 2,294 tasks |
| **SWE-bench Lite** | Apr 2024 | Curated, cleaner subset | 300 tasks |
| **SWE-bench Verified** | Sep 2024 | Human-verified, decontaminated | 500 tasks |
| **SWE-bench+** | Oct 2024 | Extended with more repos | 5,000+ tasks |
| **SWE-rebench** | Nov 2024 | Fresh tasks (post-contamination) | 1,500 tasks |

### Why Multiple Variants?

**SWE-bench Lite:**
- Faster iteration for development
- Lower variance in task difficulty
- Better for prototyping approaches

**SWE-bench Verified:**
- Addresses contamination concerns
- Human review ensures quality
- Official leaderboard benchmark

**SWE-rebench:**
- Created after major models trained on SWE-bench
- Ensures results aren't inflated by test set memorization

---

## The Localization Problem

### Repository Navigation Challenge

```
┌─────────────────────────────────────────────────────────────────┐
│                  MULTI-STEP LOCALIZATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Given: Issue description + 50,000 line repository               │
│  Task: Find the 20 lines to modify                               │
│                                                                  │
│  Step 1: Identify relevant modules                               │
│     50,000 lines → ~500 files → Which 10 files matter?           │
│                                                                  │
│  Step 2: Find bug location within files                          │
│     10 files × 100 lines → Which functions/classes?              │
│                                                                  │
│  Step 3: Understand code context                                 │
│     Read surrounding code, imports, dependencies                 │
│                                                                  │
│  Step 4: Identify related files                                  │
│     Tests, callers, related modules                              │
│                                                                  │
│  Human expert: ~30 minutes                                       │
│  LLM agent: Often fails completely                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Oracle Retrieval Experiments

The paper tested with "oracle retrieval" - giving the agent the exact files to modify:

**Impact of perfect localization:**

```
Performance with Perfect File Localization

Zero-shot (find files yourself):    ███ 2.8%
Oracle (files given):                ████████████ 12.5%

Improvement: +9.7 percentage points (347% relative increase)

Interpretation: If we solve localization, we 4x performance
```

---

## Multi-File Coordination

### The Dependency Challenge

Real fixes often require coordinating changes:

```
Example: Adding a parameter to a function

File 1: function definition
├── def process(data):  →  def process(data, encoding='utf-8'):

File 2: function call in module A
├── process(data)  →  process(data, encoding=enc)

File 3: function call in module B
├── process(raw)  →  process(raw, encoding='latin-1')

File 4: tests
├── Add tests for encoding parameter

Agent must:
1. Modify all call sites consistently
2. Maintain backward compatibility or update all callers
3. Add/update tests
4. Ensure no other code breaks
```

**Multi-file coordination statistics:**

| Files Changed | Percentage | Avg Pass@1 |
|---------------|------------|------------|
| 1 file | 65% | 8.2% |
| 2-3 files | 28% | 4.1% |
| 4+ files | 7% | 1.3% |

**Interpretation:** Multi-file tasks are 6x harder than single-file tasks.

---

## Practical Implications

### For Agent Development

**Critical capabilities identified:**

1. **Repository understanding**
   - Build mental model of codebase structure
   - Identify module boundaries
   - Understand dependency relationships

2. **Semantic search**
   - Find relevant code by meaning, not keywords
   - Connect issue description to code locations
   - Discover related files

3. **Context management**
   - Load only relevant files into context
   - Prioritize information by relevance
   - Hierarchical code exploration

4. **Patch construction**
   - Generate syntactically valid patches
   - Ensure patches apply cleanly
   - Test patches before submission

5. **Verification**
   - Run tests locally
   - Interpret test failures
   - Iterate based on feedback

### For K&V Workflow

| K&V Phase | SWE-bench Insight |
|-----------|-------------------|
| **Planning** | Explicit localization bead |
| **Execution** | Multi-file coordination tracking |
| **Validation** | Execution-based acceptance criteria |
| **ADaPT** | Decompose localization from implementation |

---

## Integration with K&V

### Localization as Dedicated Phase

```
┌─────────────────────────────────────────────────────────────────┐
│              SWE-BENCH INSPIRED WORKFLOW                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: Understanding                                          │
│  ├── Read issue description                                      │
│  ├── Understand expected behavior                                │
│  └── Output: Problem understanding                               │
│                                                                  │
│  Phase 2: Localization (CRITICAL)                                │
│  ├── Search codebase for relevant files                          │
│  ├── Identify bug locations                                      │
│  ├── Find related code (callers, tests)                          │
│  └── Output: File list + locations                               │
│                                                                  │
│  Phase 3: Fix Design                                             │
│  ├── Plan changes across files                                   │
│  ├── Identify test requirements                                  │
│  └── Output: Multi-file change plan                              │
│                                                                  │
│  Phase 4: Implementation                                         │
│  ├── Make coordinated changes                                    │
│  ├── Update tests                                                │
│  └── Output: Patch                                               │
│                                                                  │
│  Phase 5: Validation                                             │
│  ├── Apply patch                                                 │
│  ├── Run tests (FAIL_TO_PASS and PASS_TO_PASS)                  │
│  └── Verify resolution                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Acceptance Criteria Pattern

```markdown
## Bead: Fix QuerySet.only() with reverse OneToOneField

**Acceptance criteria:**
1. FAIL_TO_PASS tests:
   - tests/queries/test_q.py::test_only_with_select_related_reverse_o2o
2. PASS_TO_PASS tests:
   - tests/queries/test_q.py::test_only_basic
   - tests/queries/test_q.py::test_select_related_basic
3. No new test failures
4. Patch applies cleanly

**Validation:** Run `pytest tests/queries/test_q.py -v`
```

---

## Code Examples

### Example 1: Test-Based Validation

```python
import subprocess
from dataclasses import dataclass
from typing import Optional

@dataclass
class TestResult:
    passed: bool
    fail_to_pass: list[str]  # Tests that should pass after fix
    pass_to_pass: list[str]  # Tests that should still pass
    output: str

class SWEBenchValidator:
    """
    Validate patches using SWE-bench criteria.
    """

    def __init__(self, repo_path: str):
        self.repo_path = repo_path

    def validate_patch(
        self,
        patch_path: str,
        fail_to_pass_tests: list[str],
        pass_to_pass_tests: list[str]
    ) -> TestResult:
        """
        Validate that patch resolves issue per SWE-bench criteria.
        """
        # 1. Apply patch
        apply_result = subprocess.run(
            ['git', 'apply', patch_path],
            cwd=self.repo_path,
            capture_output=True,
            text=True
        )

        if apply_result.returncode != 0:
            return TestResult(
                passed=False,
                fail_to_pass=[],
                pass_to_pass=[],
                output=f"Patch failed to apply: {apply_result.stderr}"
            )

        # 2. Run FAIL_TO_PASS tests (should now pass)
        fail_to_pass_results = self._run_tests(fail_to_pass_tests)

        # 3. Run PASS_TO_PASS tests (should still pass)
        pass_to_pass_results = self._run_tests(pass_to_pass_tests)

        # 4. Revert patch
        subprocess.run(['git', 'apply', '-R', patch_path], cwd=self.repo_path)

        # 5. Check criteria
        all_fail_to_pass_passed = all(
            result['passed'] for result in fail_to_pass_results.values()
        )
        all_pass_to_pass_passed = all(
            result['passed'] for result in pass_to_pass_results.values()
        )

        passed = all_fail_to_pass_passed and all_pass_to_pass_passed

        return TestResult(
            passed=passed,
            fail_to_pass=[
                test for test, result in fail_to_pass_results.items()
                if result['passed']
            ],
            pass_to_pass=[
                test for test, result in pass_to_pass_results.items()
                if result['passed']
            ],
            output=self._format_output(fail_to_pass_results, pass_to_pass_results)
        )

    def _run_tests(self, tests: list[str]) -> dict:
        """Run tests and return results."""
        results = {}

        for test in tests:
            result = subprocess.run(
                ['pytest', test, '-xvs'],
                cwd=self.repo_path,
                capture_output=True,
                text=True
            )

            results[test] = {
                'passed': result.returncode == 0,
                'output': result.stdout + result.stderr
            }

        return results

    def _format_output(self, fail_to_pass, pass_to_pass) -> str:
        """Format test results for display."""
        output = []

        output.append("FAIL_TO_PASS tests:")
        for test, result in fail_to_pass.items():
            status = "✓ PASS" if result['passed'] else "✗ FAIL"
            output.append(f"  {status} {test}")

        output.append("\nPASS_TO_PASS tests:")
        for test, result in pass_to_pass.items():
            status = "✓ PASS" if result['passed'] else "✗ FAIL"
            output.append(f"  {status} {test}")

        return "\n".join(output)

# Usage
validator = SWEBenchValidator('/path/to/django')

result = validator.validate_patch(
    patch_path='fix.patch',
    fail_to_pass_tests=[
        'tests/queries/test_q.py::test_only_with_select_related_reverse_o2o'
    ],
    pass_to_pass_tests=[
        'tests/queries/test_q.py::test_only_basic',
        'tests/queries/test_q.py::test_select_related_basic'
    ]
)

print(result.output)
print(f"\n{'RESOLVED' if result.passed else 'NOT RESOLVED'}")
```

### Example 2: File Localization

```python
from pathlib import Path
from typing import List, Tuple
import re

class FileLocalizer:
    """
    Locate files relevant to a GitHub issue.
    Simulates the localization phase of SWE-bench tasks.
    """

    def __init__(self, repo_path: Path):
        self.repo_path = repo_path

    def localize(self, issue_description: str, max_files: int = 10) -> List[Tuple[Path, float]]:
        """
        Find files most likely related to the issue.
        Returns list of (file_path, score) tuples.
        """
        # Extract keywords from issue
        keywords = self._extract_keywords(issue_description)

        # Search codebase
        candidates = []

        for py_file in self.repo_path.rglob('*.py'):
            if self._should_skip(py_file):
                continue

            score = self._score_file(py_file, keywords)
            if score > 0:
                candidates.append((py_file, score))

        # Sort by score, return top N
        candidates.sort(key=lambda x: x[1], reverse=True)
        return candidates[:max_files]

    def _extract_keywords(self, issue: str) -> set[str]:
        """Extract meaningful keywords from issue description."""
        # Remove common words
        stop_words = {'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at'}

        # Extract words
        words = re.findall(r'\b\w+\b', issue.lower())

        # Filter
        keywords = {w for w in words if w not in stop_words and len(w) > 3}

        # Add exact phrases (class names, function names)
        keywords.update(re.findall(r'\b[A-Z]\w+\b', issue))  # CamelCase
        keywords.update(re.findall(r'\b\w+_\w+\b', issue))  # snake_case

        return keywords

    def _should_skip(self, file_path: Path) -> bool:
        """Skip test files, migrations, etc. during localization."""
        skip_patterns = [
            'test_', 'tests/', 'migrations/', '__pycache__',
            'venv/', '.git/', 'build/', 'dist/'
        ]
        return any(pattern in str(file_path) for pattern in skip_patterns)

    def _score_file(self, file_path: Path, keywords: set[str]) -> float:
        """Score file relevance based on keyword matches."""
        try:
            content = file_path.read_text()
        except:
            return 0.0

        score = 0.0

        # Exact keyword matches
        for keyword in keywords:
            # Weight by match type
            if re.search(rf'\bclass {keyword}\b', content):
                score += 10  # Class definition
            elif re.search(rf'\bdef {keyword}\b', content):
                score += 8   # Function definition
            elif keyword in content:
                score += 1   # General mention

        # Boost score based on file structure
        lines = content.split('\n')
        if len(lines) < 1000:  # Prefer focused files
            score *= 1.2

        return score

# Usage
localizer = FileLocalizer(Path('/path/to/django'))

issue = """
QuerySet.only() with select_related() doesn't work properly
on reverse OneToOneField relations. The query generates
incorrect SQL that fails with FieldError.
"""

files = localizer.localize(issue, max_files=5)

print("Most relevant files:")
for file_path, score in files:
    print(f"  {score:6.1f} {file_path.relative_to(localizer.repo_path)}")
```

### Example 3: Multi-File Patch Coordinator

```python
from dataclasses import dataclass
from typing import List, Dict
import subprocess

@dataclass
class FileChange:
    file_path: str
    changes: str  # Diff format
    reason: str

class MultiFilePatchCoordinator:
    """
    Coordinate changes across multiple files for SWE-bench tasks.
    """

    def __init__(self, repo_path: str):
        self.repo_path = repo_path
        self.changes: List[FileChange] = []

    def add_change(self, file_path: str, changes: str, reason: str):
        """Add a file change to the coordinated patch."""
        self.changes.append(FileChange(file_path, changes, reason))

    def validate_consistency(self) -> List[str]:
        """Check for inconsistencies across files."""
        issues = []

        # Check 1: Function signature changes are consistent
        # (simplified - real implementation would parse AST)
        function_changes = {}
        for change in self.changes:
            # Extract function definitions
            func_defs = re.findall(r'def (\w+)\([^)]*\):', change.changes)
            for func in func_defs:
                if func in function_changes:
                    if function_changes[func] != change.changes:
                        issues.append(
                            f"Inconsistent changes to function '{func}' "
                            f"in {change.file_path}"
                        )
                function_changes[func] = change.changes

        # Check 2: All modified functions have corresponding test changes
        modified_functions = set(function_changes.keys())
        test_files = [c for c in self.changes if 'test' in c.file_path]

        if modified_functions and not test_files:
            issues.append("Modified functions but no test changes")

        return issues

    def generate_patch(self) -> str:
        """Generate unified patch from all changes."""
        patch_parts = []

        for change in self.changes:
            patch_parts.append(f"# {change.reason}")
            patch_parts.append(f"--- a/{change.file_path}")
            patch_parts.append(f"+++ b/{change.file_path}")
            patch_parts.append(change.changes)
            patch_parts.append("")

        return "\n".join(patch_parts)

    def apply_and_test(self, test_commands: List[str]) -> bool:
        """Apply patch and run tests."""
        patch = self.generate_patch()

        # Write patch to file
        patch_file = Path(self.repo_path) / 'changes.patch'
        patch_file.write_text(patch)

        # Apply
        result = subprocess.run(
            ['git', 'apply', str(patch_file)],
            cwd=self.repo_path,
            capture_output=True
        )

        if result.returncode != 0:
            print(f"Patch failed to apply: {result.stderr.decode()}")
            return False

        # Run tests
        all_passed = True
        for test_cmd in test_commands:
            result = subprocess.run(
                test_cmd.split(),
                cwd=self.repo_path,
                capture_output=True
            )
            if result.returncode != 0:
                print(f"Test failed: {test_cmd}")
                all_passed = False

        # Revert
        subprocess.run(
            ['git', 'apply', '-R', str(patch_file)],
            cwd=self.repo_path
        )

        return all_passed

# Usage
coordinator = MultiFilePatchCoordinator('/path/to/django')

# Add changes
coordinator.add_change(
    file_path='django/db/models/query.py',
    changes='''@@ -1287,7 +1287,9 @@
+        if field.remote_field:
+            field = field.remote_field
''',
    reason='Fix OneToOneField reverse relation handling'
)

coordinator.add_change(
    file_path='tests/queries/test_q.py',
    changes='''@@ -45,6 +45,12 @@
+    def test_only_with_select_related_reverse_o2o(self):
+        # Test for issue #15789
+        obj = Model.objects.only('id').select_related('reverse_rel')[0]
+        self.assertEqual(obj.id, 1)
''',
    reason='Add test for OneToOneField reverse relation fix'
)

# Validate
issues = coordinator.validate_consistency()
if issues:
    print("Consistency issues:")
    for issue in issues:
        print(f"  - {issue}")

# Test
success = coordinator.apply_and_test([
    'pytest tests/queries/test_q.py::test_only_with_select_related_reverse_o2o'
])
print(f"Patch {'PASSES' if success else 'FAILS'} tests")
```

---

## Key Takeaways

1. **Real-world coding is about navigation** - Finding the right files is harder than writing the fix

2. **Localization is the bottleneck** - Oracle retrieval improves performance 4x (2.8% → 12.5%)

3. **Multi-file coordination is hard** - Tasks requiring 4+ files are 6x harder than single-file tasks

4. **Execution validation is essential** - Surface similarity to gold patch doesn't matter; tests do

5. **Tool use is mandatory** - Search, navigation, and testing tools are as important as the LLM

6. **Context management matters** - Can't load 50K lines; must prioritize intelligently

7. **Decomposition helps** - Separate localization, design, implementation, and validation phases

---

## See Also

- `007-swe-gym.md` - Training environment complementing SWE-bench evaluation
- `010-swe-agent.md` - Agent-computer interfaces for better SWE-bench performance
- `038-adapt.md` - Decomposition strategies for complex tasks
- `065-confucius-code-agent.md` - Scaling agents to large codebases
- `060-debugging-decay-index.md` - Why iteration limits matter
