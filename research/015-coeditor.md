---
title: Coeditor - Leveraging Repo-level Diffs
tags: [editing, diffs, temporal-context, repo-level, editing-prediction, attention]
protocols: [P5, P8]
---

# Coeditor: Leveraging Repo-level Diffs for Code Auto-Editing

**Paper:** Coeditor: Leveraging Repo-level Diffs for Code Auto-editing
**URL:** https://arxiv.org/html/2305.18584v2
**Date:** May 2023 (arXiv); ICLR 2024
**Venue:** ICLR 2024

---

## Summary

Pioneering research shifting focus from **code generation** to **code editing**. Coeditor trains models to predict follow-up edits by conditioning on recent repository-level diffs, recognizing that real development is iterative modification, not green-field writing.

**Key insight:** "What changed elsewhere" is critical context for predicting what to change next. Temporal context (diffs) matters as much as spatial context (surrounding code).

---

## The Code Editing Reality

### Development is Editing, Not Writing

```
Real Software Development:

New Code (Generation):     ████  15%
Editing Existing Code:     ████████████████████████████████████  85%

Traditional LLM Focus:     Code generation
Actual Developer Need:     Edit continuation and propagation
```

### Example: Edit Propagation

```python
# Developer makes change in file1.py:
- def connect(host, port):
+ def connect(host, port, timeout=30):

# Coeditor predicts needed changes:

# file2.py (caller site)
- conn = connect("localhost", 5432)
+ conn = connect("localhost", 5432, timeout=60)

# file3.py (documentation)
- Connect to database using host and port.
+ Connect to database using host, port, and optional timeout.

# test_connection.py (test coverage)
+ def test_connect_with_timeout():
+     conn = connect("localhost", 5432, timeout=10)
+     assert conn.timeout == 10
```

---

## The Coeditor Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      COEDITOR SYSTEM                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  INPUT CONTEXT                                            │   │
│  │                                                           │   │
│  │  Current Code Region:                                    │   │
│  │  ├─ Target file/function to edit                         │   │
│  │  ├─ Surrounding code context                             │   │
│  │  └─ Static analysis (types, dependencies)               │   │
│  │                                                           │   │
│  │  Recent Diff History:                                    │   │
│  │  ├─ What changed in last N commits                       │   │
│  │  ├─ Changes in related files                             │   │
│  │  └─ Pattern of edits (refactor, feature, bugfix)        │   │
│  │                                                           │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  BLOCK-SPARSE ATTENTION                                   │   │
│  │                                                           │   │
│  │  Problem: Can't attend to all diffs (quadratic cost)     │   │
│  │  Solution: Attend to relevant diff blocks only           │   │
│  │                                                           │   │
│  │  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐        │   │
│  │  │ A │──│ B │  │ C │──│ D │  │ E │──│ F │  │ G │        │   │
│  │  └───┘  └───┘  └───┘  └───┘  └───┘  └───┘  └───┘        │   │
│  │    ↓      ↓      ↓      ↓      ↓      ↓      ↓          │   │
│  │  Attend to: A, B, D, F (relevant blocks only)           │   │
│  │                                                           │   │
│  └────────────────────────┬─────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  EDIT PREDICTION                                          │   │
│  │                                                           │   │
│  │  Output: Line-diff format edit                           │   │
│  │  ├─ Lines to delete (marked with -)                      │   │
│  │  ├─ Lines to add (marked with +)                         │   │
│  │  └─ Context lines (unmarked)                             │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Diff-Based Context Representation

### Why Diffs Matter More Than Files

| Context Type | Information | Example Use |
|--------------|-------------|-------------|
| **File contents** | What the code is | Understanding current state |
| **Recent diffs** | What changed and why | Predicting continuation |
| **Both together** | Current state + change pattern | Accurate edit prediction |

### Diff Format Example

```diff
Recent changes in repository:

=== commit 1 (5 minutes ago): "Add timeout parameter" ===
diff --git a/database/connection.py b/database/connection.py
@@ -15,7 +15,7 @@
-def connect(host, port):
+def connect(host, port, timeout=30):
     """Establish database connection."""
-    conn = socket.create_connection((host, port))
+    conn = socket.create_connection((host, port), timeout=timeout)
     return Connection(conn)

=== commit 2 (3 minutes ago): "Update primary call site" ===
diff --git a/app/database.py b/app/database.py
@@ -42,7 +42,7 @@
-    db_conn = connect(DB_HOST, DB_PORT)
+    db_conn = connect(DB_HOST, DB_PORT, timeout=60)

=== Current edit location: app/cache.py ===
# Need to update this call site too
cache_conn = connect(CACHE_HOST, CACHE_PORT)  # ← Predict edit here
```

