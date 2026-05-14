# Technology Stack

## Core Language & Runtime

| Technology | Version | Purpose |
|---|---|---|
| **Node.js** | >= 22.0.0 | JavaScript runtime |
| **TypeScript** | ^5.3.3 | Strict mode, `es2022` target, `NodeNext` modules |
| **ESM** | — | Pure ESM throughout (`"type": "module"`) |

## Core Dependencies

### LLM / AI SDKs
| Package | Purpose |
|---|---|
| `@google/genai` 1.30.0 | Google Gemini API client |
| `openai` 5.11.0 | OpenAI-compatible API client |
| `@anthropic-ai/sdk` ^0.36.1 | Anthropic Claude API client |
| `@modelcontextprotocol/sdk` ^1.25.1 | MCP (Model Context Protocol) client/server |

### Terminal UI
| Package | Purpose |
|---|---|
| **ink** ^6.2.3 | React for terminal — renders components to stdout |
| **react** ^19.1.0 | UI component framework |
| `ink-gradient` ^3.0.0 | Gradient text in terminal |
| `ink-link` ^4.1.0 | Clickable terminal links |
| `ink-spinner` ^5.0.0 | Loading spinners |
| `@xterm/headless` 5.5.0 | Headless xterm for pty integration |
| `ansi-regex` ^6.2.2 | ANSI escape code detection |
| `strip-ansi` ^7.1.0 | Strip ANSI escape codes |
| `wrap-ansi` ^10.0.0 | Word-wrap with ANSI support |
| `string-width` ^7.1.0 | Display width of strings |

### CLI & Configuration
| Package | Purpose |
|---|---|
| **yargs** ^17.7.2 | CLI argument parsing |
| **zod** ^3.23.8 | Runtime schema validation |
| `comment-json` ^4.2.5 | JSON with comments parsing |
| `dotenv` ^17.1.0 | Environment variable loading |
| `@iarna/toml` ^2.2.5 | TOML format support |
| `ajv` + `ajv-formats` | JSON Schema validation |
| `prompts` ^2.4.2 | Interactive prompts |
| `update-notifier` ^7.3.1 | Update notifications |

### File System & Shell
| Package | Purpose |
|---|---|
| `chokidar` ^4.0.3 | File system watching |
| `fdir` ^6.4.6 | Fast directory traversal |
| `picomatch` ^4.0.1 | Glob pattern matching |
| `ignore` ^7.0.0 | `.gitignore` pattern matching |
| `glob` ^10.5.0 | File globbing |
| `fzf` ^0.5.2 | Fuzzy finder |
| `shell-quote` ^1.8.3 | Shell command parsing |
| `@lydell/node-pty` (optional) | PTY process spawning |

### Text Processing & Code Analysis
| Package | Purpose |
|---|---|
| `marked` ^15.0.12 | Markdown parsing |
| `html-to-text` ^9.0.5 | HTML to text conversion |
| `diff` ^7.0.0 | Text diffing |
| `highlight.js` ^11.11.1 | Syntax highlighting |
| `lowlight` ^3.3.0 | Syntax highlighting (server-side) |
| `chardet` ^2.1.0 | Character encoding detection |
| `iconv-lite` ^0.6.3 | Text encoding conversion |
| `web-tree-sitter` ^0.24.7 | Tree-sitter WASM for code parsing |

### Networking
| Package | Purpose |
|---|---|
| **undici** ^6.22.0 | HTTP client (node-fetch replacement) |
| `express` ^5.2.1 | HTTP server (for ACP mode) |
| `ws` ^8.18.0 | WebSocket client/server |
| `https-proxy-agent` ^7.0.6 | HTTP proxy support |
| `open` ^10.1.2 | Open URLs in browser |

### Telemetry & Observability
| Package | Purpose |
|---|---|
| **@opentelemetry/api** ^1.9.0 | OpenTelemetry tracing API |
| **@opentelemetry/sdk-node** ^0.203.0 | OpenTelemetry Node.js SDK |
| OTLP exporters (x6) | Trace/log/metric export via gRPC/HTTP |

### Other
| Package | Purpose |
|---|---|
| `simple-git` ^3.28.0 | Git operations |
| `tar` ^7.5.2 | Archive extraction |
| `extract-zip` ^2.0.1 | ZIP extraction |
| `uuid` ^9.0.1 | UUID generation |
| `jsonrepair` ^3.13.0 | JSON repair |
| `fast-levenshtein` ^2.0.6 | String similarity |
| `mime` 4.0.7 | MIME type detection |
| `async-mutex` ^0.5.0 | Async mutex/locking |

---

## Development Tools

| Tool | Version | Purpose |
|---|---|---|
| **esbuild** | ^0.25.0 | Bundler |
| **eslint** | ^9.24.0 | Linting (flat config) |
| **prettier** | ^3.5.3 | Code formatting |
| **vitest** | ^3.2.4 | Test runner |
| **tsx** | ^4.20.3 | TypeScript execution (dev mode) |
| **husky** | ^9.1.7 | Git hooks |
| **lint-staged** | ^16.1.6 | Staged file linting |
| **typescript-eslint** | ^8.30.1 | TypeScript ESLint rules |
| globals | ^16.0.0 | ESLint global definitions |
| **msw** | ^2.10.4 | HTTP mocking (tests) |
| **memfs** | ^4.42.0 | In-memory filesystem (tests) |
| **mock-fs** | ^5.5.0 | Mock filesystem (tests) |
| **mnemonist** | ^0.40.3 | Data structures (tests) |

## SDK Packages

| SDK | Language | Build System | Target |
|---|---|---|---|
| `packages/sdk-typescript` | TypeScript | esbuild | Node.js >= 22 |
| `packages/sdk-python` | Python 3.10+ | hatchling | Python SDK |
| `packages/sdk-java/client` | Java 8+ | Maven | ACP Java SDK |
| `packages/sdk-java/qwencode` | Java 8+ | Maven | Qwen Code Java SDK |

## IDE & Editor Extensions

| Extension | Platform | Technology |
|---|---|---|
| `packages/vscode-ide-companion` | VS Code | TypeScript + esbuild |
| `packages/zed-extension` | Zed Editor | extension.toml |

## Channel Integrations (Chat Platforms)

| Channel | Package | Purpose |
|---|---|---|
| Telegram | `packages/channels/telegram` | Telegram bot integration |
| DingTalk | `packages/channels/dingtalk` | DingTalk bot integration |
| WeChat (Weixin) | `packages/channels/weixin` | WeChat integration |
| Base | `packages/channels/base` | Abstract channel base class |

## Infrastructure

| Component | Details |
|---|---|
| **Container runtime** | Node 22 slim (Docker multi-stage build) |
| **Sandbox support** | Docker / Podman |
| **CI** | GitHub Actions (inferred) |
| **Package registry** | npm + GitHub Container Registry (ghcr.io) |
