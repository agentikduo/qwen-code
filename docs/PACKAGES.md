# Package Structure

The project is an npm workspaces monorepo defined at the root `package.json`. Workspaces span `packages/*` and several nested paths under `packages/channels/*`.

---

## Root Package (`@qwen-code/qwen-code`)

- **Purpose:** Monorepo orchestration, shared tooling, build scripts
- **Location:** `./`
- **Key:**
  - `package.json` — workspaces config, unified scripts
  - `esbuild.config.js` — Bundle `packages/cli` → `dist/cli.js`
  - `eslint.config.js` — Shared flat ESLint config
  - `tsconfig.json` — Base TS config (all packages extend)
  - `Dockerfile` — Container image build
  - `Makefile` — Convenience targets (build, test, lint, etc.)

### Scripts (`./scripts/`)
| Script | Purpose |
|---|---|
| `build.js` | Build all workspaces |
| `build_package.js` | Build individual package (TS compile + asset copy) |
| `build_sandbox.js` | Build sandbox container (non-root user setup) |
| `build_vscode_companion.js` | Build VS Code extension |
| `dev.js` | Run CLI in dev mode via tsx |
| `start.js` | Run the built CLI |
| `bundle.js` alias `esbuild.config.js` | Bundle with esbuild |
| `clean.js` | Clean all build artifacts |
| `version.js` | Version management |
| `telemetry.js` | Telemetry collection |
| `prepare-package.js` | Prepare package for distribution |
| `create-standalone-package.js` | Create standalone package |
| `pre-commit.js` | Pre-commit hook |
| `lint.js` | Multi-package linting |
| Various sandbox profiles (`.sb`) | macOS sandbox configs for different permission levels |

---

## `packages/core` (`@qwen-code/qwen-code-core`)

**Purpose:** The core engine — all LLM interaction, tool execution, business logic, and services.

### Directory Layout

```
src/
├── agents/         # Multi-agent infrastructure (Arena, Team, Swarm modes)
│   ├── arena/      # Arena mode — concurrent agent subprocesses
│   ├── backends/   # Display backends (tmux, iTerm2, InProcess)
│   └── runtime/    # Agent runtime (context, core, headless, interactive)
├── config/         # Core config, storage, model defaults
├── confirmation-bus/ # Message bus for hook execution
├── core/           # LLM client, content generators, tool scheduler, chat
│   ├── anthropicContentGenerator/  # Anthropic Claude backend
│   ├── geminiContentGenerator/     # Google Gemini backend
│   ├── loggingContentGenerator/    # Logging wrapper
│   └── openaiContentGenerator/     # OpenAI-compatible backend
├── extension/      # Extension system (CLI commands, MCP, settings)
├── followup/       # Follow-up suggestion engine
├── hooks/          # Event-driven hook system (pre/post tool, permissions, HTTP)
├── ide/            # IDE integration client (VS Code, etc.)
├── lsp/            # Language Server Protocol integration
├── mcp/            # Model Context Protocol infrastructure (OAuth, tokens)
├── memory/         # Auto-memory system (extract, index, recall, forget)
├── models/         # Model registry, config resolution, capabilities
├── output/         # JSON formatter & output types
├── permissions/    # Permission manager (tool approval, shell semantics)
├── prompts/        # MCP prompts registry
├── qwen/           # Qwen-specific auth (OAuth2) & content generator
├── services/       # Core business services
│   ├── microcompaction/  # Chat history compression
│   ├── backgroundShellRegistry.ts
│   ├── chatCompressionService.ts
│   ├── chatRecordingService.ts
│   ├── cronScheduler.ts
│   ├── fileDiscoveryService.ts
│   ├── fileReadCache.ts
│   ├── fileSystemService.ts
│   ├── gitService.ts / gitWorktreeService.ts
│   ├── loopDetectionService.ts
│   ├── monitorRegistry.ts
│   ├── sessionRecap.ts / sessionService.ts / sessionTitle.ts
│   ├── shellExecutionService.ts
│   └── toolUseSummary.ts
├── skills/         # Skill system (loading, activation, management)
├── subagents/      # Subagent orchestration (model selection, validation)
├── telemetry/      # OpenTelemetry integration, logging, metrics
├── test-utils/     # Test helpers (fake config, etc.)
├── tools/          # Tool implementations (30+ tools)
│   ├── agent/      # Agent spawn/fork tool
│   ├── cron-create/ cron-list/ cron-delete/
│   ├── edit.ts, glob.ts, grep.ts, ls.ts, lsp.ts
│   ├── mcp-client.ts, mcp-client-manager.ts
│   ├── read-file.ts, write-file.ts, shell.ts
│   ├── skill.ts, todoWrite.ts, web-fetch.ts
│   └── askUserQuestion.ts, send-message.ts, monitor.ts, ...
└── utils/          # Utilities (150+ files)
    ├── filesearch/ # File search
    └── request-tokenizer/ # Token counting
```

### Key Source Files

