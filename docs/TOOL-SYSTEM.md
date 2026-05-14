# Tool System

Qwen Code implements a comprehensive tool system that enables the LLM to perform actions on the user's behalf. This document describes the architecture, available tools, and how they integrate with the LLM.

---

## Architecture

```
LLM sends
function call
     │
     ▼
┌─────────────────────────────────────┐
│  ToolRegistry (tool-registry.ts)    │
│  - Maps tool names → Tool instances │
│  - Validates parameters             │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  CoreToolScheduler                  │
│  (coreToolScheduler.ts)             │
│  - Schedules & orders tool calls    │
│  - Manages execution queue          │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  PermissionFlow (permissionFlow.ts) │
│  - Checks user approval mode        │
│  - Prompts for confirmation         │
│  - Enforces shell semantics         │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  Hook System (hooks/)               │
│  - PreToolUse hooks                 │
│  - PostToolUse hooks                │
│  - Notification hooks               │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  Tool Implementation (tools/)       │
│  - Executes the action              │
│  - Returns result to LLM            │
└─────────────────────────────────────┘
```

## Available Tools

All tools are defined in `packages/core/src/tools/`. Some are eagerly loaded (exported from `tools.ts`), others are lazy-loaded for performance.

### File System Tools

| Tool | File | Description |
|---|---|---|
| `Read` | `read-file.ts` | Read files with optional line range |
| `Write` | `write-file.ts` | Write/create files |
| `Edit` | `edit.ts` | Search-and-replace text edits |
| `Glob` | `glob.ts` | Pattern-based file lookup |
| `LS` | `ls.ts` | Directory listing |
| `Grep` | `grep.ts` | Full-text search (ripgrep) |

### Shell & Process Tools

| Tool | File | Description |
|---|---|---|
| `Shell` | `shell.ts` | Execute shell commands |
| `Monitor` | `monitor.ts` | Long-running process output monitor |
| `TaskStop` | `task-stop.ts` | Stop a background task |
| `SendMessage` | `send-message.ts` | Send message to background task |

### Agent Tools

| Tool | File | Description |
|---|---|---|
| `Agent` | `agent/agent.ts` | Spawn a subagent |
| `ForkSubAgent` | `agent/fork-subagent.ts` | Fork subagent in child process |

### Code Analysis Tools

| Tool | File | Description |
|---|---|---|
| `LSP` | `lsp.ts` | Language Server Protocol operations |
| `RipGrep` | `ripGrep.ts` | Advanced code search |

### MCP Tools

| Tool | File | Description |
|---|---|---|
| `MCPClient` | `mcp-client.ts` | MCP tool client |
| `MCPClientManager` | `mcp-client-manager.ts` | MCP client lifecycle manager |

### Cron Tools

| Tool | File | Description |
|---|---|---|
| `CronCreate` | `cron-create.ts` | Create a cron job |
| `CronList` | `cron-list.ts` | List cron jobs |
| `CronDelete` | `cron-delete.ts` | Delete a cron job |

### Web Tools

| Tool | File | Description |
|---|---|---|
| `WebFetch` | `web-fetch.ts` | Fetch URL content |

### Interaction Tools

| Tool | File | Description |
|---|---|---|
| `AskUserQuestion` | `askUserQuestion.ts` | Prompt user with a question |
| `TodoWrite` | `todoWrite.ts` | Create/manage task todos |
| `ExitPlanMode` | `exitPlanMode.ts` | Exit plan mode and request approval |
| `Skill` | `skill.ts` | Execute a skill |
| `ToolSearch` | `tool-search.ts` | Search available tools |

### Output Tools

| Tool | File | Description |
|---|---|---|
| `SyntheticOutput` | `syntheticOutput.ts` | Structured output generation |

### Prior-Read Enforcement

| Module | Description |
|---|---|
| `priorReadEnforcement.ts` | Ensures files are read before being edited |

---

## Tool Registration

Tools are registered in `tool-registry.ts`. The registry:

1. Maps canonical tool names (`ToolNames`) to tool implementations
2. Validates tool call parameters against schemas
3. Provides tool discovery for the LLM via tool-search

## Permission System (`core/src/permissions/`)

All tool executions pass through the permission system:

| Component | Description |
|---|---|
| `permission-manager.ts` | Central permission decision making |
| `rule-parser.ts` | Parses permission rules from config |
| `shell-semantics.ts` | Analyzes shell commands for safety |
| `types.ts` | Permission types (allow, deny, ask, etc.) |

### Approval Modes

| Mode | Behavior |
|---|---|
| **Always ask** | Prompt user for every tool call |
| **Auto-approve** | Automatically approve known-safe operations |
| **Auto-deny** | Deny without asking |

## Hook System Integration

The tool system integrates with the hook system at three points:

1. **PreToolUse hooks** — Called before tool execution. Can modify, approve, or deny.
2. **PostToolUse hooks** — Called after successful tool execution.
3. **PostToolUseFailure hooks** — Called after tool execution failure.

## Tool Call Flow (Detailed)

```
1. LLM sends a function_call with tool name + args
2. Client (client.ts) receives and parses the call
3. CoreToolScheduler schedules the execution
4. PermissionFlow checks approval mode
   ├── Auto-approve: skip to step 5
   ├── Ask: prompt user for confirmation
   └── Auto-deny: return error to LLM
5. PreToolUse hooks fire
   ├── SSRF guard checks URL tools
   ├── HTTP hooks log the action
   └── Custom hooks run
6. Tool executes the action
   ├── File operations: file system
   ├── Shell commands: shell execution service
   └── Web requests: undici HTTP client
7. PostToolUse hooks fire
8. Result is formatted and sent back to LLM
```
