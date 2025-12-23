---
title: CodeTree - Agent-Guided Tree Search
tags: [tree-search, multi-agent, strategy, pruning, uct, search]
protocols: [P3, P10]
---

# CodeTree: Agent-Guided Tree Search for Code Generation

**Paper:** CodeTree: Agent-guided Tree Search for Code Generation with Large Language Models
**URL:** https://arxiv.org/html/2411.04329v2
**Date:** November 2024
**Venue:** arXiv preprint

---

## Summary

Critical research demonstrating that **explicit tree search** over solution strategies and implementations dramatically improves code generation. CodeTree organizes attempts as a tree structure: different strategies branch at the top, implementations below, refinements under each. Multi-agent roles (Thinker, Solver, Debugger, Critic) control search through execution feedback and self-critique.

**Key insight:** Hard problems have huge search spaces. Success requires systematic exploration with pruning, not random sampling.

---

## The Tree Search Problem

### Why Linear Search Fails

```
Traditional Approach: Try strategies sequentially

Attempt 1: Greedy approach      → Timeout (wrong algorithm)
Attempt 2: Dynamic programming  → Logic error (wrong subproblem)
Attempt 3: Binary search        → Edge case failure (incomplete)
[Give up or try random variations]

Problem: No systematic exploration, no learning across attempts
```

### The Search Space

```
┌─────────────────────────────────────────────────────────────────┐
│                SOLUTION SEARCH SPACE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                         Problem                                  │
│                            │                                     │
│              ┌─────────────┼─────────────┐                       │
│              │             │             │                       │
│         Strategy A    Strategy B    Strategy C                  │
│         (Greedy)        (DP)       (Binary Search)               │
│              │             │             │                       │
│        ┌─────┼─────┐       │       ┌─────┼─────┐               │
│        │     │     │       │       │     │     │                │
│      Impl  Impl  Impl    Impl    Impl  Impl  Impl               │
│       1     2     3       1       1     2     3                  │
│        │     │     │       │       │     │     │                │
│      ┌─┼─┐ ┌─┼─┐ ┌─┼─┐   ┌─┼─┐   ┌─┼─┐ ┌─┼─┐ ┌─┼─┐            │
│      │ │ │ │ │ │ │ │ │   │ │ │   │ │ │ │ │ │ │ │ │             │
│     Fix Fix Fix Fix Fix Fix Fix   Fix Fix Fix Fix Fix Fix       │
│                                                                  │
│  Total possible paths: 100s                                     │
│  CodeTree systematically explores this tree                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## The CodeTree Architecture

### Four Agent Roles

```
┌─────────────────────────────────────────────────────────────────┐
│                    CODETREE AGENT ROLES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  THINKER (Strategy Generation)                            │   │
│  │                                                           │   │
│  │  Role: Generate diverse high-level strategies            │   │
│  │  Input: Problem statement, failed paths (if any)         │   │
│  │  Output: Natural language strategy description           │   │
│  │                                                           │   │
│  │  Example:                                                 │   │
│  │  "Use sliding window to track subarray sums.             │   │
│  │   Expand window when sum < target.                       │   │
│  │   Contract window when sum >= target."                   │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  SOLVER (Implementation)                                  │   │
│  │                                                           │   │
│  │  Role: Convert strategy into working code                │   │
│  │  Input: Strategy description, problem statement          │   │
│  │  Output: Python/Java/C++ implementation                  │   │
│  │                                                           │   │
│  │  Generates multiple implementations per strategy         │   │
│  │  (variations in implementation details)                  │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  DEBUGGER (Iterative Repair)                              │   │
│  │                                                           │   │
│  │  Role: Fix failing implementations                       │   │
│  │  Input: Code, test failures, strategy                    │   │
│  │  Output: Debugged code                                   │   │
│  │                                                           │   │
│  │  Uses execution feedback to guide repairs                │   │
│  │  Max 3 iterations per implementation (decay limit)       │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  CRITIC (Search Control)                                  │   │
│  │                                                           │   │
│  │  Role: Decide which nodes to expand/prune                │   │
│  │  Input: Tree state, execution results, prior attempts    │   │
│  │  Output: Expand/prune/stop decisions                     │   │
│  │                                                           │   │
│  │  Considers:                                               │   │
│  │  - Test pass rate                                        │   │
│  │  - Error patterns                                        │   │
│  │  - Strategy promise                                      │   │
│  │  - Resource budget (time/tokens)                         │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tree Search Algorithm

### Expansion and Pruning

