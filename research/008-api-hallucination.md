# API Hallucination and Documentation Augmented Generation

**Paper:** On Mitigating Code LLM Hallucinations with API Documentation
**URL:** https://arxiv.org/abs/2407.09726
**Date:** July 2024

---

## Summary

Code LLMs hallucinate API invocations at alarming rates, especially for **low-frequency APIs** rarely seen during training. GPT-4o achieves only **38.58% valid invocations** for low-frequency cloud APIs. The paper introduces **Documentation Augmented Generation (DAG)** to selectively retrieve API documentation when needed, improving low-frequency API accuracy from 41.80% to 86.82%.

**Key insight:** The problem isn't uniform - common APIs work fine, but rare APIs need grounding in documentation.

---

## The API Hallucination Problem

### What Gets Hallucinated

```
┌─────────────────────────────────────────────────────────────────┐
│                   API HALLUCINATION TYPES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Non-existent APIs                                            │
│     └── Inventing methods that don't exist                       │
│         Example: boto3.client('s3').get_bucket_metadata()        │
│         Reality: No such method exists                           │
│                                                                  │
│  2. Wrong syntax                                                 │
│     └── Using incorrect argument patterns                        │
│         Example: s3.put_object(Bucket, Key, Content)             │
│         Reality: s3.put_object(Bucket='x', Key='y', Body=data)   │
│                                                                  │
│  3. Hallucinated arguments                                       │
│     └── Adding parameters that don't exist                       │
│         Example: ec2.create_instance(..., auto_backup=True)      │
│         Reality: No auto_backup parameter                        │
│                                                                  │
│  4. Wrong parameter types                                        │
│     └── Using incorrect data types                               │
│         Example: put_object(Body="string")                       │
│         Reality: Body should be bytes or file-like object        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Frequency Matters

The paper categorizes APIs by training data frequency:

| Frequency Category | Occurrences | Definition |
|-------------------|-------------|------------|
| **Low** | 0-10 | Rarely seen during pre-training |
| **Medium** | 11-100 | Occasionally encountered |
| **High** | 101+ | Common in training data |

**Performance by frequency:**

```
API Invocation Correctness

High-frequency APIs:
StarCoder2-15B: ██████████████████ 88.78%
GPT-4o:         ███████████████████ 93.66%

Low-frequency APIs:
StarCoder2-15B: █████ 24.72%
GPT-4o:         ████████ 38.58%

Interpretation: Models memorize common APIs but struggle with rare ones
```

---

## CloudAPIBench

### Benchmark Design

**Dataset composition:**
- 622 tasks testing AWS and Azure API invocations
- Real-world cloud infrastructure scenarios
- Categorized by API frequency in training data
- Executable validation (does the code actually work?)

```
┌─────────────────────────────────────────────────────────────────┐
│                  CLOUDAPIBENCH STRUCTURE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each task includes:                                             │
│                                                                  │
│  1. Natural language description                                 │
│     └── "Create an S3 bucket with encryption enabled"            │
│                                                                  │
│  2. API context                                                  │
│     ├── Which service (S3, EC2, Lambda, etc.)                    │
│     └── Required capabilities                                    │
│                                                                  │
│  3. Frequency label                                              │
│     └── High / Medium / Low based on training data               │
│                                                                  │
│  4. Reference solution                                           │
│     └── Correct API invocation for validation                    │
│                                                                  │
│  5. Execution environment                                        │
│     └── Mocked AWS/Azure for testing                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Task Distribution

```
CloudAPIBench (622 tasks):

AWS Services:
  S3 (storage)          ████████████ 156 tasks (25%)
  EC2 (compute)         ██████████ 124 tasks (20%)
  Lambda (serverless)   ████████ 93 tasks (15%)
  DynamoDB (database)   ██████ 81 tasks (13%)
  IAM (identity)        █████ 62 tasks (10%)
  Other AWS            ████ 54 tasks (9%)

Azure Services:
  Storage               ████ 52 tasks (8%)
  Total:                622 tasks
```

---

## Documentation Augmented Generation (DAG)

### The Approach

DAG uses the model's initial attempt as a query to retrieve relevant documentation.

