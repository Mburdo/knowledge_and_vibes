# SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering

**Paper:** SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering
**URL:** https://arxiv.org/abs/2405.15793
**Date:** May 2024
**Venue:** NeurIPS 2024

---

## Summary

SWE-agent introduces the concept of **Agent-Computer Interfaces (ACIs)** - custom-designed interfaces that dramatically improve LLM agent performance on software engineering tasks. The paper's key insight is that agent effectiveness isn't just about model intelligence; it's equally about the **ergonomics of the tools** the agent uses.

**Key result:** With a well-designed ACI, SWE-agent achieves **12.5% on SWE-bench** using GPT-4, compared to 2.8% with generic terminal access - a **347% improvement from interface design alone**.

---

## The Core Thesis

### Agents Are Computer Users

```
┌─────────────────────────────────────────────────────────────────┐
│                  HUMANS vs AGENTS AS USERS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Humans benefit from:                                            │
│  ├── IDEs (VS Code, IntelliJ)                                    │
│  ├── GUIs (file browsers, debuggers)                             │
│  ├── Syntax highlighting                                         │
│  ├── Auto-complete                                               │
│  └── Specialized tools for tasks                                 │
│                                                                  │
│  Why? Tool ergonomics matter!                                    │
│                                                                  │
│  Agents historically get:                                        │
│  ├── Raw terminal access                                         │
│  ├── Noisy, unstructured output                                  │
│  ├── Generic commands (cat, grep, vim)                           │
│  └── No task-specific affordances                                │
│                                                                  │
│  SWE-agent thesis: Agents deserve specialized interfaces too     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**The claim:** Interface design is a **first-class determinant** of agent performance, not just a minor optimization.

---

## What is an Agent-Computer Interface (ACI)?

### Definition

> An Agent-Computer Interface (ACI) is the set of commands, outputs, and feedback mechanisms through which an LLM agent interacts with computational systems.

**Components:**
1. **Action space** - What commands the agent can issue
2. **Observation space** - What feedback the agent receives
3. **State representation** - How the system presents context
4. **Error handling** - How failures are communicated

### Generic Terminal vs SWE-agent ACI

```
┌─────────────────────────────────────────────────────────────────┐
│                     INTERFACE COMPARISON                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Generic Terminal:                                               │
│  Agent: cat file.py                                              │
│  Output: [2000 lines of unfiltered code]                         │
│  Problem: Information overload, no structure                     │
│                                                                  │
│  SWE-agent ACI:                                                  │
│  Agent: view file.py                                             │
│  Output:                                                         │
│    [file.py (350 lines total)]                                   │
│    [Showing lines 1-50]                                          │
│    1 | import os                                                 │
│    2 | from typing import Optional                               │
│    ...                                                           │
│    [Use 'view <start>-<end>' to see more]                        │
│  Problem: Controlled, digestible information                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## SWE-agent's ACI Design

### Custom Commands

SWE-agent replaces generic Unix commands with purpose-built agent commands:

| Generic Command | SWE-agent Command | Improvement |
|----------------|-------------------|-------------|
| `cat file.py` | `view file.py` | Paginated, line-numbered, controlled |
| `grep "pattern" -r .` | `search "pattern"` | Relevance-ranked, context-aware |
| `vim file.py` | `edit file.py:10-20` | Line-range editing, validation |
| `git diff` | `show_diff` | Formatted, with context |
| `python -m pytest` | `test` | Filtered output, failure summaries |

### The `view` Command

**Purpose:** Display files in a digestible format for LLMs.