```
┌─────────────────────────────────────────────────────────────────┐
│                  CODETREE SEARCH ALGORITHM                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Initialize: Root node = Problem statement                      │
│                                                                  │
│  While budget remains and no solution found:                    │
│                                                                  │
│    1. SELECT: Choose node to expand (UCT algorithm)             │
│       ├─ Balance exploitation (promising nodes)                 │
│       └─ and exploration (unexplored nodes)                     │
│                                                                  │
│    2. EXPAND: Generate children based on node type              │
│       ├─ Problem node → Generate strategies (Thinker)           │
│       ├─ Strategy node → Generate implementations (Solver)      │
│       └─ Implementation node → Generate fixes (Debugger)        │
│                                                                  │
│    3. EVALUATE: Run tests on generated code                     │
│       ├─ All tests pass → SUCCESS, return solution              │
│       ├─ Some tests pass → Partial credit, continue             │
│       └─ All tests fail → Low priority for expansion            │
│                                                                  │
│    4. BACKPROP: Update ancestor scores                          │
│       ├─ Success propagates up the tree                         │
│       ├─ Failure decreases ancestor scores                      │
│       └─ Guides future selection                                │
│                                                                  │
│    5. PRUNE: Critic decides what to abandon                     │
│       ├─ Dead-end strategies (repeated failures)                │
│       ├─ Exhausted debugging (3+ attempts)                      │
│       └─ Dominated approaches (worse than alternatives)         │
│                                                                  │
│  End while                                                       │
│                                                                  │
│  Return: Best solution found or failure                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Selection Strategy (UCT)

```
Upper Confidence bound for Trees (UCT):

For each node n:

UCT(n) = exploitation_score(n) + exploration_bonus(n)

       = (wins / visits) + C × sqrt(log(parent_visits) / visits)
         ─────────────────   ──────────────────────────────────
         How good is it?     How under-explored is it?

Where:
- wins: Number of successful paths through this node
- visits: Number of times this node was explored
- parent_visits: Number of times parent was explored
- C: Exploration constant (typically √2)

Example:
Node A: 5 wins / 10 visits, parent visited 50 times
  = 0.5 + √2 × sqrt(log(50) / 10)
  = 0.5 + 1.41 × 0.62
  = 1.37

Node B: 2 wins / 3 visits, parent visited 50 times
  = 0.67 + √2 × sqrt(log(50) / 3)
  = 0.67 + 1.41 × 1.13
  = 2.26  ← Selected! (less explored, decent win rate)
```

---

## Performance Results

### Benchmark Results

```
┌──────────────────────────────────────────────────────────────┐
│         CODETREE PERFORMANCE (GPT-4o)                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  HumanEval:                                                  │
│  ├─ Naive sampling:        85%                              │
│  └─ CodeTree:              95%  (+10%)                       │
│                                                              │
│  MBPP:                                                       │
│  ├─ Naive sampling:        92%                              │
│  └─ CodeTree:              99%  (+7%)                        │
│                                                              │
│  CodeContests:                                               │
│  ├─ Naive sampling:        34%                              │
│  └─ CodeTree:              43%  (+9%)                        │
│                                                              │
│  LiveCodeBench:                                              │
│  ├─ Naive sampling:        41%                              │
│  └─ CodeTree:              53%  (+12%)                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Observation: Bigger gains on harder benchmarks (CodeContests, LiveCodeBench)
```

### Search Efficiency

```
Problem Solved Analysis:

Average Nodes Explored to Success:
├─ Naive sequential: 15.3 nodes (all same level)
├─ CodeTree: 8.7 nodes (selective expansion)
└─ Reduction: 43% fewer evaluations

Solution Quality:
├─ Naive: 82% correctness, 18% edge case issues
├─ CodeTree: 94% correctness (+12%)
└─ Better solutions through diverse strategy exploration
```

---

## Example: Tree Search in Action

### Problem: Find Longest Increasing Subsequence

```
┌─────────────────────────────────────────────────────────────────┐
│              SEARCH TREE EXPANSION                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Root: "Find longest increasing subsequence"                    │
│    │                                                             │
│    ├─ Strategy 1: Dynamic Programming (score: 0.85)             │
│    │    ├─ Impl 1.1: Tabulation (PASS all tests) ✓ SUCCESS!     │
│    │    ├─ Impl 1.2: Memoization (PASS all tests) ✓             │
│    │    └─ [No need to explore more - found solution]           │
│    │                                                             │
│    ├─ Strategy 2: Greedy with Binary Search (score: 0.72)       │
│    │    ├─ Impl 2.1: (FAIL test case 5) ✗                       │
│    │    │    └─ Fix 2.1.1: (PASS all tests) ✓                   │
│    │    └─ Impl 2.2: (FAIL test case 3) ✗                       │
│    │         └─ Fix 2.2.1: (FAIL test case 3) ✗                 │
│    │              └─ Fix 2.2.2: (timeout) ✗                     │
│    │                   └─ [PRUNED: exceeded repair limit]       │
│    │                                                             │
│    └─ Strategy 3: Backtracking (score: 0.45)                    │
│         └─ [PRUNED: low score, solution already found]          │
│                                                                  │
│  Result: Found solution at Impl 1.1 (6 nodes explored)          │
│  Efficiency: Avoided exploring Strategy 3 entirely              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation Guide