```
┌─────────────────────────────────────────────────────────────────┐
│                    DAG WORKFLOW                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Initial generation (zero-shot)                          │
│     User: "Create S3 bucket with encryption"                     │
│     Model: s3.create_bucket(Bucket='x', Encryption='AES256')     │
│     [Potentially incorrect]                                      │
│                                                                  │
│  Step 2: Extract API from attempt                                │
│     Detected APIs: ['s3.create_bucket']                          │
│                                                                  │
│  Step 3: Retrieve documentation                                  │
│     Query: "s3.create_bucket"                                    │
│     Retrieved: Official AWS SDK docs for create_bucket          │
│                                                                  │
│  Step 4: Re-generate with documentation                          │
│     Context: [User request] + [Retrieved docs]                   │
│     Model: s3.create_bucket(                                     │
│              Bucket='x',                                         │
│              CreateBucketConfiguration={                         │
│                'LocationConstraint': 'us-west-2'                 │
│              },                                                  │
│              ServerSideEncryptionConfiguration={...}             │
│            )                                                     │
│     [Correct - guided by documentation]                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Mathematical Formulation

```
Standard LLM generation:
P(code | task) = LLM(task)

DAG generation:
P(code | task, docs) = LLM(task, retrieve(initial_attempt(task)))

Where:
- initial_attempt(task) = LLM(task) [zero-shot]
- retrieve(code) = fetch_docs(extract_apis(code))
- Final output uses both task and retrieved docs
```

---

## Results

### Performance Improvements

**DAG impact by API frequency:**

| Frequency | Baseline | DAG | Improvement | Change |
|-----------|----------|-----|-------------|--------|
| **Low** | 41.80% | 86.82% | +45.02% | +108% relative |
| **Medium** | 67.23% | 78.91% | +11.68% | +17% relative |
| **High** | 88.78% | 49.76% | -39.02% | -44% relative |

```
Visual Impact of DAG

Low-frequency APIs:
Before DAG: ████████ 41.80%
After DAG:  █████████████████ 86.82%  [HUGE GAIN]

High-frequency APIs:
Before DAG: ██████████████████ 88.78%
After DAG:  ██████████ 49.76%  [MASSIVE LOSS]
```

### The Over-Retrieval Problem

**Why high-frequency APIs degrade with DAG:**

1. **Distraction from known knowledge**
   - Model already knows common APIs
   - Documentation adds noise to context
   - Reduces confidence in memorized patterns

2. **Context pollution**
   - Retrieved docs compete with internal knowledge
   - Model second-guesses correct answers
   - Similar to intrinsic self-correction problem

3. **Unnecessary overhead**
   - Wasting tokens on obvious APIs
   - Could use context for other information

---

## DAG++ (Smart Retrieval)

### The Enhancement

DAG++ only retrieves documentation when needed:

```
┌─────────────────────────────────────────────────────────────────┐
│                    DAG++ DECISION LOGIC                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  For each API in initial attempt:                                │
│                                                                  │
│  ┌─ API exists in known index?                                   │
│  │                                                               │
│  ├── NO → RETRIEVE docs                                          │
│  │   └── API might be rare, hallucinated, or new                 │
│  │                                                               │
│  └── YES → Check confidence                                      │
│      │                                                           │
│      ├── Confidence < threshold → RETRIEVE                       │
│      │   └── Model is uncertain, docs will help                  │
│      │                                                           │
│      └── Confidence ≥ threshold → SKIP retrieval                 │
│          └── Model knows this API, docs would distract           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### DAG++ Results

**Overall performance:**

| Approach | Average Accuracy | Low-freq | High-freq |
|----------|-----------------|----------|-----------|
| Baseline | 69.47% | 41.80% | 88.78% |
| DAG (always retrieve) | 71.83% | 86.82% | 49.76% |
| **DAG++ (selective)** | **77.67%** | **86.82%** | **87.12%** |

```
DAG++ Achieves Best of Both Worlds

Low-frequency APIs:
DAG++: █████████████████ 86.82%  [Matched DAG]

High-frequency APIs:
DAG++: █████████████████ 87.12%  [Nearly matched baseline]

Overall improvement: +8.20% over baseline
```

### Confidence Calibration

The paper found that LLM probability scores over API tokens are well-calibrated:

```python
# Measure confidence from token probabilities
def api_confidence(logprobs: list[float]) -> float:
    """
    Average log probability of tokens in API invocation.
    Higher = more confident
    """
    return np.exp(np.mean(logprobs))

# Empirical findings:
# - Confidence > 0.85: Usually correct (trust the model)
# - Confidence < 0.85: Often wrong (retrieve docs)
```

