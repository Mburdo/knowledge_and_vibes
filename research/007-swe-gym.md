# SWE-Gym: Training Software Engineering Agents

**Paper:** Training Software Engineering Agents and Verifiers with SWE-Gym
**URL:** https://arxiv.org/abs/2412.21139
**Date:** December 2024

---

## Summary

SWE-Gym is the first large-scale environment designed specifically for **training** (not just evaluating) software engineering agents. It provides 2,438 real Python tasks from production repositories with executable environments and unit tests, enabling rejection sampling fine-tuning on successful agent trajectories.

**Key innovation:** Training on task-specific trajectories that pass tests, combined with outcome-supervised verifiers for inference-time scaling.

**Key result:** Qwen-2.5 32B achieves **32.0% on SWE-Bench Verified** with verifier guidance - new SOTA for open-weight models.

---

## The Training Gap

### Why We Need SWE-Gym

```
┌─────────────────────────────────────────────────────────────────┐
│              EVALUATION vs TRAINING BENCHMARKS                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Evaluation Benchmarks (SWE-bench, HumanEval):                   │
│  ├── Test agent performance                                      │
│  ├── Compare different approaches                                │
│  └── ⚠️  But can't train on them (test set contamination)        │
│                                                                  │
│  Training Benchmarks (SWE-Gym):                                  │
│  ├── Generate training trajectories                              │
│  ├── Fine-tune agents on successful paths                        │
│  ├── Train verifiers on outcomes                                 │
│  └── ✓ Separate from evaluation sets                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Before SWE-Gym

| Limitation | Impact |
|------------|--------|
| No training data | Models rely solely on pre-training |
| Synthetic tasks | Don't match real-world complexity |
| Contamination risk | Can't train on test benchmarks |
| No ground truth trajectories | Can't learn successful strategies |

---

## Dataset Composition

### SWE-Gym Variants

| Dataset | Tasks | Purpose | Use Case |
|---------|-------|---------|----------|
| **SWE-Gym** | 2,438 | Core training set | Full-scale fine-tuning |
| **SWE-Gym Lite** | 230 | Fast prototyping | Quick experiments |
| **SWE-Gym Raw** | 66,894 | Future research | Data augmentation, filtering |

### Source Repositories

```
Repository Distribution (2,438 tasks):

pandas     ████████████████████ 487 tasks (20%)
MONAI      ████████████ 312 tasks (13%)
moto       ██████████ 276 tasks (11%)
mypy       █████████ 245 tasks (10%)
dask       ████████ 218 tasks (9%)
dvc        ███████ 197 tasks (8%)
conan      ██████ 183 tasks (8%)
pydantic   █████ 164 tasks (7%)
hydra      ████ 152 tasks (6%)
bokeh      ████ 124 tasks (5%)
modin      ███ 80 tasks (3%)

Total: 11 repositories, 2,438 tasks
```

**Why these repositories:**
- Production code (not toy examples)
- Active maintenance
- Comprehensive test suites
- Diverse domains (data processing, ML, infrastructure, validation)
- Real-world complexity

### Task Characteristics

```
┌─────────────────────────────────────────────────────────────────┐
│                       TASK ANATOMY                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Components of each SWE-Gym task:                                │
│                                                                  │
│  1. Repository state                                             │
│     └── Specific commit hash                                     │
│                                                                  │
│  2. Issue description                                            │
│     ├── Natural language problem statement                       │
│     └── Expected behavior                                        │
│                                                                  │
│  3. Test suite                                                   │
│     ├── Unit tests that must pass                                │
│     └── Regression tests                                         │
│                                                                  │
│  4. Executable environment                                       │
│     ├── Docker container with dependencies                       │
│     └── Isolated execution                                       │
│                                                                  │
│  5. Gold patch (hidden during training)                          │
│     └── Reference solution for validation                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Training Methodology

### Rejection Sampling Fine-Tuning

The core training approach: only keep trajectories that pass tests.

