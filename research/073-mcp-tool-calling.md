---
title: Model Context Protocol (MCP) and Tool Calling 2025
tags: [mcp, tool-calling, protocol, agents, standardization, code-mode]
protocols: [P8]
---

# Model Context Protocol (MCP) and Tool Calling 2025

**Sources:**
- MCP-BENCH: Benchmarking LLM Agents on MCP Tasks (November 2025)
- "Building AI Agents with MCP" (Developer Guide 2025)
- "Code Mode: The Better Way to Use MCP" (Cloudflare, September 2025)
- Function Calling in LLM Agents (Symflower, 2025)

---

## Summary

Comprehensive 2025 overview of the **Model Context Protocol (MCP)** — Anthropic's open standard for LLM tool use. MCP is becoming the "USB-C for AI applications," standardizing how agents interact with external tools.

**Key insight:** MCP standardizes tool calling, reducing vendor lock-in and enabling portable agent architectures.

---

## What is MCP?

### The Problem MCP Solves

Before MCP:
```
Agent A + Tool X → Custom integration code A-X
Agent A + Tool Y → Custom integration code A-Y
Agent B + Tool X → Custom integration code B-X
Agent B + Tool Y → Custom integration code B-Y

N agents × M tools = N×M custom integrations
```

With MCP:
```
Agent A ──┐
Agent B ──┼── MCP Protocol ──┬── Tool X (MCP Server)
Agent C ──┘                  ├── Tool Y (MCP Server)
                             └── Tool Z (MCP Server)

Any agent works with any MCP-compliant tool
```

### MCP Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      MCP ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐                                             │
│  │    MCP HOST     │  (Environment where LLM runs)               │
│  │  ├── Claude     │                                             │
│  │  ├── Cursor     │                                             │
│  │  └── VS Code    │                                             │
│  └────────┬────────┘                                             │
│           │                                                      │
│           ▼                                                      │
│  ┌─────────────────┐                                             │
│  │   MCP CLIENT    │  (Protocol handler)                         │
│  │  ├── Tool discovery                                           │
│  │  ├── Message routing                                          │
│  │  └── Response handling                                        │
│  └────────┬────────┘                                             │
│           │  JSON-RPC                                            │
│           ▼                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  MCP SERVER 1   │  │  MCP SERVER 2   │  │  MCP SERVER N   │  │
│  │  (File System)  │  │  (Database)     │  │  (API Client)   │  │
│  │                 │  │                 │  │                 │  │
│  │  Tools:         │  │  Tools:         │  │  Tools:         │  │
│  │  - read_file    │  │  - query        │  │  - fetch_data   │  │
│  │  - write_file   │  │  - insert       │  │  - post_data    │  │
│  │  - list_dir     │  │  - update       │  │  - auth         │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## MCP-BENCH: Evaluating Tool Use

### Benchmark Details

| Metric | Value |
|--------|-------|
| MCP Servers | 28 |
| Tools exposed | 250 |
| Domains | Finance, travel, science, academic |
| Task types | Multi-step, cross-tool |

### What MCP-BENCH Tests

1. **Tool discovery** — Can agent find the right tool?
2. **Parameter control** — Are arguments correct?
3. **Cross-tool coordination** — Can agent chain tools?
4. **Planning/reasoning** — Multi-step workflows

### Results (December 2025)

| Model | MCP-BENCH Score |
|-------|-----------------|
| Claude Opus 4.5 | 72% |
| GPT-5 | 68% |
| Gemini 3 Pro | 65% |
| Open source best | 48% |

**Key finding:** Even top models struggle with complex multi-tool tasks.

---

## Code Mode: Better Tool Calling

### The Problem with Direct Tool Calling

```json
// Standard tool call (verbose, limited)
{
  "tool": "search_files",
  "parameters": {
    "query": "authentication",
    "path": "/src"
  }
}
// Result returned
// Next tool call...
// Result returned...
// Many round trips
```

### Code Mode Solution

Convert tools to TypeScript API, let LLM write code:

```typescript
// LLM writes code that calls tools
import { searchFiles, readFile, editFile } from "tools";

// Find auth files
const authFiles = await searchFiles("authentication", "/src");

// Read each one
for (const file of authFiles) {
  const content = await readFile(file.path);
  if (content.includes("validateToken")) {
    // Edit in place
    await editFile(file.path, {
      search: "validateToken",
      replace: "validateTokenV2"
    });
  }
}
```

### Why Code Mode Works

| Aspect | Direct Tool Calls | Code Mode |
|--------|------------------|-----------|
| Round trips | Many (1 per tool) | Few (batched) |
| Context usage | High | Low (98.7% reduction) |
| Complex logic | Awkward | Natural |
| LLM training data | Limited | Extensive (TypeScript) |

---

## Tool Calling Best Practices

### Tool Definition

```typescript
// Good tool definition
{
  name: "edit_file",
  description: "Edit a file by replacing text. Use for small, targeted changes.",
  parameters: {
    file_path: {
      type: "string",
      description: "Absolute path to file"
    },
    old_text: {
      type: "string",
      description: "Exact text to find (must be unique in file)"
    },
    new_text: {
      type: "string",
      description: "Text to replace with"
    }
  }
}
```

### Error Handling

```python
async def call_tool(tool_name: str, params: dict) -> ToolResult:
    try:
        result = await mcp_client.call(tool_name, params)
        return ToolResult(success=True, data=result)
    except ToolNotFoundError:
        return ToolResult(success=False, error="Tool not available")
    except ParameterError as e:
        return ToolResult(success=False, error=f"Invalid params: {e}")
    except ToolExecutionError as e:
        return ToolResult(success=False, error=f"Execution failed: {e}")
```

---

## Practical Implications

### For Knowledge & Vibes

MCP aligns with our tool architecture:

| K&V Tool | MCP Pattern |
|----------|-------------|
| Beads CLI (`bd`) | MCP Server for task tracking |
| BV Graph | MCP Server for dependencies |
| CASS | MCP Server for memory |
| Agent Mail | MCP Server for coordination |
| UBS | MCP Server for security |

### Agent Mail as MCP

```typescript
// Agent Mail tools via MCP
const agentMailServer = {
  tools: [
    "register_agent",
    "send_message",
    "fetch_inbox",
    "file_reservation_paths",
    "release_file_reservations"
  ]
};

// Usage in agent
const myName = await call("register_agent", {
  project_key: "/path/to/project",
  program: "claude-code",
  model: "opus-4.5"
});

await call("send_message", {
  sender_name: myName,
  to: ["Coordinator"],
  subject: "[CLAIMED] bd-123",
  body_md: "Starting work on authentication."
});
```

---

## Key Takeaways

1. **MCP is the standard** — Anthropic's protocol becoming universal
2. **Code Mode beats direct calls** — 98.7% context reduction
3. **Tool discovery matters** — Agents must find right tools
4. **Multi-tool is hard** — Even top models struggle at 65-72%
5. **Standardization enables portability** — Write once, use anywhere
6. **Error handling essential** — Tools fail; agents must recover

---

## See Also

- `059-multi-agent-orchestrator-2025.md` — Agent coordination
- `.claude/skills/agent-mail/SKILL.md` — Agent Mail MCP usage
- `063-agentic-se-3.md` — SE 3.0 tool requirements
