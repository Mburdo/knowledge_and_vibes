---
title: CodePlan - Repository-level Planning
tags: [planning, repository-level, cascade, dependencies, oracle, adaptive]
protocols: [P4, P8]
---

# CodePlan: Repository-level Planning + Chain of Edits

**Paper:** CodePlan: Repository-level Coding using LLMs and Planning
**URL:** https://dl.acm.org/doi/10.1145/3643757
**Date:** FSE 2024 (paper published September 2023)
**Venue:** FSE 2024

---

## Summary

Pioneering research framing repository-level changes as a **planning problem** rather than a single-shot editing task. CodePlan demonstrates that multi-file migrations, refactors, and cascading fixes require **adaptive plan execution** where the plan expands dynamically based on downstream validation failures.

**Key insight:** The correct unit of work is often a chain of interdependent edits, not a single patch. Plans should adapt as consequences emerge.

---

## The Repository-Level Challenge

### Why Single-Shot Editing Fails

```
Scenario: Migrate from old_api.connect() to new_api.create_connection()

Naive Approach:
├─ Find all calls to old_api.connect()
├─ Replace with new_api.create_connection()
└─ Result: BUILD FAILS

Why?
├─ Import statements not updated
├─ Parameter names changed (host → hostname)
├─ Return type different (Connection → ConnectionPool)
├─ Error handling changed (raises → returns Result)
└─ Tests use old API patterns

Each change creates new obligations elsewhere
```

### The Cascade Problem

```
┌─────────────────────────────────────────────────────────────────┐
│                    EDIT CASCADE EFFECT                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Edit 1: Update function signature                              │
│  ├─ Changes return type                                         │
│  └─ CONSEQUENCE: All callers need updating                      │
│                                                                  │
│  Edit 2: Update caller A                                        │
│  ├─ Changes how result is used                                  │
│  └─ CONSEQUENCE: Error handling logic changes                   │
│                                                                  │
│  Edit 3: Update error handling                                  │
│  ├─ Imports new exception types                                 │
│  └─ CONSEQUENCE: Import statements need updating                │
│                                                                  │
│  Edit 4: Update imports                                         │
│  ├─ Module structure changed                                    │
│  └─ CONSEQUENCE: Package dependencies need updating             │
│                                                                  │
│  Edit 5: Update dependencies                                    │
│  └─ COMPLETE (for now...)                                       │
│                                                                  │
│  Note: Each edit surfaces new required edits                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## The CodePlan Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      CODEPLAN SYSTEM                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  INITIALIZATION                                           │   │
│  │                                                           │   │
│  │  Input: Task description (e.g., "migrate API X to Y")    │   │
│  │  Output: Seed edit specifications                        │   │
│  │                                                           │   │
│  │  Example:                                                 │   │
│  │  - Replace old_api.connect() in file1.py                 │   │
│  │  - Update imports in file1.py                            │   │
│  │  - Update tests in test_file1.py                         │   │
│  │                                                           │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  DEPENDENCY ANALYSIS                                      │   │
│  │                                                           │   │
│  │  ├─ Build call graph                                     │   │
│  │  ├─ Track import relationships                           │   │
│  │  ├─ Identify type dependencies                           │   │
│  │  └─ Map test-to-code coverage                            │   │
│  │                                                           │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ADAPTIVE EXECUTION LOOP                                  │   │
│  │                                                           │   │
│  │  While obligations exist:                                │   │
│  │                                                           │   │
│  │    1. Select next edit from obligations queue            │   │
│  │                                                           │   │
│  │    2. Gather context (spatial + temporal)                │   │
│  │       ├─ Spatial: Related code (dependencies, callers)   │   │
│  │       └─ Temporal: What has already changed              │   │
│  │                                                           │   │
│  │    3. Generate edit with LLM                             │   │
│  │                                                           │   │
│  │    4. Apply edit to working copy                         │   │
│  │                                                           │   │
│  │    5. Run oracle (build/typecheck/tests)                 │   │
│  │                                                           │   │
│  │    6. Analyze oracle output                              │   │
│  │       ├─ Success? → Continue                             │   │
│  │       └─ Failure? → Extract new obligations             │   │
│  │                                                           │   │
│  │    7. Update dependency graph                            │   │
│  │                                                           │   │
│  │    8. May-impact propagation                             │   │
│  │       └─ What else might need changing?                  │   │
│  │                                                           │   │
│  │  End while                                               │   │
│  │                                                           │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  VALIDATION                                               │   │
│  │                                                           │   │
│  │  ├─ All tests pass?                                      │   │
│  │  ├─ Build succeeds?                                      │   │
│  │  ├─ Type checking clean?                                 │   │
│  │  └─ No regression?                                       │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Spatial + Temporal Context

### The Two Dimensions of Context

| Context Type | What It Provides | Example |
|--------------|------------------|---------|
| **Spatial** | Related code at this moment | "What files/functions are related to this edit location?" |
| **Temporal** | What has changed so far | "What edits have we already made that affect this code?" |

### Spatial Context

```python
# Editing: update_user() function

