# Architecture Overview

**Project:** Qwen Code (`@qwen-code/qwen-code`)
**Version:** 0.15.11
**License:** Apache-2.0

Qwen Code is a terminal-based, AI-powered coding assistant developed by the Qwen Team at Alibaba. It provides an interactive CLI environment where users can converse with LLMs (with a focus on Qwen models, but supporting Gemini, Anthropic, OpenAI-compatible, and Vertex AI) to perform software engineering tasks — code generation, editing, file operations, shell commands, git operations, code search, and more.

The project is organized as an npm workspaces monorepo.

---

## High-Level Architecture

```
┌───────────────────────────────────────────────┐
│                  User (Terminal)               │
└──────────────────┬────────────────────────────┘
                   │ stdin/stdout
                   ▼
┌───────────────────────────────────────────────┐
│            packages/cli (Entry Point)          │
│  ┌──────────┐ ┌──────────┐ ┌───────────────┐  │
│  │  gemini   │ │nonInter- │ │    serve/      │  │
│  │  .tsx (UI)│ │ activeCli│ │   ACP Server   │  │
│  └────┬─────┘ └────┬─────┘ └───────┬───────┘  │
│       │            │               │           │
│  ┌────▼────────────▼───────────────▼────────┐ │
│  │      CLI Services Layer                  │ │
│  │  (commands, config, auth, i18n, export,  │ │
│  │   dual output, remote input, ACP)        │ │
│  └────────────────┬────────────────────────┘  │
└───────────────────┼──────────────────────────┘
                    │ depends-on
                    ▼
┌───────────────────────────────────────────────┐
│          packages/core (Engine)                │
│  ┌────────┐ ┌──────────┐ ┌─────────────────┐  │
│  │Client  │ │Content   │ │  Tool System    │  │
│  │(LLM    │ │Generator │ │  (30+ tools)    │  │
│  │Router) │ │(4        │ └────────┬────────┘  │
│  └───┬────┘ │backends) │          │           │
│      │      └────┬─────┘          │           │
│      └───────┬───┼───────────────┘            │
│              ▼   ▼                            │
│  ┌──────────────────────────────────┐         │
│  │  Core Services Layer             │         │
│  │  (session, git, shell, files,    │         │
│  │   telemetry, memory, cron, ...)  │         │
│  └────────────────┬─────────────────┘         │
│                   │                            │
│  ┌────────────────▼────────────────┐          │
│  │  Cross-Cutting Infrastructure  │           │
│  │  (config, models, permissions, │           │
│  │   hooks, LSP, MCP, IDE,        │           │
│  │   agents, skills, extensions,  │           │
│  │   telemetry, followup)         │           │
│  └────────────────────────────────┘           │
└───────────────────────────────────────────────┘
```

---

## Execution Modes

Qwen Code supports three distinct execution modes:

### 1. Interactive Mode (Default)
- **Entry:** `cli/src/gemini.tsx` → `main()`
- **UI Framework:** React + Ink (terminal React rendering)
- **Flow:** Parses CLI args → loads config → initializes app → renders `AppContainer` with Ink.
- **Input:** Interactive keyboard, keybindings, vim mode
- **Output:** Full terminal UI with syntax highlighting, panels, status bar

### 2. Non-Interactive Mode
- **Entry:** `cli/src/nonInteractiveCli.ts` or `cli/src/nonInteractive/session.ts`
- **Trigger:** `--stdin` flag or piped input
- **Flow:** Reads structured JSON from stdin (streaming or batch), processes via `ContentGenerator`, outputs JSON to stdout.
- **Features:** Supports streaming JSON output, dual-output bridging, and control messages.

### 3. ACP Server Mode
- **Entry:** `cli/src/serve/`
- **Protocol:** Agent Client Protocol (ACP) — an HTTP-based protocol for interacting with AI agents.
- **Flow:** Starts an Express HTTP server exposing an ACP-compliant API. Supports session management, streaming responses, tool execution, and file system operations.
- **Auth:** Token-based authentication with configurable single-user mode.

---

## Build & Deployment Pipeline

```
Source (TS/TSX) ──► TypeScript Compilation ──► dist/
                               │
                               ▼
                     esbuild Bundle ──► dist/cli.js
                               │
                    ┌──────────┼──────────┐
                    ▼          ▼          ▼
               npm pack    Docker     Standalone
               (.tgz)     (OCI)      Package
```

- **TypeScript:** Strict mode, `NodeNext` module resolution, `es2022` target
- **Bundling:** esbuild bundles `packages/cli/index.ts` → `dist/cli.js` (single ESM file)
- **Docker:** Two-stage build (builder + runtime) producing a slim Node 22 image
- **Sandbox:** Optional Docker/Podman sandbox for running agent subprocesses

---

## Configuration System

Configuration flows through multiple layers:

```
Settings Files (JSONC)
  └── CLI Argument Parsing (yargs)
       └── Environment Variables
            └── Config object (core/ src/config/config.ts)
                 ├── Model Registry (core/ src/models/)
                 ├── Provider Auth (core/ src/qwen/)
                 └── Permission Manager (core/ src/permissions/)
```

- **Settings files:** `~/.config/qwen-code/` or project-local `.qwen` directory
- **Schema:** JSON Schema for settings validation (generated)
- **Auth providers:** Qwen OAuth, Gemini API Key, OpenAI API Key, Anthropic API Key, Vertex AI

---

## Key Architectural Patterns

### Content Generator Strategy
The `ContentGenerator` interface abstracts LLM backend implementations:

| Backend | Package | Provider |
|---|---|---|
| Gemini | `core/src/core/geminiContentGenerator/` | Google Gemini API |
| OpenAI-compatible | `core/src/core/openaiContentGenerator/` | OpenAI API + compatible |
| Anthropic | `core/src/core/anthropicContentGenerator/` | Anthropic API |
| Qwen | `core/src/qwen/qwenContentGenerator.ts` | Qwen Dashscope API |
| Logging | `core/src/core/loggingContentGenerator/` | Logging wrapper |

The `Client` class (`core/src/core/client.ts`) selects and instantiates the appropriate generator based on config.

### Tool System
Tools are registered in a `ToolRegistry` and invoked by the LLM via function calling. Key patterns:
- **Lazy loading:** Heavy tool classes loaded on demand
- **Permission flow:** Every tool invocation goes through permission checks
- **MCP integration:** Remote tools via Model Context Protocol
- **Modifiable tools:** Tools that support pre/post modification hooks

### Hook System
A comprehensive event-driven hook system (`core/src/hooks/`) that fires at various lifecycle points:
- Pre/post tool use
- Permission requests
- Notifications
- SSRF guard
- HTTP hooks for external integration

### Memory System
A persistent file-based auto-memory system (`core/src/memory/`) that:
- Extracts key information from conversations
- Indexes and stores entries by relevance
- Recalls relevant memories via embedding similarity
- Supports forgetting and lifecycle management
