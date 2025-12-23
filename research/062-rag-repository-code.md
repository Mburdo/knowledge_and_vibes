---
title: Retrieval-Augmented Code Generation 2025
tags: [rag, retrieval, repository-level, embeddings, context, code-search]
protocols: [P2, P5]
---

# Retrieval-Augmented Code Generation 2025

**Sources:**
- "Retrieval-Augmented Code Generation: A Survey with Focus on Repository-Level Approaches" (September 2025)
- CodeRAG-Bench (2024-2025)
- Apple CLaRa: End-to-end RAG model (2025)
- Multiple GitHub projects: code-graph-rag, RAGBase4Code, Code-Repository-RAG

---

## Summary

2025 research consolidating **Retrieval-Augmented Code Generation (RACG)** approaches, with emphasis on **Repository-Level Code Generation (RLCG)** — understanding entire codebases, not just snippets.

**Key insight:** LLMs are only as good as the context they're given. RAG provides the right context.

---

## The Repository-Level Challenge

### Why Repository-Level is Hard

```
Single File Generation:
├── Context: The file itself
├── Dependencies: Imports visible
└── Difficulty: Low

Repository-Level Generation:
├── Context: Thousands of files
├── Dependencies: Cross-file, cross-module
├── Conventions: Project-specific patterns
├── Architecture: System design decisions
└── Difficulty: Very High
```

### The Context Window Problem

| Codebase Size | Files | Tokens | Fits in Context? |
|---------------|-------|--------|------------------|
| Small lib | 10-50 | 50K | Maybe |
| Medium project | 100-500 | 500K | No |
| Large codebase | 1000+ | 5M+ | Definitely not |

**Solution:** Retrieve only what's relevant, not everything.

---

## RAG Architecture for Code

### Core Components

```
┌─────────────────────────────────────────────────────────────────┐
│                        RAG PIPELINE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. INDEXING PHASE (one-time)                                    │
│  ─────────────────────────────                                   │
│  Codebase → Chunker → Embeddings → Vector Store                  │
│                                                                  │
│  2. RETRIEVAL PHASE (per query)                                  │
│  ──────────────────────────────                                  │
│  Query → Query Embedding → Similarity Search → Top-K Chunks      │
│                                                                  │
│  3. GENERATION PHASE                                             │
│  ────────────────────                                            │
│  [Query + Retrieved Chunks] → LLM → Generated Code               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Chunking Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **File-level** | Whole files as chunks | High-level understanding |
| **Function-level** | Individual functions | Implementation details |
| **AST-aware** | Syntax-tree boundaries | Semantic coherence |
| **Sliding window** | Overlapping windows | Dense code |

### 2025 Best Practice: Dual Granularity

From Code-Repository-RAG:

```
Dual Granularity Processing:
├── File-Level: High-level summaries for architecture
└── Chunk-Level: Detailed code for implementation

Query: "How does authentication work?"
├── File-level retrieval: auth/module.py, auth/jwt.py (overview)
└── Chunk-level retrieval: validate_token(), create_session() (details)
```

---

## Advanced RAG Techniques

### 1. Graph-Based RAG (code-graph-rag)

Uses knowledge graphs to capture relationships:

```
Code Graph:
├── Nodes: Functions, Classes, Modules
├── Edges: Calls, Imports, Inherits
└── Query: Traverse graph to find related code

Example Query: "What calls the validate_token function?"
├── Graph traversal finds all callers
└── Returns: login(), refresh_session(), middleware.auth()
```

### 2. Apple CLaRa (2025)

End-to-end RAG with document compression:

| Feature | Value |
|---------|-------|
| Compression rate | 32x-64x |
| Information preserved | Essential |
| Training | Three-stage (compression + tuning + E2E) |

### 3. Triple-Agent Retrieval (RAGBase4Code)

```
Agent 1: Source Code Retriever
├── Navigates codebase
└── Extracts functions, classes, implementations

Agent 2: Documentation Retriever
├── Searches docs, comments, READMEs
└── Extracts explanations, usage examples

Agent 3: Context Synthesizer
├── Combines code + docs
└── Produces coherent context
```

---

## CodeRAG-Bench Findings

Benchmark testing RAG for code generation:

### When RAG Helps Most

| Scenario | RAG Improvement |
|----------|-----------------|
| Using project-specific APIs | High |
| Following codebase conventions | High |
| Cross-file dependencies | High |
| Novel algorithms | Low |
| Generic utility functions | Low |

### What to Retrieve

| Retrieval Target | Impact on Generation |
|------------------|---------------------|
| Similar implementations | High (patterns) |
| API documentation | High (correct usage) |
| Type definitions | Medium (constraints) |
| Test examples | Medium (expected behavior) |
| Config files | Low (unless config-related) |

---

## Practical Implementation

### Indexing a Codebase

```python
# Using Tree-sitter for syntax-aware chunking
from tree_sitter import Parser

def chunk_codebase(repo_path):
    chunks = []
    for file in glob(f"{repo_path}/**/*.py"):
        tree = parser.parse(read_file(file))
        for node in tree.root_node.children:
            if node.type in ['function_definition', 'class_definition']:
                chunks.append({
                    'content': node.text,
                    'file': file,
                    'type': node.type,
                    'name': get_name(node)
                })
    return chunks
```

### Query-Time Retrieval

```python
def retrieve_context(query, k=5):
    # Embed the query
    query_embedding = embed(query)

    # Search vector store
    results = vector_store.similarity_search(query_embedding, k=k)

    # Return formatted context
    return format_for_prompt(results)
```

### Integration with LLM

```python
def generate_with_rag(query, codebase):
    # Retrieve relevant context
    context = retrieve_context(query, codebase)

    # Construct prompt with context
    prompt = f"""
    Based on the following code context from the repository:

    {context}

    Generate code to: {query}

    Follow the patterns and conventions shown in the context.
    """

    return llm.generate(prompt)
```

---

## Integration with Knowledge & Vibes

### CASS Search as RAG

Our CASS tool provides RAG-like retrieval:

```bash
# Find relevant patterns
cass search "authentication flow" --robot --limit 5

# Find implementation examples
cass search "JWT validation" --robot --code-only
```

### cm context as Pattern RAG

```bash
# Get patterns and anti-patterns for a task
cm context "user authentication" --json

# Returns: patterns, anti-patterns, historical context
```

### Enhanced Bead Execution

```
Before implementing:
1. cass search "{bead_title}" → Find similar past work
2. cm context "{task}" → Get patterns/anti-patterns
3. Read retrieved files → Understand conventions
4. Then implement following patterns
```

---

## Key Takeaways

1. **RAG is essential for repository-level generation** — Context windows can't hold everything
2. **Chunking strategy matters** — AST-aware beats naive splitting
3. **Dual granularity** — File-level for overview, chunk-level for details
4. **Graph relationships** — Capture calls, imports, inheritance
5. **Quality over quantity** — Better retrieval beats more retrieval
6. **Project conventions** — RAG helps LLMs follow existing patterns

---

## See Also

- `017-context-retrieval-repo-editing.md` — Earlier context retrieval research
- `057-anthropic-context-engineering.md` — Context management strategies
- `.claude/skills/cass-search/SKILL.md` — CASS tool for pattern retrieval