Spatial Context:
├─ Callees (dependencies)
│  ├─ validate_user_data()
│  ├─ database.save()
│  └─ send_notification()
│
├─ Callers (usage sites)
│  ├─ user_registration_handler()
│  ├─ admin_user_update()
│  └─ batch_user_import()
│
├─ Type dependencies
│  ├─ User class definition
│  ├─ UserSchema validator
│  └─ DatabaseConnection type
│
└─ Related tests
   ├─ test_update_user_success()
   ├─ test_update_user_validation_error()
   └─ test_update_user_database_failure()
```

### Temporal Context

```python
# What has changed so far in this plan execution?

Temporal Context:
├─ Edit 1 (5 minutes ago):
│  └─ Changed database.save() signature in db/connection.py
│
├─ Edit 2 (3 minutes ago):
│  └─ Updated import for database in models/user.py
│
└─ Edit 3 (1 minute ago):
   └─ Fixed call to database.save() in models/post.py

Current edit: Fix call to database.save() in models/user.py
Context: Need to use new signature with timeout parameter
```

### Why Both Matter

```
Without Spatial Context:
├─ Don't know what else depends on this code
├─ Can't predict cascade effects
└─ Make changes that break dependencies

Without Temporal Context:
├─ Don't know what has already changed
├─ Generate edits inconsistent with prior changes
└─ Create conflicts or redundant work

With Both:
├─ Know what's related (spatial)
├─ Know what's changed (temporal)
└─ Generate consistent, complete edits
```

---

## May-Impact Propagation

### Predicting Cascade Effects

```
┌─────────────────────────────────────────────────────────────────┐
│                  MAY-IMPACT ANALYSIS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Edit: Change function signature                                │
│  ├─ def authenticate(user, password)                            │
│  └─ def authenticate(user, password, session_token=None)        │
│                                                                  │
│  May-Impact Propagation:                                        │
│                                                                  │
│  Direct Impact (certain):                                       │
│  ├─ All call sites must be checked                             │
│  ├─ Docstring needs updating                                   │
│  └─ Tests need new parameter cases                             │
│                                                                  │
│  Indirect Impact (probable):                                    │
│  ├─ API documentation needs updating                           │
│  ├─ Client libraries may need changes                          │
│  └─ Migration guide for users                                  │
│                                                                  │
│  Potential Impact (possible):                                   │
│  ├─ Performance characteristics may change                     │
│  ├─ Security model may be affected                             │
│  └─ Logging/monitoring may need adjustment                     │
│                                                                  │
│  Generate New Obligations:                                      │
│  ├─ HIGH priority: Update call sites (direct)                  │
│  ├─ MEDIUM priority: Update tests (direct)                     │
│  └─ LOW priority: Check performance (indirect)                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Algorithm

```python
def may_impact_propagation(edit, dependency_graph):
    """
    Analyze what else might need changing after this edit.
    """
    impacted = {
        'direct': [],      # Certain to need changes
        'indirect': [],    # Probably need changes
        'potential': []    # Possibly need changes
    }

    # Direct impact: syntactic dependencies
    if edit.type == 'signature_change':
        # All callers must be checked
        impacted['direct'].extend(
            dependency_graph.find_callers(edit.function)
        )
        # Tests covering this function
        impacted['direct'].extend(
            dependency_graph.find_tests_for(edit.function)
        )

    elif edit.type == 'type_change':
        # All code using this type
        impacted['direct'].extend(
            dependency_graph.find_usages(edit.type_name)
        )

    # Indirect impact: semantic dependencies
    if edit.changes_behavior:
        # Code that depends on old behavior
        impacted['indirect'].extend(
            dependency_graph.find_behavioral_dependents(edit.function)
        )

    # Potential impact: architectural concerns
    if edit.scope == 'api_change':
        impacted['potential'].extend([
            'documentation',
            'client_libraries',
            'migration_guides'
        ])

    return impacted
```