| File | Purpose |
|---|---|
| `index.ts` | Public API exports — re-exports from all modules |
| `config/config.ts` | `Config` class — the central configuration object |
| `config/storage.ts` | Persistent storage layer |
| `core/client.ts` | `Client` class — LLM orchestration, chat loop, turn management |
| `core/contentGenerator.ts` | `ContentGenerator` interface + `AuthType` enum |
| `core/coreToolScheduler.ts` | Tool scheduling and execution |
| `core/permissionFlow.ts` | Permission request/response flow |
| `core/turn.ts` | Chat turn model |
| `tools/tools.ts` | Tool runner |
| `tools/tool-registry.ts` | Tool registry |
| `services/sessionService.ts` | Session management |
| `memory/manager.ts` | MemoryManager — auto-memory public API |

---

## `packages/cli` (`@qwen-code/qwen-code`)

**Purpose:** The CLI entry point — UI rendering, user interaction, command routing.

### Directory Layout

```
src/
├── acp-integration/     # Agent Client Protocol integration
│   ├── service/         # ACP file system service
│   └── session/         # ACP session management
├── auth/                # Authentication providers
│   ├── install/         # Auth install helpers
│   └── providers/       # Provider implementations
├── commands/            # CLI commands (auth, channel, extensions, hooks, mcp, review, serve)
├── config/              # CLI-specific config (settings, keybindings, auth)
│   └── migration/       # Config migration
├── core/                # CLI core (auth, initializer, theme)
├── dualOutput/          # Dual output (JSON + TUI simultaneously)
├── export/              # Export functionality
├── i18n/                # Internationalization
│   └── locales/         # Translation files
├── nonInteractive/      # Non-interactive (streaming JSON) mode
│   ├── io/              # I/O adapters (JSON reader/writer)
│   └── control/         # Control message dispatching
├── remoteInput/         # Remote input watcher
├── serve/               # ACP HTTP server mode
├── services/            # CLI services
│   ├── insight/         # Insight protocol integration
│   ├── prompt-processors/
│   ├── test-commands/
│   └── tips/
├── ui/                  # Terminal UI (React/Ink components)
│   ├── auth/            # Auth UI
│   ├── commands/        # Command UI
│   ├── components/      # Shared UI components
│   ├── contexts/        # React contexts
│   ├── editors/         # Editor components
│   ├── hooks/           # React hooks
│   ├── layouts/         # Layout components
│   ├── manageModels/    # Model management UI
│   ├── models/          # Model display components
│   ├── noninteractive/  # Non-interactive UI
│   ├── state/           # UI state management
│   ├── themes/          # Theme system
│   └── utils/           # UI utilities
├── utils/               # CLI utilities
├── gemini.tsx           # Main entry: `main()` function
├── gemini.test.tsx      # Integration tests for main
├── nonInteractiveCli.ts # Non-interactive mode entry
└── nonInteractiveCliCommands.ts # Non-interactive command definitions
```

### Key Source Files

| File | Purpose |
|---|---|
| `index.ts` | CLI binary entry — `#!/usr/bin/env node`, error handling |
| `gemini.tsx` | `main()` — startup orchestration, mode selection, Ink render |
| `nonInteractiveCli.ts` | `runNonInteractive()` — batch non-interactive mode |
| `nonInteractive/session.ts` | `runNonInteractiveStreamJson()` — streaming non-interactive mode |
| `serve/` | HTTP server mode |
| `config/settings.ts` | Settings loading and schema |
| `core/initializer.ts` | App initialization |

---

## `packages/sdk-typescript` (`@qwen-code/sdk`)

**Purpose:** TypeScript SDK for programmatic access to Qwen Code.

```
src/
├── daemon/      # Daemon management
├── mcp/         # MCP client
├── query/       # Query building
├── transport/   # Transport layer (stdio, etc.)
├── types/       # TypeScript type definitions
├── utils/       # Utilities
└── index.ts     # Public API
```

## `packages/sdk-python` (`qwen-code-sdk`)

**Purpose:** Python SDK for programmatic access.
- Build system: hatchling
- Python: >= 3.10
- Source: `src/qwen_code_sdk/`

## `packages/sdk-java`

**Purpose:** Java SDKs (Maven-based).
- `client/` — ACP Java SDK (`acp-sdk`, Java 8+)
- `qwencode/` — Qwen Code Java SDK (`qwencode-sdk`, Java 8+)

## `packages/vscode-ide-companion`

**Purpose:** VS Code extension for IDE integration.
- Webview-based UI
- IDE protocol server for file/diff operations
- Tailwind CSS for styling

## `packages/web-templates`

**Purpose:** HTML templates for export/insight features.
- Export HTML templates
- Insight visualization templates

## `packages/webui`

**Purpose:** Standalone web UI component library (Storybook).
- React components with Tailwind CSS
- Vite-based build
- Storybook for component documentation

## `packages/zed-extension`

**Purpose:** Zed editor extension.
- Configuration: `extension.toml`
- Minimal — just extension manifest and icon

## `packages/channels/`

**Purpose:** Chat platform integrations.

| Package | Module Name |
|---|---|
| `base` | `@qwen-code/channel-base` — Abstract base class |
| `telegram` | `@qwen-code/channel-telegram` — Telegram bot |
| `dingtalk` | `@qwen-code/channel-dingtalk` — DingTalk bot |
| `weixin` | `@qwen-code/channel-weixin` — WeChat bot |
| `plugin-example` | Example channel plugin |
