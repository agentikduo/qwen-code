# Data Flow & Execution Model

This document describes how data flows through Qwen Code during a typical interaction — from user input to LLM response to tool execution and back.

---

## Main Execution Flow

```
User Input
    │
    ▼
┌──────────────────────────────────────────────────┐
│  1. CLI Entry (packages/cli/index.ts)            │
│     • `#!/usr/bin/env node`                      │
│     • Init startup profiler                      │
│     • Import gemini.tsx → call main()             │
│     • Global error handling (PTY race, fatal)    │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  2. main() (packages/cli/src/gemini.tsx)         │
│     • Parse CLI args (yargs)                     │
│     • Load settings (~/.config/qwen-code/)       │
│     • Set up telemetry & logging                 │
│     • Initialize Config object                   │
│     • Detect IDE integration                     │
│     • Select mode:                               │
│       ├── Interactive (Ink + React)              │
│       ├── Non-interactive (stdin JSON)           │
│       └── Server (HTTP/ACP)                      │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  3. Interactive Render (Ink)                     │
│     • Render <AppContainer>                      │
│     • User types input                           │
│     • Capture via KeypressProvider               │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  4. Send to LLM (Client)                         │
│     • Build request content                      │
│     • Add system prompts + context               │
│     • Select ContentGenerator by AuthType        │
│     • Call generateContent or generateContentStream
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  5. LLM Response Processing                      │
│     • Stream response (or batch)                 │
│     • Parse function calls (tool requests)       │
│     • Handle text content                        │
│     • Manage context limits / compression        │
└──────────────────┬───────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
  (Text Response)      (Tool Call)
        │                     │
        ▼                     ▼
┌──────────────┐   ┌──────────────────────────────┐
│ Display to   │   │  6. Tool Execution Pipeline   │
│ user via Ink │   │  • ToolScheduler schedules    │
└──────────────┘   │  • PermissionFlow checks      │
                   │  • HookSystem fires hooks     │
                   │  • Tool executes               │
                   │  • Result returned to LLM      │
                   └──────────────┬───────────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │  7. LLM Continues        │
                    │  • Tool result injected  │
                    │  • LLM generates next    │
                    │    response or tool call │
                    └──────────────────────────┘
```

---

## Model Selection Flow

```
User config or --model flag
         │
         ▼
┌───────────────────────────────────────┐
│  ModelRegistry (models/modelRegistry.ts) │
│  • Maps model IDs to provider config  │
│  • Validates model capabilities       │
│  • Handles model aliases              │
└──────────────┬────────────────────────┘
               │
               ▼
┌───────────────────────────────────────┐
│  ModelConfigResolver                  │
│  (models/modelConfigResolver.ts)      │
│  • Resolves model from:               │
│    ├── CLI flag (--model)             │
│    ├── Settings (defaultModel)        │
│    ├── Environment variable           │
│    └── Built-in default               │
│  • Validates API key availability     │
│  • Resolves base URL, API key, etc.    │
└──────────────┬────────────────────────┘
               │
               ▼
┌───────────────────────────────────────┐
│  ContentGenerator Selection           │
│  (by AuthType)                        │
│                                       │
│  AuthType.USE_GEMINI  → Gemini        │
│  AuthType.USE_OPENAI  → OpenAI        │
│  AuthType.USE_ANTHROPIC → Anthropic   │
│  AuthType.QWEN_OAUTH  → Qwen          │
│  AuthType.USE_VERTEX_AI → Vertex AI   │
└───────────────────────────────────────┘
```

---

## Session Lifecycle

```
┌─────────────┐
│ New Session │
└──────┬──────┘
       │
       ▼
┌────────────────────────────┐
│ SessionService.create()    │
│ • Create session directory │
│ • Initialize state         │
│ • Load previous history    │
└────────────┬───────────────┘
             │
             ▼
     ┌───────────────┐
     │ Chat Loop     │◄──────────────┐
     │ (Client.run)  │              │
     └───────┬───────┘              │
             │                      │
             ▼                      │
     ┌──────────────────┐           │
     │ User Input       │           │
     └──────┬───────────┘           │
            │                       │
            ▼                       │
     ┌──────────────────┐           │
     │ LLM Response     │           │
     └──────┬───────────┘           │
            │                       │
     ┌──────┴──────┐               │
     ▼             ▼               │
  (Text)      (Tool Call)          │
     │             │               │
     │             ▼               │
     │     ┌──────────────┐        │
     │     │ Execute Tool │        │
     │     └──────┬───────┘        │
     │            │                │
     └─────┬──────┘                │
           │                       │
           ▼                       │
     ┌──────────────┐              │
     │ Continue?    ├──────────────┘
     └──────┬───────┘ (yes)
            │ (no)
            ▼
     ┌──────────────┐
     │ End Session  │
     │ • Save state │
     │ • Summary    │
     └──────────────┘
```

---

## Non-Interactive Mode Data Flow

```
External Process
     │
     ▼
stdin (JSON lines)
     │
     ▼
┌────────────────────────────────┐
│ StreamJsonInputReader          │
│ (nonInteractive/io/)           │
│ • Parse JSON lines from stdin  │
│ • Deserialize CLIMessage types │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ ControlDispatcher              │
│ (nonInteractive/control/)      │
│ • Route control messages       │
│ • Manage session lifecycle     │
│ • Handle cancellations         │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ runNonInteractive()            │
│ (nonInteractiveCli.ts)         │
│ • Process user messages        │
│ • Call LLM via Client          │
│ • Execute tools                 │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ StreamJsonOutputAdapter        │
│ (nonInteractive/io/)           │
│ • Serialize results to JSON    │
│ • Write to stdout              │
└────────────┬───────────────────┘
             │
             ▼
stdout (JSON lines)
     │
     ▼
External Process
```

---

## Server Mode Data Flow (ACP Protocol)

```
HTTP Client
     │
     ▼
┌────────────────────────────────┐
│ Express HTTP Server            │
│ (serve/server.ts)             │
│ • Route: POST /acp/*           │
│ • Auth middleware              │
│ • CORS headers                 │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ ACP Request Handlers           │
│ (serve/httpAcpBridge.ts)       │
│ • Map HTTP → ACP protocol      │
│ • Manage sessions              │
│ • Stream responses             │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ Session Manager                │
│ (acp-integration/session/)     │
│ • Create/manage sessions       │
│ • Track sub-agents             │
│ • Permission handling          │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ LLM Engine (via Client)        │
│ • Process messages             │
│ • Execute tools                 │
│ • Return responses             │
└────────────────────────────────┘
     │
     ▼
HTTP Response (JSON / SSE stream)
```

---

## Memory System Data Flow

```
Conversation Turn
       │
       ▼
┌────────────────────────────────┐
│ MemoryManager.extract()        │
│ • Analyze conversation         │
│ • Identify key information     │
│ • Classify memory type         │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ MemoryIndexer                  │
│ (memory/indexer.ts)            │
│ • Create embeddings            │
│ • Update vector index          │
│ • Store in filesystem          │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ Memory Recall                  │
│ (on next relevant turn)        │
│ • Calculate relevance scores   │
│ • Select top-K memories        │
│ • Inject into system prompt    │
└────────────────────────────────┘
```
