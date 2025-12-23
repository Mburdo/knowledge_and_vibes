# SWE-bench Multimodal: Visual Software Engineering Tasks

**Paper:** SWE-bench Multimodal: Does Your Agent Write Better Code When It Can See?
**URL:** https://arxiv.org/abs/2410.03859
**Date:** October 2024
**Venue:** arXiv preprint

---

## Summary

Extension of SWE-bench with **617 multimodal instances** requiring image understanding for successful bug fixes. Images include screenshots, diagrams, plots, and visual artifacts. Models with vision outperform text-only by **12% absolute** (31% vs 19%).

**Key innovation:** Many real-world bugs require visual understanding—text-only agents are fundamentally limited. Vision isn't just nice-to-have, it's necessary.

---

## The Visual Gap in Software Engineering

### Why Images Matter for Debugging

```
┌─────────────────────────────────────────────────────────────────┐
│              TEXT-ONLY vs VISION-CAPABLE DEBUGGING               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TASK: "Chart labels are cut off at bottom"                      │
│                                                                  │
│  TEXT-ONLY AGENT:                                                │
│  ├── Reads issue description                                    │
│  ├── Searches for "chart" in code                               │
│  ├── Finds matplotlib code                                       │
│  ├── Guesses at margin settings                                 │
│  └── Result: Random changes, likely wrong                       │
│                                                                  │
│  VISION-CAPABLE AGENT:                                           │
│  ├── Reads issue description                                    │
│  ├── SEES screenshot of cut-off labels                          │
│  ├── Identifies exact problem visually                          │
│  ├── Searches for relevant layout code                          │
│  └── Result: Targeted fix to bottom margin                      │
│                                                                  │
│  Difference: Vision provides ground truth signal                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Types of Visual Information

| Image Type | Example Bug | Why Vision Helps |
|------------|-------------|------------------|
| **Screenshots** | UI layout issue | See exact rendering problem |
| **Error dialogs** | Exception popup | Visual stack trace context |
| **Plots/charts** | Incorrect axis labels | Compare expected vs actual |
| **Diagrams** | Architecture mismatch | Understand intended design |
| **Before/after** | Visual regression | Diff visual output |
| **Mockups** | Design spec | See target appearance |

---

## Dataset Construction

### Mining Multimodal Issues

```
┌─────────────────────────────────────────────────────────────────┐
│           MULTIMODAL INSTANCE COLLECTION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  START: SWE-bench base (2,294 instances)                         │
│      │                                                           │
│      ↓                                                           │
│  FILTER 1: Has images in issue/comments                          │
│  ├── GitHub API: Check for image attachments                    │
│  ├── Parse markdown for image URLs                              │
│  └── Result: 892 instances with images                          │
│      │                                                           │
│      ↓                                                           │
│  FILTER 2: Image is relevant to bug                              │
│  ├── LLM judges if image shows bug                              │
│  ├── Manual review of sample                                    │
│  └── Result: 617 multimodal instances                           │
│      │                                                           │
│      ↓                                                           │
│  ANNOTATION: Classify image types                                │
│  ├── Screenshot: 42%                                             │
│  ├── Plot/chart: 28%                                             │
│  ├── Diagram: 18%                                                │
│  ├── Error message: 8%                                           │
│  └── Other: 4%                                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example Multimodal Instance

```markdown
## Instance: matplotlib-12345

**Issue Title:** Chart x-axis labels overlapping

**Issue Description:**
When plotting with many data points, x-axis labels overlap
making them unreadable. See screenshot.

**Image:** [screenshot showing overlapping x-labels]

**Buggy Code:**
```python
plt.plot(x_values, y_values)
plt.xticks(x_values)  # Bug: too many ticks
plt.show()
```

**Fix:**
```python
plt.plot(x_values, y_values)
plt.xticks(x_values[::10])  # Show every 10th label
plt.show()
```

**Why Vision Helps:**
- Text alone: "overlapping" is vague
- Screenshot: Shows exact spacing problem
- Agent can count labels and calculate spacing
```

---

## Vision-Augmented Agent Architecture

