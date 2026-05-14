# Qwen Code Documentation

This directory contains comprehensive documentation for the Qwen Code project architecture, technology stack, and internal systems.

## Contents

| Document | Description |
|---|---|
| [ARCHITECTURE.md](ARCHITECTURE.md) | High-level architecture overview, execution modes, build pipeline |
| [TECH-STACK.md](TECH-STACK.md) | Complete technology stack with dependencies |
| [PACKAGES.md](PACKAGES.md) | Package-by-package breakdown with directory structures |
| [AGENT-SYSTEM.md](AGENT-SYSTEM.md) | Multi-agent infrastructure (arena, backends, runtime) |
| [TOOL-SYSTEM.md](TOOL-SYSTEM.md) | Tool architecture, tool registry, permission system |
| [TESTING.md](TESTING.md) | Testing strategy — unit, integration, E2E, benchmarks |
| [HOOKS-AND-EXTENSIONS.md](HOOKS-AND-EXTENSIONS.md) | Hook system, extension system, skills, MCP integration |
| [UI-SYSTEM.md](UI-SYSTEM.md) | Terminal UI framework (React/Ink), component hierarchy |
| [DATA-FLOW.md](DATA-FLOW.md) | Data flow diagrams for all execution modes |

## Quick Reference

### Key Source Files

| File | Purpose |
|---|---|
| `packages/cli/index.ts` | Binary entry point |
| `packages/cli/src/gemini.tsx` | Main `main()` function |
| `packages/core/src/core/client.ts` | LLM client (chat loop engine) |
| `packages/core/src/core/contentGenerator.ts` | ContentGenerator interface |
| `packages/core/src/config/config.ts` | Config class |
| `packages/core/src/tools/tools.ts` | Tool runner |
| `packages/core/src/services/sessionService.ts` | Session management |
| `packages/core/src/memory/manager.ts` | Memory manager |

### Key Commands

```bash
npm install        # Install dependencies
npm run build      # Build all packages
npm run bundle     # Bundle dist/cli.js
npm run dev        # Run from source (tsx)
npm test           # Run tests
npm run preflight  # Full quality gate
```
