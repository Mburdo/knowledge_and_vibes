---
title: DeepCode - Open Agentic Coding
tags: [synthesis, blueprint, stateful-memory, paper-to-code, context-management, generation]
protocols: [P3, P8]
---

# DeepCode: Open Agentic Coding

**Paper:** DeepCode: Open Agentic Coding
**URL:** https://arxiv.org/abs/2512.07921
**Date:** December 8, 2025

---

## Summary

Research introducing **DeepCode**, a fully autonomous framework for high-fidelity document-to-codebase synthesis. Designed to turn scientific papers into working code repositories.

**Key innovation:** Manages information flow to overcome LLM context bottlenecks using blueprint distillation, stateful code memory, and closed-loop error correction.

---

## Core Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        DEEPCODE PIPELINE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. BLUEPRINT DISTILLATION                                       │
│  ─────────────────────────                                       │
│  Document → Extract architecture → Generate blueprint            │
│  (Condenses complex papers into actionable structure)            │
│                                                                  │
│  2. STATEFUL CODE MEMORY                                         │
│  ────────────────────────                                        │
│  Maintains context across files and iterations                   │
│  (Prevents context window overflow)                              │
│                                                                  │
│  3. RETRIEVAL-AUGMENTED GENERATION                               │
│  ─────────────────────────────────                               │
│  Pulls relevant code patterns and documentation                  │
│  (Grounds generation in existing knowledge)                      │
│                                                                  │
│  4. CLOSED-LOOP ERROR CORRECTION                                 │
│  ────────────────────────────────                                │
│  Execute → Detect errors → Correct → Re-execute                  │
│  (Iterative refinement with execution feedback)                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Techniques

### Blueprint Distillation

Transform complex specifications into structured blueprints:

| Input | Output |
|-------|--------|
| Research paper | Architecture diagram |
| API documentation | Interface specifications |
| Requirements doc | Component graph |

```markdown
## Blueprint Example

### Components
1. DataLoader
   - Inputs: file_path, batch_size
   - Outputs: Iterator[Batch]
   - Dependencies: numpy, torch

2. Model
   - Architecture: Transformer
   - Layers: 12
   - Dependencies: DataLoader

3. Trainer
   - Dependencies: Model, DataLoader
   - Outputs: trained_model.pt
```

### Stateful Code Memory

Maintains knowledge across the generation session:

```python
class CodeMemory:
    def __init__(self):
        self.files_generated = {}      # filename -> content
        self.imports_used = set()       # all imports across files
        self.interfaces_defined = {}    # class/function signatures
        self.errors_encountered = []    # for learning
        self.corrections_applied = []   # what worked

    def add_file(self, filename, content):
        self.files_generated[filename] = content
        self.imports_used.update(extract_imports(content))
        self.interfaces_defined.update(extract_signatures(content))

    def get_context_for(self, new_file):
        # Return relevant context without exceeding token limit
        return select_relevant(self.files_generated, new_file)
```

### Closed-Loop Error Correction

Iterative refinement with execution feedback:

```
Loop (max 3 iterations per file):
├── Generate code
├── Execute/test
├── If error:
│   ├── Extract error message
│   ├── Identify error location
│   ├── Generate targeted fix
│   └── Re-execute
└── If success: Move to next file
```

---

## Performance Results

From December 2025 evaluations:

| Benchmark | DeepCode | Previous SOTA | Improvement |
|-----------|----------|---------------|-------------|
| ResearchCodeBench | 52% | 38% | +14% |
| Paper→Code (novel) | 41% | 28% | +13% |
| Multi-file synthesis | 67% | 54% | +13% |

---

## Practical Implications

### For Knowledge & Vibes

DeepCode's techniques align with our architecture:

| DeepCode Technique | K&V Equivalent |
|--------------------|----------------|
| Blueprint distillation | Phase planning + bead decomposition |
| Stateful code memory | CASS + cm context |
| Closed-loop correction | TDD + ADaPT pattern |
| Context management | Minimal viable context principle |

### Blueprint → Bead Mapping

```
DeepCode Blueprint:
├── Component A (DataLoader)
├── Component B (Model)
└── Component C (Trainer)

K&V Beads:
├── phase-1.1: DataLoader implementation
├── phase-1.2: Model architecture
├── phase-1.3: Trainer logic
├── phase-1.4: DataLoader tests
├── phase-1.5: Model tests
└── phase-1.6: Trainer tests
```

---

## Key Insights

1. **Context management is critical** — Stateful memory prevents token overflow
2. **Distillation before generation** — Compress specs into actionable blueprints
3. **Execution feedback essential** — Closed-loop correction catches errors
4. **Multi-file coherence** — Track interfaces and imports across files
5. **3-iteration limit** — Matches DDI research on repair decay

---

## Limitations

- Requires execution environment (Docker/sandbox)
- Blueprint quality depends on document clarity
- Novel algorithms still challenging (41% success)
- Memory management overhead for large projects

---

## See Also

- `062-rag-repository-code.md` — RAG for code context
- `060-debugging-decay-index.md` — Why 3-iteration limit
- `038-adapt.md` — Adaptive decomposition
