---
title: AutoCodeRover - Structure-Aware Navigation
tags: [navigation, structure-aware, localization, sbfl, retrieval, ast]
protocols: [P2, P8]
---

# AutoCodeRover: Autonomous Program Improvement

**Paper:** AutoCodeRover: Autonomous Program Improvement
**URL:** https://arxiv.org/html/2404.05427v1
**Date:** April 2024
**Venue:** arXiv preprint

---

## Summary

Pioneering research framing LLM-based software engineering as a **program structure problem** rather than a "bag of files" problem. AutoCodeRover combines structure-aware code navigation, iterative context retrieval, and optional spectrum-based fault localization (SBFL) to achieve state-of-the-art results on SWE-bench.

**Key insight:** Tooling beats prompting. Give models structure-aware navigation APIs instead of dumping entire repos into context.

---

## The Core Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     AUTOCODEROVER SYSTEM                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │           CONTEXT RETRIEVAL ENGINE                       │   │
│  │                                                           │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │   │
│  │  │  Structure  │  │  Semantic   │  │    SBFL     │      │   │
│  │  │  Search API │  │   Search    │  │  (Optional) │      │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘      │   │
│  │         │                │                │              │   │
│  │         └────────────────┴────────────────┘              │   │
│  │                         ▼                                │   │
│  │              Iterative Hypothesis Refinement             │   │
│  │                                                           │   │
│  └───────────────────────────┬──────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              PATCH GENERATION                             │   │
│  │                                                           │   │
│  │  Input: Localized code context + issue                   │   │
│  │  Output: Targeted patch                                  │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Structure-Aware Navigation API

### The Key Innovation

Traditional approach:
```python
# Dump entire repo into context (fails at scale)
context = read_entire_repo(repo_path)
```

AutoCodeRover approach:
```python
# Navigate by program structure
context = ""
context += search_classes("authentication")
context += search_methods("verify_password")
context += get_class_hierarchy("User")
```

### Navigation Primitives

| API Call | Purpose | Example |
|----------|---------|---------|
| `search_classes(query)` | Find classes by name/description | `search_classes("database connection")` |
| `search_methods(query)` | Find methods/functions | `search_methods("validate input")` |
| `search_code(pattern)` | Pattern-based search | `search_code("SELECT .* FROM users")` |
| `get_class_hierarchy(cls)` | Understand inheritance | `get_class_hierarchy("BaseModel")` |
| `get_method_callers(method)` | Find usage sites | `get_method_callers("authenticate")` |
| `get_method_callees(method)` | Find dependencies | `get_method_callees("login")` |

### Visual Representation of Structure Navigation

```
Repository (100,000 LOC)
    │
    ├─ search_classes("auth")
    │      │
    │      ├─── AuthManager (src/auth/manager.py)
    │      ├─── AuthenticationError (src/auth/errors.py)
    │      └─── AuthMiddleware (src/middleware/auth.py)
    │
    ├─ search_methods("verify")
    │      │
    │      ├─── verify_password() (src/auth/utils.py)
    │      ├─── verify_token() (src/auth/jwt.py)
    │      └─── verify_permissions() (src/auth/rbac.py)
    │
    └─ get_method_callers("verify_password")
           │
           ├─── AuthManager.login() (called at line 42)
           ├─── reset_password_handler() (called at line 156)
           └─── admin_override() (called at line 289)

Total context loaded: ~500 LOC (0.5% of repository)
```

---

## Iterative Context Retrieval

### The Search → Read → Refine Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                 ITERATIVE HYPOTHESIS REFINEMENT                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Iteration 1: Initial Hypothesis                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Hypothesis: "Bug is in authentication logic"               │ │
│  │ Action: search_classes("authentication")                   │ │
│  │ Result: Found 3 classes                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  Iteration 2: Narrow Down                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Hypothesis: "Bug is in password verification"              │ │
│  │ Action: search_methods("verify_password")                  │ │
│  │ Result: Found verify_password() in auth/utils.py           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  Iteration 3: Understand Context                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Hypothesis: "Need to see how it's called"                  │ │
│  │ Action: get_method_callers("verify_password")              │ │
│  │ Result: Called from AuthManager.login()                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  Iteration 4: Root Cause                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Hypothesis: "Missing null check before verification"       │ │
│  │ Action: Read full method implementation                    │ │
│  │ Result: CONFIRMED - no null check for password param       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Spectrum-Based Fault Localization (SBFL)

