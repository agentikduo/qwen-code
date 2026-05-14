# Agent System

Qwen Code implements a sophisticated multi-agent infrastructure that supports concurrent agent execution across different display backends.

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────┐
│                  AgentManager                         │
│  (subagent-manager.ts — spawns & manages agents)     │
└──────────────────────┬───────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  ┌──────────┐  ┌────────────┐  ┌──────────┐
  │ Headless │  │ Interactive│  │ Forked   │
  │ Agent    │  │ Agent      │  │ Agent    │
  └────┬─────┘  └─────┬──────┘  └────┬─────┘
       │              │              │
       ▼              ▼              ▼
  ┌──────────────────────────────────────┐
  │         Backend (Display)            │
  │  ┌──────────┐ ┌──────────┐ ┌──────┐ │
  │  │  tmux    │ │  iTerm2  │ │ In-  │ │
  │  │  Backend │ │  Backend │ │Proc  │ │
  │  └──────────┘ └──────────┘ └──────┘ │
  └──────────────────────────────────────┘
```

---

## Components

### 1. Agent Manager (`core/src/subagents/subagent-manager.ts`)

Central orchestrator that:
- Manages agent lifecycle (spawn, monitor, resume)
- Handles model selection for subagents
- Validates agent configurations
- Provides override mechanism for custom agents

### 2. Agent Types (`core/src/subagents/types.ts`)

| Type | Description |
|---|---|
| **SubAgent** | A secondary agent spawned for a specific subtask |
| **Built-in Agent** | Predefined agents (explore, debug, review, etc.) |
| **Forked Agent** | A child process running a separate Qwen Code instance |
| **Background Agent** | Long-running agents for background tasks |

### 3. Agent Runtime (`core/src/agents/runtime/`)

| File | Purpose |
|---|---|
| `agent-core.ts` | Core agent logic — message handling, tool execution |
| `agent-context.ts` | Agent execution context (config, services, state) |
| `agent-types.ts` | Type definitions for agent configuration |
| `agent-headless.ts` | Headless agent (no terminal display) |
| `agent-interactive.ts` | Interactive agent (with terminal display) |
| `agent-events.ts` | Event definitions for agent lifecycle |
| `agent-statistics.ts` | Performance statistics collection |

### 4. Arena Mode (`core/src/agents/arena/`)

**Purpose:** Multi-agent collaboration where multiple AI agents work concurrently on different aspects of a problem.

| File | Purpose |
|---|---|
| `ArenaManager.ts` | Manages multiple agents in arena mode |
| `ArenaAgentClient.ts` | Client for communicating with arena agents |
| `arena-events.ts` | Event types for arena coordination |
| `diff-summary.ts` | Generates diff summaries from arena results |
| `types.ts` | Arena-specific type definitions |

### 5. Backends (`core/src/agents/backends/`)

Display backends provide the visual/multiplexer layer for agent subprocesses:

| Backend | File | Platform |
|---|---|---|
| **tmux** | `TmuxBackend.ts` | Unix (tmux required) |
| **iTerm2** | `ITermBackend.ts` | macOS (iTerm2 required) |
| **InProcess** | `InProcessBackend.ts` | All platforms (no external dep) |

All implement a common interface from `types.ts`:
- `spawn()` — Launch agent in backend
- `attach()` — Attach to existing session
- `detect()` — Auto-detect available backends

### 6. Agent Tool (`core/src/tools/agent/`)

The `AgentTool` allows the LLM to spawn sub-agents during a conversation:

| File | Purpose |
|---|---|
| `agent.ts` | Agent tool implementation |
| `agent-override.ts` | Override mechanism for agent behavior |
| `fork-subagent.ts` | Fork subagent into child process |

---

## Built-in Agents (`core/src/subagents/builtin-agents.ts`)

| Agent | Purpose |
|---|---|
| **Explore** | Codebase exploration and research |
| **Debug** | Hypothesis-driven debugging |
| **Review** | Code review and quality checks |
| **Test Engineer** | Bug reproduction and verification |
| **General Purpose** | General-purpose subagent tasks |

---

## Agent Lifecycle

```
1. Spawn
   ├── Resolve model config for subagent
   ├── Create agent context (Config, services, state)
   └── Select backend (tmux / iTerm2 / InProcess)

2. Run
   ├── Execute conversation loop (headless or interactive)
   ├── Process tool calls
   └── Handle streaming responses

3. Complete
   ├── Collect results
   ├── Calculate statistics
   └── Return to parent agent

4. Resume (Optional)
   ├── Restore agent state
   ├── Reattach to backend display
   └── Continue execution
```

---

## Background Tasks (`core/src/agents/background-tasks.ts`)

- Long-running agents that operate independently of the main conversation
- Managed by `BackgroundShellRegistry` service
- Support for persistent monitoring and cron-style scheduling
- Integration with the Cron scheduler service

---

## Forked Agents (`core/src/utils/forkedAgent.ts`)

- Spawns a separate Qwen Code process for isolated execution
- Communication via JSON over stdin/stdout
- Supports sandbox execution (Docker/Podman)
- Used for tasks requiring process isolation