---

## Oracle-Driven Planning

### The Oracle as Truth Source

```
┌─────────────────────────────────────────────────────────────────┐
│                     ORACLE FEEDBACK LOOP                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Apply Edit → Run Oracle → Analyze Results                      │
│                                                                  │
│  Oracle Types:                                                   │
│  ├─ Build system (compilation errors)                           │
│  ├─ Type checker (type errors)                                  │
│  ├─ Test suite (behavioral errors)                              │
│  └─ Linter (style/convention errors)                            │
│                                                                  │
│  Oracle Output → New Obligations:                               │
│                                                                  │
│  Example: Build Failure                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Error: NameError: name 'old_api' is not defined           │ │
│  │ File: src/handlers/user.py, line 42                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                           ↓                                      │
│  New Obligation:                                                 │
│  ├─ File: src/handlers/user.py                                  │
│  ├─ Issue: Missing import for old_api                           │
│  ├─ Fix: Add import or replace with new_api                     │
│  └─ Priority: HIGH (blocking)                                   │
│                                                                  │
│  Example: Type Error                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Error: Argument 'timeout' missing in call to connect()    │ │
│  │ File: src/database/pool.py, line 156                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                           ↓                                      │
│  New Obligation:                                                 │
│  ├─ File: src/database/pool.py                                  │
│  ├─ Issue: Call signature doesn't match new definition          │
│  ├─ Fix: Add timeout parameter to connect() call                │
│  └─ Priority: HIGH (type error)                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Oracle Quality Matters

| Oracle Strength | Plan Quality | Example |
|----------------|--------------|---------|
| **Strong** (fast, precise) | High-quality feedback, rapid iteration | Rust compiler with detailed errors |
| **Medium** (slow or noisy) | Useful but limited | Python with partial type hints |
| **Weak** (absent or brittle) | Limited guidance, more speculation | Bash scripts with no tests |

---

## Performance Results

### Repository-Level Tasks

```
Benchmark: Multi-file API Migration
(10 repositories, 50-500 files each)

┌──────────────────┬─────────┬─────────┬──────────┐
│     System       │ Success │  Time   │  Edits   │
├──────────────────┼─────────┼─────────┼──────────┤
│ CodePlan         │  8/10   │  12 min │   47     │
│ Manual (expert)  │ 10/10   │  45 min │   52     │
│ GPT-4 (direct)   │  1/10   │   5 min │   12     │
│ Template-based   │  4/10   │  30 min │   38     │
└──────────────────┴─────────┴─────────┴──────────┘

Success = Repository builds, tests pass, no regressions
```

### Cost Analysis

```
Per-Repository Costs (API Migration Task):

CodePlan Approach:
├─ Initial analysis: $0.50
├─ 47 edits @ $0.15 each: $7.05
├─ Oracle runs: $0.00 (local)
├─ Total: $7.55
└─ Success rate: 80%

Direct GPT-4 Approach:
├─ Single large prompt: $2.50
├─ Retry attempts (9 failures): $22.50
├─ Total: $25.00
└─ Success rate: 10%

