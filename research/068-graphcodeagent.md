---
title: GraphCodeAgent - Dual Graph-Guided Code Generation
tags: [graph, retrieval, rag, ast, dependencies, context]
protocols: [P2, P5]
---

# GraphCodeAgent: Dual Graph-Guided Code Generation

**Paper:** GraphCodeAgent: Dual Graph-Guided LLM Agent for Retrieval-Augmented Repo-Level Code Generation
**URL:** https://arxiv.org/abs/2504.10046
**Date:** April 2025 (revised November 2025)

---

## Summary

Research introducing **GraphCodeAgent**, a dual graph-guided LLM agent for retrieval-augmented repository-level code generation. Uses both **code structure graphs** and **dependency graphs** to improve context retrieval.

**Key innovation:** Dual graph guidance provides structural and semantic understanding of codebases.

---

## The Repository Context Problem

### Why Simple RAG Fails for Code

| Approach | Limitation |
|----------|------------|
| Keyword search | Misses semantic relationships |
| Embedding similarity | Ignores code structure |
| File-based retrieval | Misses cross-file dependencies |
| Random sampling | Low relevance |

### What Graphs Capture

```
Code Structure Graph (CSG):
├── Nodes: Functions, classes, modules
├── Edges: Contains, inherits, implements
└── Purpose: Understand code organization

Dependency Graph (DG):
├── Nodes: Functions, classes, variables
├── Edges: Calls, imports, uses
└── Purpose: Understand data/control flow
```

---

## Dual Graph Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    GRAPHCODEAGENT                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────┐    ┌─────────────────────┐             │
│  │ CODE STRUCTURE      │    │ DEPENDENCY          │             │
│  │ GRAPH (CSG)         │    │ GRAPH (DG)          │             │
│  │                     │    │                     │             │
│  │ Module              │    │ Function A          │             │
│  │ ├── Class           │    │ ├── calls B         │             │
│  │ │   ├── Method      │    │ ├── uses var X      │             │
│  │ │   └── Method      │    │ └── imports mod Y   │             │
│  │ └── Function        │    │                     │             │
│  └──────────┬──────────┘    └──────────┬──────────┘             │
│             │                          │                         │
│             └──────────┬───────────────┘                         │
│                        │                                         │
│             ┌──────────┴──────────┐                              │
│             │   GRAPH NAVIGATOR   │                              │
│             │   ├── BFS/DFS       │                              │
│             │   ├── Relevance     │                              │
│             │   └── Pruning       │                              │
│             └──────────┬──────────┘                              │
│                        │                                         │
│             ┌──────────┴──────────┐                              │
│             │  CONTEXT ASSEMBLER  │                              │
│             │  ├── Rank by graph  │                              │
│             │  ├── Fit to window  │                              │
│             │  └── Format prompt  │                              │
│             └──────────┬──────────┘                              │
│                        │                                         │
│             ┌──────────┴──────────┐                              │
│             │     LLM AGENT       │                              │
│             │  (Code Generation)  │                              │
│             └─────────────────────┘                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Graph Construction

### Code Structure Graph

Built using AST parsing:

```python
def build_csg(repo_path: str) -> Graph:
    csg = Graph()

    for file in repo_files(repo_path):
        tree = parse_ast(file)

        # Add module node
        module = csg.add_node(type="module", path=file)

        for node in tree.walk():
            if node.type == "class_definition":
                cls = csg.add_node(type="class", name=node.name)
                csg.add_edge(module, cls, "contains")

                for method in node.methods:
                    meth = csg.add_node(type="method", name=method.name)
                    csg.add_edge(cls, meth, "contains")

            elif node.type == "function_definition":
                func = csg.add_node(type="function", name=node.name)
                csg.add_edge(module, func, "contains")

    return csg
```

### Dependency Graph

Built using call/import analysis:

```python
def build_dg(repo_path: str) -> Graph:
    dg = Graph()

    for file in repo_files(repo_path):
        tree = parse_ast(file)

        for node in tree.walk():
            if node.type == "import":
                dg.add_edge(current_module, node.module, "imports")

            elif node.type == "call":
                caller = get_enclosing_function(node)
                callee = resolve_call(node)
                dg.add_edge(caller, callee, "calls")

            elif node.type == "attribute_access":
                user = get_enclosing_function(node)
                used = resolve_attribute(node)
                dg.add_edge(user, used, "uses")

    return dg
```

---

## Graph-Guided Retrieval

### Navigation Algorithm

```python
def retrieve_context(query: str, csg: Graph, dg: Graph, k: int) -> List[CodeSnippet]:
    # 1. Find entry points via embedding similarity
    entry_points = embedding_search(query, all_nodes, top_k=5)

    # 2. Expand via Code Structure Graph
    structural_context = set()
    for node in entry_points:
        # Get containing class/module
        parent = csg.get_parent(node)
        structural_context.add(parent)
        # Get sibling methods
        siblings = csg.get_children(parent)
        structural_context.update(siblings)

    # 3. Expand via Dependency Graph
    dependency_context = set()
    for node in entry_points:
        # Get callees (what this calls)
        callees = dg.get_neighbors(node, edge_type="calls")
        dependency_context.update(callees)
        # Get callers (what calls this)
        callers = dg.get_reverse_neighbors(node, edge_type="calls")
        dependency_context.update(callers)
        # Get imports
        imports = dg.get_neighbors(node, edge_type="imports")
        dependency_context.update(imports)

    # 4. Rank and select top-k
    all_context = structural_context | dependency_context
    ranked = rank_by_relevance(query, all_context)
    return ranked[:k]
```

### Relevance Ranking

| Factor | Weight | Purpose |
|--------|--------|---------|
| Embedding similarity | 0.3 | Semantic relevance |
| Graph distance | 0.3 | Structural proximity |
| Call frequency | 0.2 | Usage importance |
| Recency of changes | 0.2 | Active code priority |

---

## Performance Results

| Benchmark | Baseline RAG | GraphCodeAgent | Improvement |
|-----------|--------------|----------------|-------------|
| CrossCodeEval | 34% | 48% | +14% |
| RepoEval | 41% | 56% | +15% |
| Multi-file generation | 28% | 45% | +17% |

### Ablation Study

| Configuration | Score |
|---------------|-------|
| No graphs (embedding only) | 34% |
| CSG only | 41% |
| DG only | 43% |
| **Both graphs (full)** | **48%** |

---

## Practical Implications

### For Knowledge & Vibes

GraphCodeAgent's approach applies to our context retrieval:

| GraphCodeAgent | K&V Application |
|----------------|-----------------|
| CSG | File/class structure in edit locus |
| DG | Dependency tracking in beads |
| Graph navigation | bv --robot-suggest for dependencies |
| Ranked retrieval | cass search + cm context |

### Integration Example

```markdown
## Before Implementing a Bead

1. Identify entry points (functions to modify)
2. CSG expansion: Get containing classes, sibling methods
3. DG expansion: Get callers, callees, imports
4. Assemble context: Most relevant code snippets
5. Generate with full context
```

---

## Key Takeaways

1. **Dual graphs beat single** — Structure + dependencies both matter
2. **Graph navigation > embedding search** — Captures code relationships
3. **Multi-file requires graphs** — Can't understand cross-file with embeddings alone
4. **Ranking combines signals** — Semantic + structural + usage
5. **AST parsing essential** — Accurate graphs need proper parsing

---

## Limitations

- Graph construction has startup cost
- Dynamic languages harder to analyze
- Large repos need incremental updates
- Some relationships (runtime) not captured

---

## See Also

- `062-rag-repository-code.md` — RAG for code overview
- `017-context-retrieval-repo-editing.md` — Context retrieval research
- `012-autocoderover.md` — Structure-aware code search