```
┌─────────────────────────────────────────────────────────────────┐
│                     view COMMAND DESIGN                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Syntax: view <file> [<start>-<end>]                             │
│                                                                  │
│  Example: view models/user.py 50-100                             │
│                                                                  │
│  Output:                                                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ [models/user.py (450 lines total)]                        │   │
│  │ [Showing lines 50-100]                                    │   │
│  │                                                           │   │
│  │  50 | class User(BaseModel):                             │   │
│  │  51 |     """User model with authentication."""          │   │
│  │  52 |     id: int                                        │   │
│  │  53 |     username: str                                  │   │
│  │  ...                                                      │   │
│  │ 100 |         return self.is_active                      │   │
│  │                                                           │   │
│  │ [50 more lines above, 350 more lines below]               │   │
│  │ [Use: view models/user.py 100-150 to see more]           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Design principles:                                              │
│  ├── Pagination prevents information overload                    │
│  ├── Line numbers enable precise references                      │
│  ├── Total line count provides context                           │
│  └── Navigation hints guide next action                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The `search` Command

**Purpose:** Find code semantically and present ranked results.

```
┌─────────────────────────────────────────────────────────────────┐
│                    search COMMAND DESIGN                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Syntax: search "pattern" [in <directory>]                       │
│                                                                  │
│  Example: search "def authenticate" in src/                      │
│                                                                  │
│  Output:                                                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Found 3 matches:                                          │   │
│  │                                                           │   │
│  │ 1. src/auth/user.py:145                                  │   │
│  │    def authenticate(self, username, password):           │   │
│  │        """Primary authentication method"""               │   │
│  │    [95% relevance]                                       │   │
│  │                                                           │   │
│  │ 2. src/auth/oauth.py:67                                  │   │
│  │    def authenticate_oauth(self, token):                  │   │
│  │        """OAuth authentication wrapper"""                │   │
│  │    [72% relevance]                                       │   │
│  │                                                           │   │
│  │ 3. tests/test_auth.py:22                                 │   │
│  │    def test_authenticate():                              │   │
│  │        """Test authentication flow"""                    │   │
│  │    [45% relevance - test file]                           │   │
│  │                                                           │   │
│  │ [Use: view <file>:<line> to inspect]                     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  vs grep output:                                                 │
│  src/auth/user.py:145:    def authenticate(self, username...    │
│  src/auth/oauth.py:67:    def authenticate_oauth(self, tok...   │
│  tests/test_auth.py:22:def test_authenticate():                 │
│  [No context, no ranking, truncated]                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The `edit` Command

**Purpose:** Make surgical code edits with validation.

```
┌─────────────────────────────────────────────────────────────────┐
│                     edit COMMAND DESIGN                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Syntax: edit <file>:<start>-<end>                               │
│          <new_content>                                           │
│          END                                                     │
│                                                                  │
│  Example:                                                        │
│  Agent: edit models/user.py:50-52                                │
│         class User(BaseModel):                                   │
│             """User model with enhanced authentication."""       │
│             id: int                                              │
│         END                                                      │
│                                                                  │
│  Output:                                                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ ✓ Edit applied successfully                               │   │
│  │                                                           │   │
│  │ Diff:                                                     │   │
│  │  50 | class User(BaseModel):                             │   │
│  │  51 |-    """User model with authentication."""          │   │
│  │  51 |+    """User model with enhanced authentication.""" │   │
│  │  52 |     id: int                                        │   │
│  │                                                           │   │
│  │ [File saved, 1 line modified]                             │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Built-in validation:                                            │
│  ├── Syntax checking (Python AST parsing)                        │
│  ├── Line range verification                                     │
│  ├── Indentation preservation                                    │
│  └── Backup creation                                             │
│                                                                  │
│  If edit fails:                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ ✗ Edit failed: SyntaxError at line 51                    │   │
│  │                                                           │   │
│  │ Expected ":" after class definition                       │   │
│  │                                                           │   │
│  │ [File unchanged, try again]                               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Results

### SWE-bench Performance

**Impact of ACI design:**

| Interface | Model | SWE-bench Pass@1 | Improvement |
|-----------|-------|-----------------|-------------|
| Raw terminal | GPT-4 | 2.8% | Baseline |
| Basic tools | GPT-4 | 6.2% | +3.4% |
| **SWE-agent ACI** | **GPT-4** | **12.5%** | **+9.7%** |

```
Performance by Interface Design

Raw terminal:    ███ 2.8%
Basic tools:     ██████ 6.2%
SWE-agent ACI:   ████████████ 12.5%

Interpretation: Interface design accounts for 9.7% absolute improvement
                (347% relative increase from baseline)
```

### HumanEvalFix Performance

On the HumanEvalFix benchmark (fixing buggy code):

| System | Pass@1 |
|--------|--------|
| GPT-4 (zero-shot) | 48.2% |
| GPT-4 (with feedback) | 67.3% |
| **SWE-agent** | **87.7%** |

**Key insight:** Well-designed interfaces enable better feedback loops, leading to higher success rates on debugging tasks.

---

## Design Principles for ACIs

### 1. Output Shaping

**Principle:** Transform raw outputs into agent-digestible formats.

**Anti-pattern (raw terminal):**
```bash
$ ls -la
total 1456
drwxr-xr-x  45 user  staff   1440 Dec 23 10:00 .
drwxr-xr-x   8 user  staff    256 Dec 22 15:30 ..
-rw-r--r--   1 user  staff   6148 Dec 23 09:45 .DS_Store
drwxr-xr-x  12 user  staff    384 Dec 23 10:00 .git
-rw-r--r--   1 user  staff    523 Dec 20 14:22 .gitignore
[... 40 more lines of noise ...]
```

**Pattern (SWE-agent):**
```
Files in current directory:
  src/          (directory - source code)
  tests/        (directory - test files)
  README.md     (documentation)
  setup.py      (package configuration)

[Use 'view <file>' to inspect files]
```

### 2. Action Affordances

**Principle:** Make next actions obvious and easy.

**Anti-pattern:**
```
Agent sees error but doesn't know how to proceed
"NameError: name 'authenticate' is not defined"
[Agent must figure out how to search codebase]
```

**Pattern:**
```
Error: NameError: name 'authenticate' is not defined
File: src/views.py, line 42

Suggested actions:
  search "def authenticate"  - Find definition
  view src/views.py 30-50    - See context around error

[SWE-agent knows exactly what to do next]
```

### 3. Bounded Complexity

**Principle:** Prevent agents from getting lost in unbounded spaces.

**Anti-pattern:**
```bash
$ cat large_file.py
[10,000 lines scroll by]
[Agent context window overwhelmed]
[Agent loses track of what it's looking for]
```

**Pattern:**
```
view large_file.py
[Showing lines 1-50 of 10,000]
...
[Use 'search "<pattern>"' to find specific code]
[Use 'view <start>-<end>' to see specific range]
```

### 4. Error Recovery

**Principle:** Make failures informative and actionable.

**Anti-pattern:**
```bash
$ python fix.py
SyntaxError: invalid syntax
[Agent doesn't know what's wrong or where]
```

**Pattern:**
```
test
✗ Tests failed

Error in src/models.py:145
SyntaxError: invalid syntax
    class User(BaseModel)
                        ^
Expected ':' after class name

[Use 'view src/models.py:140-150' to see context]
[Use 'edit src/models.py:145' to fix]
```

---

## Integration with K&V Workflow

### Tool Ergonomics for Agents

SWE-agent validates K&V's slash-command approach:

| K&V Pattern | SWE-agent Equivalent |
|-------------|---------------------|
| `/plan` | High-level command that structures thinking |
| `/test` | `test` command with filtered output |
| `/search` | `search` command with ranked results |
| Bead files | Structured task representation |

### Designing Agent-Native Tools

**Lessons for K&V tool design:**

```
┌─────────────────────────────────────────────────────────────────┐
│                   TOOL DESIGN CHECKLIST                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ✓ Pagination for large outputs                                 │
│    └── Never dump 1000+ lines unfiltered                         │
│                                                                  │
│  ✓ Structured output formats                                    │
│    └── Use tables, lists, sections - not raw text               │
│                                                                  │
│  ✓ Built-in validation                                          │
│    └── Catch errors before execution when possible              │
│                                                                  │
│  ✓ Navigation hints                                             │
│    └── Suggest next actions based on current state              │
│                                                                  │
│  ✓ Relevance ranking                                            │
│    └── Sort results by usefulness, not arbitrary order          │
│                                                                  │
│  ✓ Error messages with context                                  │
│    └── Explain what went wrong AND how to fix it                │
│                                                                  │
│  ✓ State transparency                                           │
│    └── Agent always knows where it is and what's changed        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Code Examples

### Example 1: ACI-Inspired File Viewer

```python
from pathlib import Path
from typing import Optional, Tuple

class ACIFileViewer:
    """
    File viewing with ACI principles:
    - Pagination
    - Line numbers
    - Navigation hints
    - Context awareness
    """

    def __init__(self, page_size: int = 50):
        self.page_size = page_size

    def view(
        self,
        file_path: str,
        line_range: Optional[Tuple[int, int]] = None
    ) -> str:
        """
        View file with agent-friendly formatting.
        """
        path = Path(file_path)

        if not path.exists():
            return self._error_response(f"File not found: {file_path}")

        try:
            lines = path.read_text().split('\n')
        except Exception as e:
            return self._error_response(f"Cannot read file: {e}")

        total_lines = len(lines)

        # Determine range
        if line_range:
            start, end = line_range
            start = max(1, start)
            end = min(total_lines, end)
        else:
            start, end = 1, min(self.page_size, total_lines)

        # Format output
        output = []
        output.append(f"[{file_path} ({total_lines} lines total)]")
        output.append(f"[Showing lines {start}-{end}]")
        output.append("")

        # Show lines with numbers
        for i in range(start - 1, end):
            line_num = i + 1
            line_content = lines[i]
            output.append(f"{line_num:4} | {line_content}")

        # Navigation hints
        output.append("")
        if start > 1:
            output.append(f"[{start - 1} lines above]")
        if end < total_lines:
            output.append(f"[{total_lines - end} lines below]")

        # Suggest next actions
        if total_lines > end:
            next_start = end + 1
            next_end = min(end + self.page_size, total_lines)
            output.append(f"[Use: view {file_path} {next_start}-{next_end} to see more]")

        return "\n".join(output)

    def _error_response(self, message: str) -> str:
        """Format error message with recovery hints."""
        return f"""Error: {message}

Suggested actions:
  - Check file path spelling
  - Use 'search "<pattern>"' to find the file
  - List directory contents to verify file exists
"""

# Usage
viewer = ACIFileViewer(page_size=30)

# First view
print(viewer.view("src/models/user.py"))

# Continue reading
print(viewer.view("src/models/user.py", line_range=(31, 60)))
```

### Example 2: ACI-Inspired Code Search

```python
import re
from pathlib import Path
from dataclasses import dataclass
from typing import List

@dataclass
class SearchMatch:
    file_path: str
    line_number: int
    line_content: str
    context_before: List[str]
    context_after: List[str]
    relevance_score: float

class ACICodeSearch:
    """
    Code search with ACI principles:
    - Relevance ranking
    - Context inclusion
    - Result limits
    - Action suggestions
    """

    def __init__(self, repo_path: Path):
        self.repo_path = repo_path

    def search(
        self,
        pattern: str,
        directory: Optional[str] = None,
        max_results: int = 10
    ) -> str:
        """
        Search codebase with agent-friendly output.
        """
        search_path = self.repo_path / directory if directory else self.repo_path

        matches = []

        for py_file in search_path.rglob('*.py'):
            if self._should_skip(py_file):
                continue

            file_matches = self._search_file(py_file, pattern)
            matches.extend(file_matches)

        # Sort by relevance
        matches.sort(key=lambda m: m.relevance_score, reverse=True)

        # Limit results
        matches = matches[:max_results]

        return self._format_results(pattern, matches)

    def _search_file(self, file_path: Path, pattern: str) -> List[SearchMatch]:
        """Search single file for pattern."""
        try:
            lines = file_path.read_text().split('\n')
        except:
            return []

        matches = []

        for i, line in enumerate(lines):
            if re.search(pattern, line, re.IGNORECASE):
                # Calculate relevance
                relevance = self._calculate_relevance(line, pattern, file_path)

                # Get context
                context_before = lines[max(0, i-2):i]
                context_after = lines[i+1:min(len(lines), i+3)]

                matches.append(SearchMatch(
                    file_path=str(file_path.relative_to(self.repo_path)),
                    line_number=i + 1,
                    line_content=line.strip(),
                    context_before=context_before,
                    context_after=context_after,
                    relevance_score=relevance
                ))

        return matches

    def _calculate_relevance(self, line: str, pattern: str, file_path: Path) -> float:
        """Score match relevance."""
        score = 50.0  # Base score

        # Exact match bonus
        if pattern in line:
            score += 30

        # Definition bonus
        if re.match(r'\s*(def|class)\s+', line):
            score += 20

        # Test file penalty
        if 'test' in str(file_path):
            score -= 20

        return score

    def _should_skip(self, file_path: Path) -> bool:
        """Skip irrelevant files."""
        skip_patterns = ['__pycache__', '.git', 'venv', 'node_modules']
        return any(pattern in str(file_path) for pattern in skip_patterns)

    def _format_results(self, pattern: str, matches: List[SearchMatch]) -> str:
        """Format search results for agent."""
        if not matches:
            return f"""No matches found for "{pattern}"

Suggestions:
  - Try a broader search pattern
  - Check spelling
  - Search in a different directory
"""

        output = []
        output.append(f'Found {len(matches)} matches for "{pattern}"')
        output.append("")

        for i, match in enumerate(matches, 1):
            output.append(f"{i}. {match.file_path}:{match.line_number}")
            output.append(f"   {match.line_content}")
            output.append(f"   [{match.relevance_score:.0f}% relevance]")
            output.append("")

        # Suggest next actions
        output.append("Suggested actions:")
        if matches:
            top_match = matches[0]
            output.append(f"  view {top_match.file_path}:{top_match.line_number-5}-{top_match.line_number+5}")
            output.append(f"    - Inspect top match with context")

        return "\n".join(output)

# Usage
searcher = ACICodeSearch(Path('/path/to/repo'))

result = searcher.search("def authenticate", directory="src/auth")
print(result)
```

### Example 3: ACI-Inspired Test Runner

```python
import subprocess
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class TestFailure:
    test_name: str
    file_path: str
    line_number: int
    error_message: str
    suggestion: str

class ACITestRunner:
    """
    Test runner with ACI principles:
    - Filtered output (only failures)
    - Error locations
    - Fix suggestions
    - Success summaries
    """

    def __init__(self, repo_path: str):
        self.repo_path = repo_path

    def run_tests(self, test_path: Optional[str] = None) -> str:
        """
        Run tests with agent-friendly output.
        """
        cmd = ['pytest', '-v', '--tb=short']
        if test_path:
            cmd.append(test_path)

        result = subprocess.run(
            cmd,
            cwd=self.repo_path,
            capture_output=True,
            text=True
        )

        # Parse output
        failures = self._parse_failures(result.stdout + result.stderr)

        return self._format_results(result.returncode == 0, failures)

    def _parse_failures(self, output: str) -> List[TestFailure]:
        """Extract failure information from pytest output."""
        # Simplified parsing - real implementation would be more robust
        failures = []

        lines = output.split('\n')
        current_test = None
        current_file = None
        current_line = None

        for line in lines:
            # Detect test failure
            if 'FAILED' in line:
                match = re.match(r'FAILED ([\w/]+\.py)::(test_\w+)', line)
                if match:
                    current_file = match.group(1)
                    current_test = match.group(2)

            # Detect error location
            if current_test and re.match(r'\s+File.*line \d+', line):
                match = re.search(r'line (\d+)', line)
                if match:
                    current_line = int(match.group(1))

            # Detect error message
            if current_test and (
                'AssertionError' in line or
                'Error' in line
            ):
                error_msg = line.strip()
                suggestion = self._generate_suggestion(error_msg, current_file)

                failures.append(TestFailure(
                    test_name=current_test,
                    file_path=current_file or 'unknown',
                    line_number=current_line or 0,
                    error_message=error_msg,
                    suggestion=suggestion
                ))

                current_test = None

        return failures

    def _generate_suggestion(self, error: str, file_path: Optional[str]) -> str:
        """Generate fix suggestion based on error."""
        if 'AssertionError' in error:
            return f"view {file_path} - Check assertion logic"
        elif 'NameError' in error:
            return f"search \"def <name>\" - Find missing definition"
        elif 'TypeError' in error:
            return f"view {file_path} - Check function signature"
        else:
            return f"view {file_path} - Inspect error location"

    def _format_results(self, all_passed: bool, failures: List[TestFailure]) -> str:
        """Format test results for agent."""
        if all_passed:
            return """✓ All tests passed

Test suite is green. Ready to proceed.
"""

        output = []
        output.append(f"✗ {len(failures)} test(s) failed")
        output.append("")

        for i, failure in enumerate(failures, 1):
            output.append(f"{i}. {failure.test_name}")
            output.append(f"   File: {failure.file_path}:{failure.line_number}")
            output.append(f"   Error: {failure.error_message}")
            output.append(f"   → {failure.suggestion}")
            output.append("")

        output.append("Next steps:")
        if failures:
            first_failure = failures[0]
            output.append(f"  1. {first_failure.suggestion}")
            output.append(f"  2. Fix the issue")
            output.append(f"  3. Run tests again")

        return "\n".join(output)

# Usage
runner = ACITestRunner('/path/to/repo')

result = runner.run_tests('tests/test_auth.py')
print(result)
```

---

## Key Takeaways

1. **Interface design determines performance** - Well-designed ACIs improve success rates by 347% (2.8% → 12.5%)

2. **Agents need specialized tools** - Generic terminal commands overwhelm agents with noise

3. **Output shaping is critical** - Transform raw data into agent-digestible formats

4. **Navigation hints enable autonomy** - Tell agents what actions are possible next

5. **Error recovery must be explicit** - Failures should include fix suggestions

6. **Bounded complexity prevents confusion** - Pagination and filtering prevent information overload

7. **Tool ergonomics matter as much as model quality** - Better interface beats better model

---

## See Also

- `009-swe-bench.md` - The benchmark SWE-agent targets
- `065-confucius-code-agent.md` - Scaling principles for agent systems
- `057-anthropic-context-engineering.md` - Minimal context through tool design
- `007-swe-gym.md` - Training agents with well-designed tools