### When Tests Are Available

AutoCodeRover can leverage existing test failures to guide localization:

```
┌─────────────────────────────────────────────────────────────────┐
│                  SBFL WORKFLOW                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Run Test Suite                                         │
│  ├── Passing tests: 487                                         │
│  └── Failing tests: 3                                           │
│                                                                  │
│  Step 2: Compute Coverage                                       │
│  ├── Lines executed by failing tests: L1, L2, ..., Ln          │
│  └── Lines executed by passing tests: M1, M2, ..., Mm          │
│                                                                  │
│  Step 3: Calculate Suspiciousness Scores                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Tarantula Formula:                                         │ │
│  │                                                             │ │
│  │         %failed(s)                                          │ │
│  │  ────────────────────────────────────────────────────────   │ │
│  │  %failed(s) + %passed(s)                                    │ │
│  │                                                             │ │
│  │  Where:                                                     │ │
│  │    %failed(s) = failed tests covering s / total failed      │ │
│  │    %passed(s) = passed tests covering s / total passed      │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Step 4: Rank Lines by Suspiciousness                           │
│  ├── auth/utils.py:42   (score: 0.95) ← High suspicion         │
│  ├── auth/utils.py:38   (score: 0.87)                          │
│  ├── auth/manager.py:15 (score: 0.12) ← Low suspicion          │
│  └── ...                                                        │
│                                                                  │
│  Step 5: Guide Context Retrieval                                │
│  └── Focus on high-scoring lines first                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### SBFL Impact on Performance

| Configuration | Pass@1 | Avg Iterations |
|---------------|--------|----------------|
| No SBFL | 16.2% | 4.7 |
| With SBFL | **19.8%** | **3.2** |

**Key benefit:** SBFL reduces search space by 40-60% when tests are available.

---

## Performance Results

### SWE-bench Lite Results

```
Success Rate
  │
25├────────────────────●  AutoCodeRover + SBFL (19.8%)
  │
20├───────────────●  AutoCodeRover (16.2%)
  │
15├────────
  │
10├──────●  GPT-4 Baseline (8.2%)
  │
 5├───
  │
 0└──────────────────────────────────────
   No      Basic    Structure   +SBFL
  Tools   Prompt   Navigation
```

### Cost Analysis

| Metric | Value | Notes |
|--------|-------|-------|
| Average cost per issue | **<$1.00** | GPT-4 API costs (2024) |
| Average iterations | 3.2 | With SBFL |
| Average context size | 1,200 lines | 1-2% of typical repo |
| Success rate | 19.8% | SWE-bench Lite |

---

## The Structure vs. Semantic Tradeoff

### Comparison of Search Approaches

| Approach | Precision | Recall | Cost | When to Use |
|----------|-----------|--------|------|-------------|
| **Keyword search** | Low | High | Low | Exploratory phase |
| **Semantic search** | Medium | Medium | Medium | Concept-based queries |
| **Structure search** | High | Medium | Low | Known target types |
| **Hybrid** | **High** | **High** | **Medium** | **Best practice** |

### Example: Finding Authentication Bug

```python
# Approach 1: Pure keyword (noisy)
results = grep("password", repo)
# Returns: 347 files (too many!)

# Approach 2: Pure semantic (expensive)
results = semantic_search("authentication issues", repo)
# Returns: Relevant but slow, high API cost

# Approach 3: Structure-aware (precise)
results = search_methods("authenticate") + search_classes("Auth")
# Returns: 12 symbols (manageable!)