Effective Cost per Success:
├─ CodePlan: $7.55 / 0.80 = $9.44
└─ Direct: $25.00 / 0.10 = $250.00
```

---

## Implementation Guide

### Core Planning Engine

```python
class CodePlanEngine:
    """
    Implements adaptive plan execution for repository-level changes.
    """

    def __init__(self, repo_path, oracle):
        self.repo = Repository(repo_path)
        self.oracle = oracle  # Build/test/typecheck system
        self.dependency_graph = self._build_dependency_graph()
        self.obligations = []  # Queue of required edits
        self.completed_edits = []  # History of changes

    def execute_plan(self, initial_task):
        """Execute adaptive plan from initial task description."""

        # Initialize with seed obligations
        self.obligations = self._generate_seed_obligations(initial_task)

        while self.obligations:
            # Select next edit
            edit_spec = self._select_next_obligation()

            print(f"Processing: {edit_spec['description']}")

            # Gather context
            context = self._gather_context(edit_spec)

            # Generate edit
            edit = self._generate_edit(edit_spec, context)

            # Apply edit
            self._apply_edit(edit)

            # Run oracle
            oracle_result = self.oracle.run()

            if oracle_result.success:
                print(f"✓ Oracle passed")
                self.completed_edits.append(edit)
            else:
                print(f"✗ Oracle failed: {oracle_result.errors}")
                # Extract new obligations from failures
                new_obligations = self._extract_obligations(oracle_result)
                self.obligations.extend(new_obligations)
                print(f"  Added {len(new_obligations)} new obligations")

            # Update dependency graph with changes
            self._update_dependency_graph(edit)

            # May-impact propagation
            impacted = self._may_impact_propagation(edit)
            if impacted:
                print(f"  May-impact: {len(impacted)} potential changes")
                self.obligations.extend(impacted)

        # Final validation
        final_result = self.oracle.run()
        return final_result.success

    def _gather_context(self, edit_spec):
        """Gather spatial + temporal context for an edit."""

        spatial_context = {
            'target_code': self._load_code(edit_spec['file'], edit_spec['location']),
            'callees': self.dependency_graph.find_callees(edit_spec['function']),
            'callers': self.dependency_graph.find_callers(edit_spec['function']),
            'types': self.dependency_graph.find_types(edit_spec['function']),
            'tests': self.dependency_graph.find_tests(edit_spec['function'])
        }

        temporal_context = {
            'recent_edits': self._get_recent_edits(edit_spec['file']),
            'related_changes': self._get_related_changes(edit_spec)
        }

        return {
            'spatial': spatial_context,
            'temporal': temporal_context
        }

    def _generate_edit(self, edit_spec, context):
        """Use LLM to generate edit given spec and context."""

        prompt = f"""
        Task: {edit_spec['description']}

        Target Code:
        {context['spatial']['target_code']}

        Dependencies:
        - Calls: {context['spatial']['callees']}
        - Called by: {context['spatial']['callers']}
        - Uses types: {context['spatial']['types']}

        Recent Changes:
        {context['temporal']['recent_edits']}

        Generate the edit needed to fulfill this obligation.
        Be consistent with recent changes.
        Maintain all dependencies.
        """

        edit = llm.call(prompt)
        return edit

    def _extract_obligations(self, oracle_result):
        """Extract new edit obligations from oracle failures."""

        obligations = []

        for error in oracle_result.errors:
            if error.type == 'NameError':
                # Missing import or undefined variable
                obligations.append({
                    'file': error.file,
                    'location': error.line,
                    'type': 'fix_name_error',
                    'description': f"Fix undefined name: {error.name}",
                    'priority': 'HIGH'
                })

            elif error.type == 'TypeError':
                # Signature mismatch
                obligations.append({
                    'file': error.file,
                    'location': error.line,
                    'type': 'fix_type_error',
                    'description': f"Fix type error: {error.message}",
                    'priority': 'HIGH'
                })

            elif error.type == 'TestFailure':
                # Behavioral regression
                obligations.append({
                    'file': error.test_file,
                    'location': error.test_name,
                    'type': 'fix_test_failure',
                    'description': f"Fix failing test: {error.test_name}",
                    'priority': 'MEDIUM'
                })

        return obligations

    def _may_impact_propagation(self, edit):
        """Determine what else might need changing."""

        impacted = []

        # Check if edit changes signatures
        if edit.changes_signature:
            # All callers may need updating
            callers = self.dependency_graph.find_callers(edit.function)
            for caller in callers:
                if not self._already_obligated(caller):
                    impacted.append({
                        'file': caller.file,
                        'location': caller.line,
                        'type': 'check_caller',
                        'description': f"Check caller affected by signature change",
                        'priority': 'LOW'  # May not actually need change
                    })

        # Check if edit changes types
        if edit.changes_types:
            # All type users may need updating
            users = self.dependency_graph.find_type_users(edit.type_name)
            for user in users:
                if not self._already_obligated(user):
                    impacted.append({
                        'file': user.file,
                        'location': user.line,
                        'type': 'check_type_usage',
                        'description': f"Check type usage affected by type change",
                        'priority': 'MEDIUM'
                    })

        return impacted

    def _select_next_obligation(self):
        """Select next edit to perform (priority queue)."""

        # Sort by priority: HIGH > MEDIUM > LOW
        priority_order = {'HIGH': 0, 'MEDIUM': 1, 'LOW': 2}
        self.obligations.sort(
            key=lambda x: priority_order.get(x['priority'], 3)
        )

        return self.obligations.pop(0)