```
┌─────────────────────────────────────────────────────────────────┐
│              REJECTION SAMPLING WORKFLOW                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  For each task in SWE-Gym:                                       │
│                                                                  │
│  1. Generate N candidate trajectories                            │
│     ├── Agent attempts to solve task                             │
│     ├── Records actions, thoughts, edits                         │
│     └── Produces patch                                           │
│                                                                  │
│  2. Execute tests on each patch                                  │
│     ├── Apply patch to repository                                │
│     ├── Run test suite                                           │
│     └── Record: pass ✓ or fail ✗                                │
│                                                                  │
│  3. Filter trajectories                                          │
│     ├── KEEP: Trajectories where tests pass ✓                   │
│     └── REJECT: Trajectories where tests fail ✗                 │
│                                                                  │
│  4. Fine-tune on successful trajectories                         │
│     └── Model learns from working solutions only                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Training Statistics

**Trajectory generation (Qwen-2.5 32B baseline):**
- Total tasks attempted: 2,438
- Successful trajectories: 491 (20.1% solve rate)
- Failed trajectories: 1,947 (rejected)
- Fine-tuning data: 491 successful trajectories

**Computational cost:**
- Trajectory generation: ~100 GPU hours (A100)
- Fine-tuning: ~50 GPU hours
- Total: ~150 GPU hours for full training run

**Limitation noted:**
> "We generated 491 training trajectories, limited by compute budget rather than task availability. With more compute, we could generate multiple successful trajectories per task."

---

## Results

### Performance Improvements

**Qwen-2.5 32B on SWE-Bench:**

| Benchmark | Before Training | After Training | Improvement |
|-----------|----------------|----------------|-------------|
| SWE-Bench Lite | 3.0% | 15.3% | **+12.3%** |
| SWE-Bench Verified | 7.0% | 20.6% | **+13.6%** |

```
Performance Gains from Training

SWE-Bench Lite:
Before: ███ 3.0%
After:  ███████████████ 15.3%  [+410% relative]

SWE-Bench Verified:
Before: ███████ 7.0%
After:  ████████████████████ 20.6%  [+194% relative]
```

### Inference-Time Scaling with Verifiers

**Training outcome-supervised reward models (verifiers):**

```
┌─────────────────────────────────────────────────────────────────┐
│                    VERIFIER TRAINING                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Input: Agent trajectory (actions, thoughts, edits)              │
│  Output: P(success) ∈ [0, 1]                                     │
│                                                                  │
│  Training data:                                                  │
│  ├── Successful trajectories → Label: 1.0                        │
│  └── Failed trajectories → Label: 0.0                            │
│                                                                  │
│  Inference strategy:                                             │
│  1. Generate K candidate solutions                               │
│  2. Score each with verifier                                     │
│  3. Select highest-scoring solution                              │
│  4. Apply to repository                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Results with verifier (Best@K selection):**

| Benchmark | Base Agent | + Verifier (Best@8) | Improvement |
|-----------|-----------|---------------------|-------------|
| SWE-Bench Verified | 20.6% | **32.0%** | +11.4% |
| SWE-Bench Lite | 15.3% | **26.0%** | +10.7% |

**Comparison to proprietary systems:**
- Claude Opus 3.5: 35.6% on SWE-Bench Verified
- Qwen-2.5 32B + Verifier: 32.0%
- **Gap closing:** Open-weight models reaching competitive performance

### Analysis: Pass@K vs Best@K