# Approach 4: Hybrid (AutoCodeRover's approach)
candidates = search_classes("Auth")  # Structure
filtered = semantic_filter(candidates, "password verification")  # Semantic
# Returns: 3 highly relevant symbols
```

---

## Implementation Guide

### Building a Structure-Aware Search API

```python
class StructureAwareSearch:
    """
    Implements AutoCodeRover's structure-aware navigation.
    """

    def __init__(self, repo_path):
        self.repo_path = repo_path
        self.index = self._build_index()

    def _build_index(self):
        """Build AST-based index of repository."""
        index = {
            'classes': {},
            'methods': {},
            'functions': {},
            'imports': {},
            'callgraph': {}
        }

        for file_path in glob(f"{self.repo_path}/**/*.py"):
            tree = ast.parse(open(file_path).read())

            for node in ast.walk(tree):
                if isinstance(node, ast.ClassDef):
                    index['classes'][node.name] = {
                        'file': file_path,
                        'lineno': node.lineno,
                        'bases': [b.id for b in node.bases],
                        'methods': [m.name for m in node.body
                                  if isinstance(m, ast.FunctionDef)]
                    }

                elif isinstance(node, ast.FunctionDef):
                    key = f"{file_path}::{node.name}"
                    index['functions'][key] = {
                        'file': file_path,
                        'lineno': node.lineno,
                        'args': [a.arg for a in node.args.args],
                        'calls': self._extract_calls(node)
                    }

        return index

    def search_classes(self, query):
        """Find classes matching query."""
        results = []

        for name, info in self.index['classes'].items():
            # Exact match
            if query.lower() in name.lower():
                results.append({
                    'type': 'class',
                    'name': name,
                    **info,
                    'score': 1.0
                })

            # Semantic match (using embeddings)
            else:
                doc = self._get_class_docstring(info['file'], name)
                score = semantic_similarity(query, doc)
                if score > 0.7:
                    results.append({
                        'type': 'class',
                        'name': name,
                        **info,
                        'score': score
                    })

        return sorted(results, key=lambda x: x['score'], reverse=True)

    def search_methods(self, query):
        """Find methods/functions matching query."""
        results = []

        for key, info in self.index['functions'].items():
            func_name = key.split('::')[1]

            if query.lower() in func_name.lower():
                results.append({
                    'type': 'function',
                    'name': func_name,
                    **info,
                    'score': 1.0
                })

        return sorted(results, key=lambda x: x['score'], reverse=True)

    def get_class_hierarchy(self, class_name):
        """Get inheritance hierarchy for a class."""
        if class_name not in self.index['classes']:
            return None

        cls_info = self.index['classes'][class_name]
        hierarchy = {
            'name': class_name,
            'bases': [],
            'derived': []
        }

        # Find base classes
        for base in cls_info['bases']:
            if base in self.index['classes']:
                hierarchy['bases'].append(
                    self.get_class_hierarchy(base)
                )

        # Find derived classes
        for name, info in self.index['classes'].items():
            if class_name in info['bases']:
                hierarchy['derived'].append(name)

        return hierarchy

    def get_method_callers(self, method_name):
        """Find all locations calling a method."""
        callers = []

        for key, info in self.index['functions'].items():
            if method_name in info['calls']:
                callers.append({
                    'caller': key.split('::')[1],
                    'file': info['file'],
                    'lineno': info['lineno']
                })

        return callers

    def _extract_calls(self, node):
        """Extract all function calls in an AST node."""
        calls = []
        for child in ast.walk(node):
            if isinstance(child, ast.Call):
                if isinstance(child.func, ast.Name):
                    calls.append(child.func.id)
                elif isinstance(child.func, ast.Attribute):
                    calls.append(child.func.attr)
        return calls