### Multimodal Processing Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│            VISION-AUGMENTED SWE AGENT                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INPUT: Issue + Images + Codebase                                │
│      │                                                           │
│      ↓                                                           │
│  ┌────────────────────────────────────────────────┐             │
│  │  STEP 1: Image Understanding                   │             │
│  │  ├── Extract images from issue                 │             │
│  │  ├── Pass to vision model                      │             │
│  │  ├── Generate visual descriptions              │             │
│  │  └── Identify bug manifestation                │             │
│  └────────────────────────────────────────────────┘             │
│      │                                                           │
│      ↓                                                           │
│  ┌────────────────────────────────────────────────┐             │
│  │  STEP 2: Visual + Textual Reasoning            │             │
│  │  ├── Combine image understanding + issue text  │             │
│  │  ├── Form hypothesis about bug location        │             │
│  │  └── Plan investigation strategy                │             │
│  └────────────────────────────────────────────────┘             │
│      │                                                           │
│      ↓                                                           │
│  ┌────────────────────────────────────────────────┐             │
│  │  STEP 3: Code Search with Visual Grounding     │             │
│  │  ├── Search for code related to visual issue   │             │
│  │  ├── Example: "chart layout" if screenshot     │             │
│  │  └── Focus on relevant subsystems               │             │
│  └────────────────────────────────────────────────┘             │
│      │                                                           │
│      ↓                                                           │
│  ┌────────────────────────────────────────────────┐             │
│  │  STEP 4: Patch Generation                      │             │
│  │  ├── Generate fix based on visual + code       │             │
│  │  ├── May reference specific visual elements    │             │
│  │  └── Apply patch                                │             │
│  └────────────────────────────────────────────────┘             │
│      │                                                           │
│      ↓                                                           │
│  ┌────────────────────────────────────────────────┐             │
│  │  STEP 5: Visual Validation (if possible)       │             │
│  │  ├── Run code to regenerate visual output      │             │
│  │  ├── Compare to expected (if provided)         │             │
│  │  └── Verify fix addresses visual issue         │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Reported Results

### Performance by Modality

```
SWE-bench Multimodal Results:

Text-Only Models:
├── GPT-4 (text): 19% (119/617)
├── Claude-3 (text): 17% (105/617)
└── Gemini (text): 16% (99/617)

Vision-Capable Models:
├── GPT-4V: 31% (191/617) [+12 points]
├── Claude-3 Opus: 29% (179/617) [+12 points]
└── Gemini Pro Vision: 27% (167/617) [+11 points]

Key finding: Vision provides ~60% relative improvement
```

### Performance by Image Type

| Image Type | Text-Only | With Vision | Improvement |
|------------|-----------|-------------|-------------|
| **Screenshots** | 15% | 34% | +19 points |
| **Plots/charts** | 18% | 32% | +14 points |
| **Diagrams** | 22% | 30% | +8 points |
| **Error messages** | 25% | 30% | +5 points |

**Insight:** Screenshots benefit most from vision (harder to describe in text).

---

## Why Vision Helps: Case Studies

### Case 1: Layout Bug

```python
# ISSUE: "Button text is cut off on mobile"
# IMAGE: Screenshot showing "Subm..." instead of "Submit"

# Text-only approach:
# Agent guesses at font size, padding, width...
# Many possible causes, unclear which to fix

# Vision approach:
# Sees exact clipping point
# Measures visible vs required width
# Identifies padding constraint

# Fix (guided by visual evidence):
button.style.minWidth = '120px'  # Was 80px
```

### Case 2: Data Visualization

```python
# ISSUE: "Bar chart colors don't match legend"
# IMAGE: Chart with red bars but legend says blue

# Text-only approach:
# Searches for "color" in code
# Finds multiple color specifications
# Unclear which is wrong

# Vision approach:
# Compares chart vs legend visually
# Identifies mismatch: bars use 'primary' color
# Legend specifies 'secondary'

# Fix:
plt.bar(x, y, color='secondary')  # Was 'primary'
```

### Case 3: Regression Identification

```python
# ISSUE: "Plot looks different after update"
# IMAGES: Before (correct) and After (wrong) screenshots

# Text-only approach:
# No way to know what "different" means
# Could be anything: colors, scale, labels, data...

# Vision approach:
# Diff images visually
# Identifies axis scale changed from linear to log
# Searches for scale setting in recent commits

# Fix:
ax.set_yscale('linear')  # Was accidentally set to 'log'
```

---

## Mathematical Model

### Vision Signal Strength

```
Let V = quality of visual information
Let T = quality of textual description
Let P = performance

Text-only: P_text = f(T)
With vision: P_vision = f(T + V)

Empirical observations:
  If V >> T (visual bug): Improvement large (~60%)
  If V ≈ T (describable): Improvement medium (~30%)
  If V << T (logic bug): Improvement small (~5%)

Average improvement:
  ΔP = P_vision - P_text ≈ 12 points (60% relative)
```

---

## Integration with K&V Workflow

### Multimodal Evidence in Calibration

SWE-bench Multimodal validates K&V principle: evidence comes in many forms.

| K&V Pattern | Multimodal Evidence |
|-------------|---------------------|
| Rich evidence > sparse | Screenshots >> vague descriptions |
| Show, don't tell | Visual bugs need visual evidence |
| Grounding in reality | Images are ground truth, not interpretation |
| Multiple validation signals | Visual + test + code review |

### Bead Template with Visual Evidence