**Model input:** Recent diffs + current code
**Model output:**
```diff
-cache_conn = connect(CACHE_HOST, CACHE_PORT)
+cache_conn = connect(CACHE_HOST, CACHE_PORT, timeout=30)
```

---

## Block-Sparse Attention

### The Attention Problem

```
Traditional Attention:
├─ Every token attends to every other token
├─ Cost: O(n²) where n = sequence length
└─ Doesn't scale to large diffs + full files

Full Repository Context:
├─ Recent diffs: ~5,000 tokens
├─ Current file: ~2,000 tokens
├─ Related files: ~3,000 tokens
├─ Total: ~10,000 tokens
└─ Traditional attention: 100M operations (too slow!)
```

### Block-Sparse Solution

```
┌─────────────────────────────────────────────────────────────────┐
│              BLOCK-SPARSE ATTENTION PATTERN                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Sequence divided into blocks:                                  │
│                                                                  │
│  [Diff 1] [Diff 2] [Diff 3] [File A] [File B] [Target]         │
│     ▼       ▼       ▼         ▼        ▼        ▼              │
│   Block1  Block2  Block3   Block4   Block5   Block6            │
│                                                                  │
│  Attention pattern (only attend to relevant blocks):            │
│                                                                  │
│  Block6 (target) attends to:                                    │
│  ├─ Block6 (self) ✓                                             │
│  ├─ Block1 (related diff) ✓                                     │
│  ├─ Block2 (unrelated diff) ✗                                   │
│  ├─ Block3 (related diff) ✓                                     │
│  ├─ Block4 (related file) ✓                                     │
│  └─ Block5 (unrelated file) ✗                                   │
│                                                                  │
│  Complexity reduction:                                           │
│  ├─ Full attention: O(n²) = 36 operations                       │
│  └─ Block-sparse: O(b×k) = 12 operations (3× faster)            │
│                                                                  │
│  Where:                                                          │
│  - n = 6 blocks                                                  │
│  - b = 6 blocks                                                  │
│  - k = 2 relevant blocks per target                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Relevance Detection

```python
def select_relevant_blocks(target_block, all_blocks):
    """Select which diff blocks are relevant to target."""

    relevant = []

    for block in all_blocks:
        # Check syntactic relevance
        if shares_symbols(target_block, block):
            relevant.append(block)

        # Check semantic relevance
        elif semantic_similarity(target_block, block) > 0.7:
            relevant.append(block)

        # Check file relationship
        elif are_files_related(target_block.file, block.file):
            relevant.append(block)

    return relevant
```

---

## The PyCommits Dataset

### Novel Contribution

| Dataset | Task | Size | Focus |
|---------|------|------|-------|
| **PyCommits** | Edit prediction | 1,650 repos | Multi-round edits |
| HumanEval | Code generation | 164 problems | Single-shot |
| MBPP | Code generation | 427 problems | Single-shot |
| CoNaLa | Code generation | 2,879 pairs | Single-shot |

### Data Collection

```
┌─────────────────────────────────────────────────────────────────┐
│                  PYCOMMITS CONSTRUCTION                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Repository Selection                                   │
│  ├─ Filter for high-quality Python repos                        │
│  ├─ Require test coverage > 70%                                 │
│  └─ Require commit history > 500 commits                        │
│                                                                  │
│  Step 2: Commit Extraction                                      │
│  ├─ Extract multi-file commits (2-10 files)                     │
│  ├─ Filter trivial changes (whitespace, comments)               │
│  └─ Ensure commits are self-contained                           │
│                                                                  │
│  Step 3: Training Example Construction                          │
│                                                                  │
│  For each commit with changes to files [A, B, C]:               │
│                                                                  │
│  Example 1:                                                      │
│  ├─ Context: Changes to file A                                  │
│  ├─ Target: Predict changes to file B                           │
│  └─ Label: Actual changes made to file B                        │
│                                                                  │
│  Example 2:                                                      │
│  ├─ Context: Changes to files A, B                              │
│  ├─ Target: Predict changes to file C                           │
│  └─ Label: Actual changes made to file C                        │
│                                                                  │
│  Result: ~450K training examples                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Performance Results

### Edit Prediction Accuracy