**Confidence vs correctness correlation:**

| Confidence Range | Correctness Rate | Recommendation |
|-----------------|------------------|----------------|
| 0.90+ | 94.2% | Skip retrieval |
| 0.85-0.90 | 87.3% | Skip retrieval |
| 0.70-0.85 | 68.4% | **Retrieve docs** |
| < 0.70 | 42.1% | **Retrieve docs** |

---

## Integration with K&V Workflow

### When to Use External Documentation

**Retrieval triggers (from DAG++ insights):**

```
┌─────────────────────────────────────────────────────────────────┐
│              DOCUMENTATION RETRIEVAL POLICY                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ALWAYS retrieve for:                                            │
│  ├── Cloud APIs (AWS, Azure, GCP)                                │
│  ├── Domain-specific libraries (not in training data)            │
│  ├── Version-specific behavior ("use library v3.2")              │
│  └── APIs not in known index                                     │
│                                                                  │
│  CONDITIONAL retrieval for:                                      │
│  ├── Low confidence invocations (< 85%)                          │
│  ├── Complex parameter structures                                │
│  └── User explicitly mentions version/behavior                   │
│                                                                  │
│  SKIP retrieval for:                                             │
│  ├── Python standard library (os, json, datetime)                │
│  ├── Extremely common packages (requests, pandas)                │
│  ├── High confidence invocations (> 85%)                         │
│  └── Simple, well-known patterns                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### K&V Implementation

| Workflow Phase | DAG Integration |
|---------------|-----------------|
| **Planning** | Identify API-heavy beads |
| **Execution** | Agent checks confidence on API calls |
| **Repair** | Retrieve docs for failing API invocations |
| **Calibration** | Flag low-confidence API usage |

---

## Code Examples

### Example 1: API Confidence Checker

```python
import re
from typing import Optional
import numpy as np

class APIConfidenceChecker:
    """
    Check confidence in API invocations and decide whether to
    retrieve documentation.
    """

    def __init__(self, confidence_threshold: float = 0.85):
        self.threshold = confidence_threshold

        # Common APIs that don't need docs
        self.known_apis = {
            'os.path', 'json.loads', 'json.dumps',
            'datetime.now', 'requests.get', 'requests.post',
            'pandas.DataFrame', 'numpy.array',
            # ... etc
        }

        # APIs that always need docs
        self.always_retrieve = {
            'boto3', 'azure', 'google.cloud',  # Cloud SDKs
            'tensorflow', 'torch',  # Complex ML frameworks
        }

    def extract_api_calls(self, code: str) -> list[str]:
        """Extract API invocations from code."""
        # Simple regex-based extraction
        # In production, use AST parsing
        pattern = r'(\w+(?:\.\w+)*)\s*\('
        return re.findall(pattern, code)

    def should_retrieve_docs(
        self,
        api: str,
        confidence: Optional[float] = None
    ) -> tuple[bool, str]:
        """
        Decide whether to retrieve documentation for an API.
        Returns (should_retrieve, reason).
        """
        # Always retrieve for cloud APIs
        if any(api.startswith(prefix) for prefix in self.always_retrieve):
            return True, f"Cloud/complex API: {api}"

        # Skip for well-known APIs
        if api in self.known_apis:
            if confidence is None or confidence > self.threshold:
                return False, f"Common API with high confidence: {api}"

        # Check confidence
        if confidence is not None:
            if confidence < self.threshold:
                return True, f"Low confidence ({confidence:.2%}): {api}"
            else:
                return False, f"High confidence ({confidence:.2%}): {api}"

        # Default: retrieve if uncertain
        return True, f"Unknown API, no confidence info: {api}"

    def analyze_code(
        self,
        code: str,
        logprobs: Optional[dict[str, list[float]]] = None
    ) -> dict:
        """Analyze code and recommend documentation retrieval."""
        apis = self.extract_api_calls(code)

        analysis = {
            'total_apis': len(apis),
            'retrieve': [],
            'skip': [],
        }

        for api in apis:
            # Get confidence if available
            confidence = None
            if logprobs and api in logprobs:
                confidence = np.exp(np.mean(logprobs[api]))

            should_retrieve, reason = self.should_retrieve_docs(api, confidence)

            if should_retrieve:
                analysis['retrieve'].append({'api': api, 'reason': reason})
            else:
                analysis['skip'].append({'api': api, 'reason': reason})

        return analysis