```

### Iterative Context Retrieval Loop

```python
class ContextRetriever:
    """
    Implements AutoCodeRover's iterative hypothesis refinement.
    """

    def __init__(self, search_api, issue_text, max_iterations=5):
        self.search = search_api
        self.issue = issue_text
        self.max_iterations = max_iterations
        self.context = []
        self.hypotheses = []

    def retrieve_context(self):
        """Iteratively refine hypothesis and gather context."""

        for iteration in range(self.max_iterations):
            # Generate hypothesis
            hypothesis = self._generate_hypothesis()
            self.hypotheses.append(hypothesis)

            print(f"Iteration {iteration + 1}: {hypothesis['description']}")

            # Execute search based on hypothesis
            new_context = self._execute_hypothesis(hypothesis)

            # Add to context if relevant
            if self._is_relevant(new_context):
                self.context.extend(new_context)

                # Check if we have enough context
                if self._is_sufficient():
                    print(f"Sufficient context found after {iteration + 1} iterations")
                    break
            else:
                print("Search returned irrelevant results, refining...")

        return self.context

    def _generate_hypothesis(self):
        """Generate next hypothesis based on context so far."""
        prompt = f"""
        Issue: {self.issue}

        Context gathered so far:
        {self._format_context()}

        Previous hypotheses:
        {self._format_hypotheses()}

        What should we investigate next? Generate a hypothesis about
        where the bug might be and what search to perform.

        Return JSON with:
        - description: What we're looking for
        - search_type: 'class', 'method', 'code_pattern'
        - query: Search query
        """

        response = llm.call(prompt)
        return response

    def _execute_hypothesis(self, hypothesis):
        """Execute search based on hypothesis."""
        if hypothesis['search_type'] == 'class':
            return self.search.search_classes(hypothesis['query'])

        elif hypothesis['search_type'] == 'method':
            return self.search.search_methods(hypothesis['query'])

        elif hypothesis['search_type'] == 'code_pattern':
            return self.search.search_code(hypothesis['query'])

        elif hypothesis['search_type'] == 'callers':
            return self.search.get_method_callers(hypothesis['query'])

        return []

    def _is_relevant(self, results):
        """Check if search results are relevant to issue."""
        if not results:
            return False

        # Use LLM to judge relevance
        prompt = f"""
        Issue: {self.issue}

        Search results:
        {json.dumps(results, indent=2)}

        Are these results relevant to the issue? (yes/no)
        """

        response = llm.call(prompt)
        return response.strip().lower() == 'yes'

    def _is_sufficient(self):
        """Check if we have enough context to proceed."""
        prompt = f"""
        Issue: {self.issue}

        Context gathered:
        {self._format_context()}

        Do we have sufficient context to localize and fix this issue? (yes/no)

        Consider:
        - Do we understand the bug location?
        - Do we have the relevant code?
        - Do we understand how to fix it?
        """

        response = llm.call(prompt)
        return response.strip().lower() == 'yes'
```

---

## Integration with K&V Workflow

### Mapping to Beads

```yaml
# Investigation Phase: Structure-Aware Search
- id: bd-201
  title: "Locate bug using structure search"
  phase: investigation
  tools:
    - search_classes
    - search_methods
    - get_class_hierarchy
  deliverable: "List of suspicious symbols with context"

# Investigation Phase: Iterative Refinement
- id: bd-202
  title: "Refine hypothesis through iteration"
  phase: investigation
  depends: [bd-201]
  deliverable: "Root cause hypothesis with evidence"

# Planning Phase: Patch Design
- id: bd-203
  title: "Design patch based on localized context"
  phase: planning
  depends: [bd-202]
  deliverable: "Patch strategy"

# Execution Phase: Generate Patch
- id: bd-204
  title: "Generate and validate patch"
  phase: execution
  depends: [bd-203]
  deliverable: "Tested patch"
```

---

## Key Takeaways

1. **Structure-aware navigation beats context dumping** — AST-based search is more precise than grep
2. **Iterative hypothesis refinement works** — Search → read → refine loop finds bugs efficiently
3. **SBFL adds significant value** — When tests exist, use them to guide search
4. **Context size matters** — 1-2% of repo is often sufficient with right tools
5. **Tool design is critical** — The API shapes what agents can accomplish
6. **Hybrid search is best** — Combine structural and semantic approaches
7. **Track your hypothesis** — Explicit hypothesis makes debugging searchable

---

## See Also

- `011-agentless.md` — Hierarchical localization approach
- `013-specrover.md` — Specification inference on top of retrieval
- `017-context-retrieval-repo-editing.md` — Context sufficiency research
- `065-confucius-code-agent.md` — Hierarchical navigation at scale
- `057-anthropic-context-engineering.md` — Minimal context principle