```
┌─────────────────────────────────────────────────────────────────┐
│                  PASS@K vs BEST@K GAP                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Pass@K (at least one solution in K attempts works):            │
│  ████████████████████████████ 42.3%                             │
│                                                                  │
│  Best@K (verifier selects best of K attempts):                  │
│  ████████████████████ 32.0%                                     │
│                                                                  │
│  Gap: 10.3 percentage points                                     │
│                                                                  │
│  Interpretation:                                                 │
│  └── Verifier is good but not perfect at identifying             │
│      successful solutions. Opportunity for better reward models. │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Specialized Agent Performance

### MoatlessTools Integration

The paper tested integration with specialized workflow agents (MoatlessTools).

**Results:**
- MoatlessTools baseline: 8.2% on SWE-Bench Verified
- After SWE-Gym training: 9.1% (+0.9%)
- Generic agent after training: 20.6%

**Interpretation:**
> "Specialized tools and workflows can become rigid constraints that prevent models from learning new strategies. Generic agents with good tools showed better improvement from training."

### Implications for Agent Design

| Design Choice | Training Effectiveness |
|---------------|----------------------|
| **Flexible workflows** | High improvement potential |
| **Rigid tool sequences** | Limited improvement |
| **Tool-rich environment** | Good (if tools are optional) |
| **Prescribed steps** | Bad (prevents learning) |

---

## Self-Improvement Experiments

### On-Policy vs Off-Policy Training

The paper tested mixing different data sources:

```
Training Data Composition Experiments:

1. Off-policy only (SWE-Gym trajectories):
   ████████████████████ 20.6% ✓ Baseline

2. On-policy + off-policy (50/50 mix):
   ████████████ 15.2% ✗ Degradation