# Usage
checker = APIConfidenceChecker()

code = """
import boto3
import json

s3 = boto3.client('s3')
s3.create_bucket(Bucket='my-bucket', Encryption='AES256')
data = json.dumps({'key': 'value'})
"""

analysis = checker.analyze_code(code)
print(f"APIs needing docs: {len(analysis['retrieve'])}")
for item in analysis['retrieve']:
    print(f"  - {item['api']}: {item['reason']}")
```

### Example 2: Documentation Retriever

```python
from typing import Optional
import requests

class DocumentationRetriever:
    """
    Retrieve API documentation from various sources.
    """

    def __init__(self):
        self.sources = {
            'boto3': 'https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/{service}.html',
            'azure': 'https://learn.microsoft.com/en-us/python/api/{package}',
            'python': 'https://docs.python.org/3/library/{module}.html',
        }

    def retrieve(self, api: str) -> Optional[str]:
        """
        Retrieve documentation for an API.
        Returns formatted documentation string.
        """
        # Determine source
        if api.startswith('boto3'):
            return self._retrieve_boto3(api)
        elif api.startswith('azure'):
            return self._retrieve_azure(api)
        else:
            return self._retrieve_python_stdlib(api)

    def _retrieve_boto3(self, api: str) -> str:
        """Retrieve AWS SDK documentation."""
        # Parse: boto3.client('s3').create_bucket
        # Extract: service=s3, method=create_bucket

        # In production, use actual AWS docs or cached index
        # For now, return mock documentation
        return f"""
AWS SDK Documentation for {api}

Parameters:
  Bucket (str): The name of the bucket to create
  CreateBucketConfiguration (dict): Configuration for bucket creation
    LocationConstraint (str): Region for the bucket
  ACL (str): Canned ACL to apply (e.g., 'private', 'public-read')

Returns:
  dict: Response with Location and other metadata

Example:
  s3.create_bucket(
      Bucket='my-bucket',
      CreateBucketConfiguration={{
          'LocationConstraint': 'us-west-2'
      }}
  )
"""

    def _retrieve_azure(self, api: str) -> str:
        """Retrieve Azure SDK documentation."""
        return f"[Azure docs for {api}]"

    def _retrieve_python_stdlib(self, api: str) -> str:
        """Retrieve Python standard library documentation."""
        return f"[Python stdlib docs for {api}]"

# Usage
retriever = DocumentationRetriever()
docs = retriever.retrieve('boto3.client.s3.create_bucket')
print(docs)
```

### Example 3: DAG++ Implementation

```python
class DAGPlusPlus:
    """
    Documentation Augmented Generation with smart retrieval.
    Implements the DAG++ algorithm from the paper.
    """

    def __init__(self, llm, retriever, checker):
        self.llm = llm
        self.retriever = retriever
        self.checker = checker

    def generate(self, task: str) -> str:
        """
        Generate code with selective documentation augmentation.
        """
        # Step 1: Initial zero-shot generation
        initial_response = self.llm.generate(task)
        code = initial_response['code']
        logprobs = initial_response.get('logprobs', {})

        # Step 2: Analyze API confidence
        analysis = self.checker.analyze_code(code, logprobs)

        # Step 3: Retrieve docs only for low-confidence APIs
        if not analysis['retrieve']:
            # No docs needed, return initial attempt
            return code

        # Step 4: Retrieve documentation
        docs = []
        for item in analysis['retrieve']:
            api = item['api']
            doc = self.retriever.retrieve(api)
            if doc:
                docs.append(doc)

        # Step 5: Re-generate with documentation
        augmented_prompt = self._create_augmented_prompt(task, docs)
        final_response = self.llm.generate(augmented_prompt)

        return final_response['code']

    def _create_augmented_prompt(
        self,
        task: str,
        docs: list[str]
    ) -> str:
        """Create prompt with task and retrieved documentation."""
        docs_section = "\n\n".join([
            f"Documentation {i+1}:\n{doc}"
            for i, doc in enumerate(docs)
        ])

        return f"""Task: {task}

Relevant API Documentation:
{docs_section}

Generate code to complete the task using the provided documentation.
"""