```

### Dependency Graph Builder

```python
class DependencyGraphBuilder:
    """
    Build and maintain repository dependency graph.
    """

    def build_graph(self, repo_path):
        """Construct initial dependency graph."""

        graph = {
            'files': {},
            'functions': {},
            'classes': {},
            'types': {},
            'calls': [],
            'imports': [],
            'tests': {}
        }

        for file_path in glob(f"{repo_path}/**/*.py"):
            tree = ast.parse(open(file_path).read())

            # Extract functions
            for node in ast.walk(tree):
                if isinstance(node, ast.FunctionDef):
                    func_key = f"{file_path}::{node.name}"
                    graph['functions'][func_key] = {
                        'file': file_path,
                        'lineno': node.lineno,
                        'calls': self._extract_calls(node),
                        'params': [a.arg for a in node.args.args],
                        'returns': self._extract_return_type(node)
                    }

                # Extract call relationships
                elif isinstance(node, ast.Call):
                    graph['calls'].append({
                        'caller': func_key,
                        'callee': self._resolve_call(node)
                    })

        return graph

    def update_graph(self, edit, graph):
        """Incrementally update graph after an edit."""

        if edit.type == 'function_change':
            # Re-analyze affected function
            func_key = f"{edit.file}::{edit.function}"
            new_info = self._analyze_function(edit.file, edit.function)
            graph['functions'][func_key] = new_info

        elif edit.type == 'import_change':
            # Update import relationships
            self._update_imports(edit, graph)

        return graph
```

---

## Integration with K&V Workflow

### CodePlan Principles in Beads

```yaml
# Initial Plan (Seed Obligations)
- id: bd-401
  title: "Migrate API calls in core module"
  phase: execution
  deliverable: "Updated core/api.py with new API calls"
  may_create:
    - "Import update beads"
    - "Test update beads"
    - "Documentation update beads"

# Conditional Beads (Generated by Oracle)
- id: bd-402
  title: "Fix import errors surfaced by build"
  phase: execution
  created_by: "Oracle feedback from bd-401"
  condition: "Build fails with import errors"
  deliverable: "Fixed imports"

- id: bd-403
  title: "Update tests broken by API migration"
  phase: verification
  created_by: "Test failures from bd-401"
  condition: "Tests fail after API changes"
  deliverable: "Fixed tests"
```

### Adaptive Planning in K&V

```python
# In calibration after phase:

if oracle_result.has_failures:
    # Extract new beads from failures
    new_beads = extract_beads_from_oracle(oracle_result)

    # Add to plan
    for bead in new_beads:
        plan.add_bead(bead, priority='HIGH')

    # Update dependencies
    plan.update_dependencies()

    # Message coordinator
    send_message(
        subject="[ORACLE FAILURES] New beads created",
        body=f"Oracle found {len(new_beads)} issues. Created beads: {new_beads}"
    )
```

---

## Key Takeaways

1. **Plans must adapt** — Can't know all required edits upfront; discover through execution
2. **Oracle feedback is critical** — Build/test failures guide plan expansion
3. **Spatial + temporal context both matter** — Need related code AND change history
4. **May-impact propagation reduces surprises** — Predict cascade effects proactively
5. **Incremental dependency tracking** — Update graph as changes happen
6. **Priority matters** — Fix blocking errors before speculative improvements
7. **Conditional beads are natural** — "If build fails with X, create bead Y"

---

## See Also

- `011-agentless.md` — Single-patch validation approach
- `013-specrover.md` — Specification-driven repair
- `015-coeditor.md` — Diff-based editing with temporal context
- `038-adapt.md` — Adaptive decomposition patterns
- `014-codeplan.md` — Multi-step edit planning