```markdown
## Visual Bug Bead

**Bead ID:** bd-visual-001
**Type:** UI regression

**Issue:** Chart labels cut off
**Visual Evidence:**
![screenshot-before.png](artifacts/screenshot-before.png)
- X-axis labels: "Jan", "Feb", "Ma..."  (cut off)
- Bottom margin: ~10px (too small)

**Hypothesis:**
Bottom margin insufficient for label height

**Investigation:**
```python
# Found in chart.py:
fig.tight_layout(pad=1.0)  # Current setting
```

**Fix:**
```python
fig.tight_layout(pad=2.5)  # Increased padding
```

**Validation:**
![screenshot-after.png](artifacts/screenshot-after.png)
- All labels visible: "Jan", "Feb", "Mar", "Apr"...
- Bottom margin: ~25px
- Tests pass

**Evidence Chain:**
1. Visual: Screenshot shows exact problem
2. Code: Identified tight_layout setting
3. Visual: After screenshot confirms fix
4. Tests: All assertions pass
```

---

## Critical Caveats

### What Multimodal Doesn't Solve

1. **Not All Bugs Are Visual**
   - Logic errors: No visual manifestation
   - Performance issues: Not visible
   - Security bugs: Often invisible
   - Coverage: ~27% of SWE-bench is multimodal

2. **Vision Models Have Limits**
   - Small text hard to read
   - Complex diagrams may confuse
   - Color perception varies
   - Resolution affects quality

3. **Visual Ground Truth is Expensive**
   - Screenshots need manual capture
   - Expected outputs hard to specify
   - Visual diffs are subjective
   - Validation more complex

4. **Increased Cost and Latency**
   - Vision models slower than text
   - Higher API costs
   - More context to process
   - May hit token limits faster

---

## Practical Implications

### When to Use Vision-Capable Agents

```
Use vision when:
✓ Issue includes screenshots
✓ UI/visualization bugs
✓ Visual regression reports
✓ Design spec provided
✓ "See image" in issue description

Text-only may suffice when:
○ Pure logic bugs
○ API/backend issues
○ Test failure (text output)
○ Performance problems
○ No visual artifacts
```

### Designing Visual Prompts

```python
VISUAL_PROMPT_TEMPLATE = """
## Issue Description
{text_description}

## Visual Evidence
{image_descriptions}

Image 1: [Screenshot of bug]
Analysis:
- What I see: {visual_observation}
- Expected: {expected_appearance}
- Difference: {visual_diff}

## Investigation Plan
Based on visual evidence:
1. Search for code controlling: {visual_element}
2. Check: {hypothesis}
3. Test fix by: {validation_method}
"""
```

---

## Research Methodology

### Instance Selection Criteria

```
Multimodal Inclusion Rules:
├── Issue contains ≥1 image
├── Image shows bug manifestation
├── Image is essential (not decorative)
├── Fix addresses visual issue
└── Tests validate visual correctness

Exclusion Rules:
├── Image is logo/branding
├── Image is documentation/tutorial
├── Bug could be described textually
└── Image added post-fix (not original)
```

### Vision Model Evaluation

```python
def evaluate_multimodal(agent, instance):
    """
    Evaluate vision-capable agent on multimodal task.
    """
    # Extract images from issue
    images = instance.images

    # Text-only baseline
    text_only_result = agent.solve(
        issue_text=instance.description,
        images=None
    )

    # With vision
    with_vision_result = agent.solve(
        issue_text=instance.description,
        images=images
    )

    return {
        "text_only_passed": text_only_result.tests_passed,
        "with_vision_passed": with_vision_result.tests_passed,
        "vision_helped": (
            with_vision_result.tests_passed and
            not text_only_result.tests_passed
        )
    }
```

---

## Key Metrics to Track

When evaluating multimodal agents:

```json
{
  "task_id": "matplotlib-12345",
  "modality": "multimodal",
  "image_types": ["screenshot", "expected_output"],
  "image_count": 2,
  "evaluation": {
    "text_only": {
      "resolved": false,
      "tests_passed": 2,
      "tests_failed": 5
    },
    "with_vision": {
      "resolved": true,
      "tests_passed": 7,
      "tests_failed": 0,
      "vision_observations": [
        "X-axis labels overlap",
        "Bottom margin too small",
        "Expected spacing ~15px, actual ~5px"
      ]
    }
  },
  "improvement": {
    "vision_helped": true,
    "delta_tests": 5,
    "explanation": "Vision enabled precise spacing measurement"
  }
}
```

---

## Key Takeaways

1. **Vision provides 60% relative improvement** — From 19% to 31% on multimodal tasks
2. **Screenshots most critical** — Hardest to describe, biggest benefit from vision
3. **~27% of real bugs are visual** — Text-only agents fundamentally limited
4. **Visual evidence is ground truth** — More reliable than textual descriptions
5. **Multimodal is the future** — Software engineering is inherently multimodal

---

## See Also

- `065-confucius-code-agent.md` — Multi-turn agents benefit from visual context
- `022-chatrepair.md` — Feedback loops enhanced by visual validation
- `060-debugging-decay-index.md` — Visual evidence reduces iteration needs
- `021-swe-bench-plus.md` — Quality control applies to visual evidence too
- `029-swe-rebench.md` — Standardized evaluation includes multimodal protocol