### Core Tree Search Engine

```python
class CodeTreeSearch:
    """
    Implements CodeTree: tree search with multiple agent roles.
    """

    def __init__(self, thinker, solver, debugger, critic):
        self.thinker = thinker
        self.solver = solver
        self.debugger = debugger
        self.critic = critic
        self.tree = SearchTree()

    def search(self, problem, budget=100):
        """Execute tree search to solve problem."""

        # Initialize root
        root = self.tree.add_node(
            type='problem',
            content=problem,
            parent=None
        )

        iterations = 0

        while iterations < budget:
            # 1. SELECT: Choose node to expand
            node = self._select_node(root)

            if node is None:
                break  # Tree fully explored or pruned

            # 2. EXPAND: Generate children based on node type
            children = self._expand_node(node)

            # 3. EVALUATE: Test generated code
            for child in children:
                if child.type == 'implementation' or child.type == 'fix':
                    result = self._evaluate_code(child, problem)
                    child.test_results = result

                    # Success!
                    if result['all_passed']:
                        return {
                            'solution': child.content,
                            'path': self._get_path(child),
                            'iterations': iterations
                        }

            # 4. BACKPROP: Update ancestor scores
            for child in children:
                self._backpropagate(child)

            # 5. PRUNE: Critic decides what to abandon
            pruned = self.critic.prune_tree(self.tree)

            iterations += 1

        # Return best solution found
        best = self.tree.get_best_solution()
        return {
            'solution': best.content if best else None,
            'iterations': iterations,
            'exhausted': True
        }

    def _select_node(self, root):
        """Select most promising node using UCT."""

        # Find all expandable nodes
        expandable = self.tree.get_expandable_nodes(root)

        if not expandable:
            return None

        # Calculate UCT score for each
        best_node = None
        best_score = -float('inf')

        for node in expandable:
            score = self._calculate_uct(node)
            if score > best_score:
                best_score = score
                best_node = node

        return best_node

    def _calculate_uct(self, node):
        """Calculate UCT score for node selection."""

        if node.visits == 0:
            return float('inf')  # Always explore unvisited nodes

        parent = node.parent
        parent_visits = parent.visits if parent else 1

        exploitation = node.wins / node.visits
        exploration = math.sqrt(math.log(parent_visits) / node.visits)

        C = math.sqrt(2)  # Exploration constant

        return exploitation + C * exploration

    def _expand_node(self, node):
        """Generate children based on node type."""

        children = []

        if node.type == 'problem':
            # Generate strategies
            strategies = self.thinker.generate_strategies(
                node.content,
                num_strategies=5
            )

            for strategy in strategies:
                child = self.tree.add_node(
                    type='strategy',
                    content=strategy,
                    parent=node
                )
                children.append(child)

        elif node.type == 'strategy':
            # Generate implementations
            implementations = self.solver.implement(
                node.content,
                node.get_root().content,  # Original problem
                num_implementations=3
            )

            for impl in implementations:
                child = self.tree.add_node(
                    type='implementation',
                    content=impl,
                    parent=node
                )
                children.append(child)

        elif node.type == 'implementation' or node.type == 'fix':
            # Generate fixes (if not too many attempts)
            if node.repair_attempts < 3:  # Debugging decay limit
                fixes = self.debugger.debug(
                    node.content,
                    node.test_results,
                    node.parent.content  # Strategy
                )

                for fix in fixes:
                    child = self.tree.add_node(
                        type='fix',
                        content=fix,
                        parent=node,
                        repair_attempts=node.repair_attempts + 1
                    )
                    children.append(child)

        return children

    def _evaluate_code(self, node, problem):
        """Run tests on code node."""

        try:
            results = run_tests(problem.tests, node.content)
            return results
        except Exception as e:
            return {
                'all_passed': False,
                'error': str(e),
                'passed': 0,
                'total': len(problem.tests)
            }

    def _backpropagate(self, node):
        """Update ancestor scores based on node result."""

        current = node

        while current is not None:
            current.visits += 1

            # If this node or any descendant succeeded
            if self._has_success(node):
                current.wins += 1

            current = current.parent

    def _has_success(self, node):
        """Check if node or any descendant has passing solution."""

        if hasattr(node, 'test_results'):
            if node.test_results.get('all_passed', False):
                return True

        for child in node.children:
            if self._has_success(child):
                return True

        return False
```