# Usage
dag = DAGPlusPlus(llm, retriever, checker)

task = "Create an S3 bucket with server-side encryption in us-west-2"
code = dag.generate(task)
print(code)
```

### Example 4: Known API Index

```python
import json
from pathlib import Path

class KnownAPIIndex:
    """
    Maintain an index of known APIs to avoid unnecessary retrieval.
    """

    def __init__(self, index_path: Path):
        self.index_path = index_path
        self.index = self._load_index()

    def _load_index(self) -> dict:
        """Load API frequency index."""
        if self.index_path.exists():
            with open(self.index_path) as f:
                return json.load(f)
        return {}

    def get_frequency(self, api: str) -> str:
        """
        Get frequency category for an API.
        Returns 'high', 'medium', 'low', or 'unknown'.
        """
        if api not in self.index:
            return 'unknown'

        count = self.index[api]
        if count >= 101:
            return 'high'
        elif count >= 11:
            return 'medium'
        else:
            return 'low'

    def is_common(self, api: str) -> bool:
        """Check if API is high-frequency (common)."""
        return self.get_frequency(api) == 'high'

    def add_api(self, api: str, frequency: int):
        """Add API to index."""
        self.index[api] = frequency
        self._save_index()

    def _save_index(self):
        """Persist index to disk."""
        with open(self.index_path, 'w') as f:
            json.dump(self.index, f, indent=2)

# Build initial index from common packages
def build_common_api_index() -> KnownAPIIndex:
    """Create index of commonly-used APIs."""
    index = KnownAPIIndex(Path('api_index.json'))

    # Standard library (high frequency)
    stdlib_apis = [
        'os.path.join', 'os.path.exists', 'os.makedirs',
        'json.loads', 'json.dumps',
        'datetime.datetime.now', 'datetime.timedelta',
        'sys.argv', 'sys.exit',
        're.match', 're.search', 're.findall',
    ]
    for api in stdlib_apis:
        index.add_api(api, 1000)  # High frequency

    # Common packages (high frequency)
    common_packages = [
        'requests.get', 'requests.post',
        'pandas.DataFrame', 'pandas.read_csv',
        'numpy.array', 'numpy.zeros',
    ]
    for api in common_packages:
        index.add_api(api, 500)

    return index
```

---

## Practical Recommendations

### Documentation Retrieval Strategy

**For K&V agents:**

1. **Pre-task analysis**
   - Scan bead description for cloud/domain-specific APIs
   - Pre-fetch docs for known complex APIs
   - Cache docs for phase duration

2. **During execution**
   - Monitor API confidence in real-time
   - Retrieve docs on first low-confidence invocation
   - Reuse retrieved docs for related APIs

3. **During repair**
   - Always retrieve docs for failing API calls
   - Include error message + docs in retry context
   - Track which docs resolved which errors

### Cost-Benefit Analysis

| Retrieval Strategy | Accuracy | Context Cost | Latency |
|-------------------|----------|--------------|---------|
| Never retrieve | 69.5% | Low | Fast |
| Always retrieve (DAG) | 71.8% | High | Slow |
| **Smart retrieve (DAG++)** | **77.7%** | **Medium** | **Medium** |

---

## Key Takeaways

1. **Low-frequency APIs are the problem** - Common APIs work fine (88-93%), but rare APIs fail catastrophically (24-41%)

2. **Documentation retrieval helps rare APIs** - Improves low-frequency API correctness from 42% to 87% (+45 points)

3. **Over-retrieval hurts performance** - Adding docs for known APIs degrades accuracy from 89% to 50% (-39 points)

4. **Confidence is well-calibrated** - Model probability scores accurately indicate when to trust/distrust API invocations

5. **Selective retrieval wins** - DAG++ achieves +8% overall by only retrieving when needed

6. **Cloud APIs need special handling** - AWS/Azure/GCP SDKs are low-frequency and benefit most from docs

7. **Standard library doesn't need docs** - Skip retrieval for os, json, datetime, etc.

---

## See Also

- `006-dark-side-self-correction.md` - Over-documentation similar to intrinsic self-correction
- `057-anthropic-context-engineering.md` - Minimal context principle
- `004-context-length-hurts.md` - Why unnecessary docs degrade performance
- `007-swe-gym.md` - Test-based validation of API correctness