3. On-policy only (model's own trajectories):
   ████████ 11.7% ✗ Worse

Conclusion: Off-policy learning from diverse trajectories
works better than self-improvement for this task.
```

**Why self-improvement struggled:**
1. Model's own trajectories less diverse than curated set
2. Reinforces existing biases and patterns
3. Needs more sophisticated curriculum learning

---

## Practical Implications

### For Agent Development

**What works:**
1. Training on successful trajectories (rejection sampling)
2. Test-based verification as reward signal
3. Outcome-supervised verifiers for inference scaling
4. Open-weight models can reach competitive performance
5. Generic + flexible > specialized + rigid

**What doesn't work:**
1. Naive self-improvement (on-policy training)
2. Mixing on-policy and off-policy data carelessly
3. Over-specialized workflows that prevent learning
4. Ignoring the Pass@K vs Best@K gap

### For K&V Workflow

| K&V Pattern | SWE-Gym Support |
|-------------|-----------------|
| **Test-driven validation** | Core success signal |
| **Acceptance criteria** | Tests define correctness |
| **Multiple attempts** | Pass@K shows value of tries |
| **Verifier agents** | Separate generation from evaluation |
| **Fresh attempts** | Better than self-iteration |

---

## Training Your Own Agents

### Recipe

```python
# 1. Collect trajectories on SWE-Gym tasks
trajectories = []
for task in swe_gym_tasks:
    for attempt in range(n_attempts):
        trajectory = agent.solve(task)
        test_result = run_tests(trajectory.patch, task)

        if test_result.passed:
            trajectories.append({
                'task': task,
                'trajectory': trajectory,
                'success': True
            })

# 2. Fine-tune on successful trajectories
model.fine_tune(
    data=[t for t in trajectories if t['success']],
    learning_rate=1e-5,
    epochs=3
)

# 3. Train verifier on all trajectories
verifier.train(
    data=trajectories,
    labels=[t['success'] for t in trajectories]
)

# 4. Inference with verifier
def solve_with_verifier(task, k=8):
    candidates = [model.solve(task) for _ in range(k)]
    scores = [verifier.score(c) for c in candidates]
    best_idx = np.argmax(scores)
    return candidates[best_idx]
```

### Cost Estimates

**Based on paper's Qwen-2.5 32B experiments:**

| Component | GPU Hours | Cost (A100 at $2/hr) |
|-----------|-----------|---------------------|
| Trajectory generation | 100 | $200 |
| Fine-tuning | 50 | $100 |
| Verifier training | 20 | $40 |
| **Total** | **170** | **$340** |

**For smaller models (7B-13B):**
- 3-5x faster
- $50-$100 total cost

---

## Limitations & Future Work

### Known Limitations

1. **Limited trajectory diversity**
   - Only 491 successful trajectories
   - Could benefit from multiple solutions per task
   - Compute-limited, not task-limited

2. **Verifier gap**
   - 10.3% gap between Pass@K and Best@K
   - Room for better outcome prediction
   - Could incorporate richer signals (test coverage, code quality)

3. **Specialized agents struggle**
   - MoatlessTools saw minimal improvement
   - Suggests tension between structure and learning
   - Need flexible specialization

4. **Self-improvement challenges**
   - On-policy learning degraded performance
   - Needs curriculum learning approach
   - Diversity of trajectories matters

### Future Directions

**From the paper:**
- Generate multiple successful trajectories per task
- Improve verifier training (better reward modeling)
- Test on other languages beyond Python
- Explore curriculum learning for self-improvement
- Better integration of specialized tools with learning

---

## Code Examples

### Example 1: Trajectory Collection

```python
from swe_gym import load_task
import docker

class TrajectoryCollector:
    """Collect agent trajectories with test-based validation."""

    def __init__(self, agent, docker_client):
        self.agent = agent
        self.docker = docker_client

    def collect_trajectory(self, task_id: str) -> dict:
        """Generate and validate a single trajectory."""
        task = load_task(task_id)

        # Agent solves task
        trajectory = self.agent.solve(
            issue=task.issue_description,
            repository=task.repo_path,
            commit=task.base_commit
        )

        # Execute tests
        container = self.docker.run(
            image=task.docker_image,
            command=f"pytest {task.test_path}",
            volumes={task.repo_path: '/workspace'}
        )

        test_output = container.logs()
        success = "PASSED" in test_output

        return {
            'task_id': task_id,
            'trajectory': trajectory.to_dict(),
            'success': success,
            'test_output': test_output,
            'patch': trajectory.get_patch()
        }

    def collect_dataset(self, task_ids: list[str], n_per_task: int = 5):
        """Collect multiple trajectories per task."""
        dataset = []

        for task_id in task_ids:
            print(f"Collecting trajectories for {task_id}...")

            for attempt in range(n_per_task):
                traj = self.collect_trajectory(task_id)
                dataset.append(traj)

                if traj['success']:
                    print(f"  ✓ Attempt {attempt + 1}: SUCCESS")
                else:
                    print(f"  ✗ Attempt {attempt + 1}: FAILED")

        return dataset
```

### Example 2: Rejection Sampling Fine-Tuning

```python
from transformers import AutoModelForCausalLM, TrainingArguments, Trainer
import torch

class RejectionSamplingTrainer:
    """Fine-tune on successful trajectories only."""

    def __init__(self, model_name: str):
        self.model = AutoModelForCausalLM.from_pretrained(model_name)
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)

    def prepare_training_data(self, trajectories: list[dict]):
        """Filter to successful trajectories and format for training."""
        successful = [t for t in trajectories if t['success']]

        print(f"Filtered: {len(successful)}/{len(trajectories)} successful")

        # Format as instruction-following examples
        examples = []
        for traj in successful:
            example = {
                'prompt': self._format_prompt(traj),
                'completion': self._format_completion(traj)
            }
            examples.append(example)

        return examples

    def _format_prompt(self, trajectory: dict) -> str:
        """Convert task to prompt."""
        return f"""<|system|>
You are an expert software engineer. Fix the following issue.
<|user|>
Repository: {trajectory['task']['repo']}
Issue: {trajectory['task']['issue_description']}
<|assistant|>
"""

    def _format_completion(self, trajectory: dict) -> str:
        """Convert successful trajectory to completion."""
        # Include the agent's thought process and final patch
        steps = trajectory['trajectory']['steps']
        return "\n".join([
            step['thought'] + "\n" + step['action']
            for step in steps
        ])

    def train(self, trajectories: list[dict], output_dir: str):
        """Fine-tune model on successful trajectories."""
        train_data = self.prepare_training_data(trajectories)

        training_args = TrainingArguments(
            output_dir=output_dir,
            num_train_epochs=3,
            per_device_train_batch_size=4,
            learning_rate=1e-5,
            warmup_steps=100,
            logging_steps=10,
            save_steps=500,
        )

        trainer = Trainer(
            model=self.model,
            args=training_args,
            train_dataset=train_data,
        )

        trainer.train()