### Critic Agent (Pruning Logic)

```python
class CriticAgent:
    """
    Decides which tree nodes to prune.
    """

    def prune_tree(self, tree):
        """Identify nodes to prune."""

        pruned = []

        for node in tree.get_all_nodes():
            # Prune if: Exceeded repair attempts
            if node.type in ['implementation', 'fix']:
                if node.repair_attempts >= 3:
                    tree.prune_node(node)
                    pruned.append(node)

            # Prune if: Strategy repeatedly fails
            if node.type == 'strategy':
                if self._is_dead_end(node):
                    tree.prune_node(node)
                    pruned.append(node)

            # Prune if: Dominated by better strategy
            if node.type == 'strategy':
                if self._is_dominated(node):
                    tree.prune_node(node)
                    pruned.append(node)

        return pruned

    def _is_dead_end(self, strategy_node):
        """Check if strategy has no promising paths."""

        # If all implementations have failed
        implementations = [c for c in strategy_node.children
                          if c.type == 'implementation']

        if len(implementations) >= 3:  # Tried multiple implementations
            all_failed = all(
                not c.test_results.get('all_passed', False)
                for c in implementations
                if hasattr(c, 'test_results')
            )

            return all_failed

        return False

    def _is_dominated(self, strategy_node):
        """Check if another strategy is clearly better."""

        siblings = strategy_node.parent.children

        for sibling in siblings:
            if sibling == strategy_node:
                continue

            # If sibling has much better success rate
            if sibling.wins / max(sibling.visits, 1) > \
               2 * (strategy_node.wins / max(strategy_node.visits, 1)):
                return True

        return False
```

---

## Integration with K&V Workflow

### Tree Search as Calibration

```yaml
# Use CodeTree-style search in calibration

# Phase: Multiple strategy exploration
- id: bd-strategy-a
  title: "Explore Strategy A"
  phase: exploration
  status: ready

- id: bd-strategy-b
  title: "Explore Strategy B"
  phase: exploration
  status: ready

- id: bd-strategy-c
  title: "Explore Strategy C"
  phase: exploration
  status: ready

# Calibration evaluates tree and prunes
# Expands most promising branches
```

### Search Control in Multi-Agent System

```python
# Coordinator acts as Critic

def coordinate_search(active_beads, results):
    """Coordinator controls tree search."""

    # Evaluate results from parallel execution
    for bead in active_beads:
        if bead.status == 'completed':
            update_search_tree(bead.id, bead.results)

    # Decide next expansion
    next_nodes = select_nodes_to_expand(search_tree)

    # Prune dead ends
    pruned = prune_unsuccessful_branches(search_tree)

    # Create beads for next iteration
    for node in next_nodes:
        create_bead_for_node(node)

    # Message about pruning
    if pruned:
        send_message(
            subject="[SEARCH CONTROL] Pruned branches",
            body=f"Pruned {len(pruned)} unpromising branches"
        )
```

---

## Key Takeaways

1. **Tree search beats linear search** — Systematic exploration finds solutions faster
2. **Multi-agent roles enable search** — Thinker, Solver, Debugger, Critic each have clear jobs
3. **UCT balances exploration/exploitation** — Don't over-commit to first promising path
4. **Pruning is critical** — Avoid wasting budget on dead ends
5. **Debugging has limits** — 3-attempt limit matches decay research
6. **Backpropagation guides search** — Successes make ancestor nodes more attractive
7. **Execution feedback drives decisions** — Tests determine which branches to explore

---

## See Also

- `019-plansearch.md` — Plan-based search strategies
- `060-debugging-decay-index.md` — Why repair attempts should be limited
- `018-livecodebench.md` — Self-repair evaluation
- `038-adapt.md` — Adaptive decomposition and search
- `059-multi-agent-orchestrator-2025.md` — Multi-agent coordination patterns