```
Benchmark: Single Edit Prediction (Exact Match)

┌────────────────────┬──────────┬─────────────┬──────────┐
│       Model        │   Size   │  Exact Match│ Partial  │
├────────────────────┼──────────┼─────────────┼──────────┤
│ Coeditor           │   220M   │    60.2%    │  78.5%   │
│ CodeT5             │   220M   │    41.3%    │  65.2%   │
│ CodeGen            │   350M   │    38.7%    │  62.1%   │
│ GPT-3.5 (few-shot) │    -     │    45.1%    │  68.4%   │
└────────────────────┴──────────┴─────────────┴──────────┘

Exact Match: Generated edit matches ground truth exactly
Partial: Generated edit is semantically correct but formatted differently
```

### Multi-Round Editing

```
Benchmark: Multi-round Edit Completion

Scenario: After user makes change, predict N follow-up edits

┌────────────────┬──────────────┬──────────────┬──────────┐
│     Metric     │   Coeditor   │   CodeT5     │  GPT-3.5 │
├────────────────┼──────────────┼──────────────┼──────────┤
│ Lines automated│    46.7%     │    32.1%     │  38.2%   │
│ Keystrokes saved│   28.6%     │    19.4%     │  23.5%   │
│ Precision      │    73.2%     │    68.5%     │  71.8%   │
│ Recall         │    64.1%     │    46.9%     │  53.2%   │
└────────────────┴──────────────┴──────────────┴──────────┘

Lines automated: % of total changed lines correctly predicted
Keystrokes saved: Reduction in developer typing
```

---

## Implementation Guide

### Diff-Based Context Builder

```python
class DiffContextBuilder:
    """
    Build editing context from recent diffs and current code.
    """

    def __init__(self, repo_path, max_diffs=5):
        self.repo = Repository(repo_path)
        self.max_diffs = max_diffs

    def build_context(self, target_file, target_location):
        """Construct context for predicting edit at target location."""

        context = {
            'current_code': self._get_current_code(target_file, target_location),
            'recent_diffs': self._get_recent_diffs(target_file),
            'related_changes': self._get_related_changes(target_file),
            'static_context': self._get_static_context(target_file)
        }

        return context

    def _get_recent_diffs(self, target_file):
        """Get recent commit diffs, prioritize related files."""

        commits = self.repo.get_recent_commits(limit=20)
        diffs = []

        for commit in commits:
            for file_path, diff in commit.get_diffs().items():
                # Include diffs from target file
                if file_path == target_file:
                    diffs.append({
                        'file': file_path,
                        'diff': diff,
                        'message': commit.message,
                        'timestamp': commit.timestamp,
                        'relevance': 1.0  # High relevance
                    })

                # Include diffs from related files
                elif self._are_files_related(target_file, file_path):
                    diffs.append({
                        'file': file_path,
                        'diff': diff,
                        'message': commit.message,
                        'timestamp': commit.timestamp,
                        'relevance': 0.7  # Medium relevance
                    })

            if len(diffs) >= self.max_diffs:
                break

        # Sort by relevance × recency
        diffs.sort(key=lambda d: d['relevance'] * (1.0 / (time.time() - d['timestamp'])),
                   reverse=True)

        return diffs[:self.max_diffs]

    def _get_related_changes(self, target_file):
        """Find changes in files that depend on or are depended by target."""

        dependency_graph = self._build_dependency_graph()

        related_files = set()
        related_files.update(dependency_graph.find_dependencies(target_file))
        related_files.update(dependency_graph.find_dependents(target_file))

        changes = []
        for file in related_files:
            recent_diff = self._get_most_recent_diff(file)
            if recent_diff:
                changes.append(recent_diff)

        return changes

    def _are_files_related(self, file1, file2):
        """Check if two files are related (imports, tests, etc)."""

        # Same directory
        if os.path.dirname(file1) == os.path.dirname(file2):
            return True

        # Test relationship
        if self._is_test_file(file1) and self._tests_file(file1, file2):
            return True

        # Import relationship
        if self._has_import_relationship(file1, file2):
            return True

        return False
```

### Block-Sparse Attention Implementation