```

### Example 3: Verifier Training

```python
class OutcomeVerifier:
    """Train a model to predict trajectory success."""

    def __init__(self, base_model: str):
        self.model = AutoModelForSequenceClassification.from_pretrained(
            base_model,
            num_labels=1  # Regression: P(success)
        )

    def prepare_verifier_data(self, trajectories: list[dict]):
        """Prepare all trajectories (success + failure) for verifier."""
        examples = []

        for traj in trajectories:
            features = self._extract_features(traj)
            label = 1.0 if traj['success'] else 0.0

            examples.append({
                'features': features,
                'label': label
            })

        return examples

    def _extract_features(self, trajectory: dict) -> str:
        """Convert trajectory to feature representation."""
        # Include: task description, agent's approach, code changes
        return f"""Task: {trajectory['task']['issue_description']}

Agent trajectory:
{trajectory['trajectory']['summary']}

Changes:
{trajectory['patch']}
"""

    def train(self, trajectories: list[dict], output_dir: str):
        """Train verifier on all trajectories."""
        data = self.prepare_verifier_data(trajectories)

        training_args = TrainingArguments(
            output_dir=output_dir,
            num_train_epochs=5,
            per_device_train_batch_size=8,
            learning_rate=2e-5,
        )

        trainer = Trainer(
            model=self.model,
            args=training_args,
            train_dataset=data,
        )

        trainer.train()

    def score(self, trajectory: dict) -> float:
        """Predict probability of success."""
        features = self._extract_features(trajectory)
        inputs = self.tokenizer(features, return_tensors='pt')

        with torch.no_grad():
            logits = self.model(**inputs).logits
            score = torch.sigmoid(logits).item()

        return score

    def select_best(self, candidates: list[dict]) -> dict:
        """Select highest-scoring trajectory."""
        scores = [self.score(c) for c in candidates]
        best_idx = np.argmax(scores)
        return candidates[best_idx]
```

### Example 4: Inference with Verifier

```python
def solve_with_verifier(
    task,
    agent,
    verifier,
    k: int = 8
) -> dict:
    """
    Generate K candidate solutions and select best via verifier.
    Implements Best@K strategy from paper.
    """
    print(f"Generating {k} candidate solutions...")
    candidates = []

    for i in range(k):
        trajectory = agent.solve(task)
        score = verifier.score(trajectory)

        candidates.append({
            'trajectory': trajectory,
            'score': score,
            'attempt': i + 1
        })

        print(f"  Candidate {i+1}: score={score:.3f}")

    # Select best
    best = max(candidates, key=lambda c: c['score'])
    print(f"\nSelected candidate {best['attempt']} (score={best['score']:.3f})")

    return best['trajectory']

# Usage
task = load_task("pandas-issue-12345")
solution = solve_with_verifier(task, agent, verifier, k=8)
```

---

## Key Takeaways

1. **Training on successful trajectories works** - 12-13% absolute improvement from rejection sampling fine-tuning

2. **Test-based reward signal is sufficient** - No need for complex reward modeling; tests define success

3. **Verifiers enable scaling** - Best@K selection adds 10%+ improvement at inference time

4. **Open-weight models are competitive** - 32% on SWE-Bench Verified approaches proprietary systems

5. **Flexible > specialized** - Generic agents with good tools learn better than rigid workflows

6. **Self-improvement is hard** - On-policy training degraded performance; diversity matters

7. **Compute-limited, not data-limited** - Only 491 trajectories used; more compute would help

8. **Pass@K vs Best@K gap** - 10% room for better verifiers and reward models

---

## See Also

- `009-swe-bench.md` - The evaluation benchmark SWE-Gym complements
- `010-swe-agent.md` - Agent-computer interfaces for better performance
- `060-debugging-decay-index.md` - Why fresh attempts beat iteration
- `038-adapt.md` - Decomposition strategies for complex tasks
- `022-chatrepair.md` - Iterative repair approaches