```python
class BlockSparseAttention:
    """
    Efficient attention over diff blocks.
    """

    def __init__(self, block_size=128):
        self.block_size = block_size

    def compute_attention(self, target_tokens, context_blocks):
        """Compute attention only over relevant blocks."""

        # Divide target into blocks
        target_blocks = self._create_blocks(target_tokens, self.block_size)

        # Select relevant context blocks for each target block
        relevant_blocks = {}
        for i, target_block in enumerate(target_blocks):
            relevant_blocks[i] = self._select_relevant_blocks(
                target_block,
                context_blocks
            )

        # Compute attention only between relevant pairs
        attention_output = []
        for i, target_block in enumerate(target_blocks):
            # Self-attention within target block
            block_output = self._attend(target_block, target_block)

            # Cross-attention to relevant context blocks
            for context_block in relevant_blocks[i]:
                context_output = self._attend(target_block, context_block)
                block_output += context_output

            attention_output.append(block_output)

        return attention_output

    def _select_relevant_blocks(self, target_block, context_blocks):
        """Select context blocks relevant to target."""

        relevant = []
        target_symbols = self._extract_symbols(target_block)

        for block in context_blocks:
            block_symbols = self._extract_symbols(block)

            # Symbol overlap
            overlap = len(target_symbols & block_symbols)
            if overlap > 0:
                relevant.append(block)

            # Semantic similarity (using embeddings)
            elif self._semantic_similarity(target_block, block) > 0.6:
                relevant.append(block)

        return relevant

    def _attend(self, query_block, key_block):
        """Standard attention computation between two blocks."""

        Q = self.query_projection(query_block)
        K = self.key_projection(key_block)
        V = self.value_projection(key_block)

        attention_scores = (Q @ K.T) / math.sqrt(self.d_k)
        attention_weights = softmax(attention_scores)

        output = attention_weights @ V
        return output
```

### Edit Prediction with Diff Context

```python
class EditPredictor:
    """
    Predict edits using diff-based context.
    """

    def __init__(self, model, context_builder):
        self.model = model
        self.context_builder = context_builder

    def predict_edit(self, target_file, target_location):
        """Predict edit at target location given recent diffs."""

        # Build context
        context = self.context_builder.build_context(
            target_file,
            target_location
        )

        # Format input for model
        model_input = self._format_input(context)

        # Generate edit prediction
        predicted_diff = self.model.generate(model_input)

        # Parse diff format
        edit = self._parse_diff(predicted_diff)

        return edit

    def _format_input(self, context):
        """Format context into model input."""

        input_text = []

        # Add recent diffs
        for diff in context['recent_diffs']:
            input_text.append(f"=== Recent change in {diff['file']} ===")
            input_text.append(f"Message: {diff['message']}")
            input_text.append(diff['diff'])
            input_text.append("")

        # Add current code
        input_text.append("=== Current code to edit ===")
        input_text.append(context['current_code'])

        return "\n".join(input_text)

    def _parse_diff(self, diff_text):
        """Parse model output (diff format) into structured edit."""

        lines = diff_text.split('\n')
        edit = {
            'deletions': [],
            'additions': [],
            'context': []
        }

        for line in lines:
            if line.startswith('-'):
                edit['deletions'].append(line[1:])
            elif line.startswith('+'):
                edit['additions'].append(line[1:])
            else:
                edit['context'].append(line)

        return edit
```

---

## Integration with K&V Workflow

### Diff Tracking in Beads

```yaml
# Track temporal context in bead artifacts
- id: bd-501
  title: "Update database connection signature"
  phase: execution
  deliverable: "Modified connect() function"
  artifacts:
    diff: |
      @@ -15,7 +15,7 @@
      -def connect(host, port):
      +def connect(host, port, timeout=30):

# Next bead uses previous diff as context
- id: bd-502
  title: "Propagate timeout parameter to call sites"
  phase: execution
  depends: [bd-501]
  context_from: [bd-501.diff]  # Use diff from previous bead
  deliverable: "Updated call sites"
```

### Calibration with Diff History

```python
# In phase calibration:

# Collect all diffs from completed phase
phase_diffs = []
for bead in completed_beads:
    if bead.has_artifact('diff'):
        phase_diffs.append(bead.get_artifact('diff'))

# Check consistency
for diff in phase_diffs:
    for other_diff in phase_diffs:
        if conflicts(diff, other_diff):
            report_conflict(diff, other_diff)

# Store in CASS for next phase
cass.store('phase_N_diffs', phase_diffs)
```

---

## Key Takeaways

1. **Development is editing, not generation** — 85% of coding is modifying existing code
2. **Temporal context matters** — Recent diffs predict future edits
3. **Block-sparse attention enables scale** — Can handle large diff histories efficiently
4. **Diff format is natural** — Line-diff representation is intuitive and precise
5. **Multi-round editing is realistic** — Real tasks require multiple related edits
6. **Dataset matters** — PyCommits captures real edit patterns
7. **Small models can compete** — 220M parameters sufficient with right training data

---

## See Also

- `014-codeplan.md` — Temporal context in adaptive planning
- `011-agentless.md` — Diff validation and application
- `012-autocoderover.md` — Spatial context retrieval
- `038-adapt.md` — Multi-stage edit workflows
