# Hooks & Extension System

Qwen Code provides a comprehensive hook system and extension mechanism for integrating external tools, custom behaviors, and third-party services.

---

## Hook System

### Overview

The hook system (`packages/core/src/hooks/`) is an event-driven framework that allows code to be injected at specific lifecycle points during Qwen Code's operation.

### Architecture

```
┌─────────────────────────────────────────────┐
│               HookSystem                     │
│  (hookSystem.ts — central orchestrator)      │
├─────────────────────────────────────────────┤
│  HookRegistry (hookRegistry.ts)             │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐ │
│  │ Function │ │  HTTP    │ │   Skill      │ │
│  │  Hooks   │ │  Hooks   │ │   Hooks      │ │
│  └──────────┘ └──────────┘ └──────────────┘ │
└─────────────────────────────────────────────┘
```

### Hook Types

| Type | Description |
|---|---|
| **PreToolUse** | Before tool execution — can modify/approve/deny |
| **PostToolUse** | After successful tool execution |
| **PostToolUseFailure** | After failed tool execution |
| **PermissionRequest** | When permission is required |
| **Notification** | General event notifications |

### Hook Runners

| Runner | File | Description |
|---|---|---|
| `FunctionHookRunner` | `functionHookRunner.ts` | Execute JavaScript/TypeScript functions as hooks |
| `HttpHookRunner` | `httpHookRunner.ts` | Make HTTP requests for hook execution |
| `HookRunner` | `hookRunner.ts` | Generic hook runner base |
| `HookAggregator` | `hookAggregator.ts` | Aggregate results from multiple hooks |
| `HookPlanner` | `hookPlanner.ts` | Plan execution order for hooks |

### Hook Events (`core/src/core/toolHookTriggers.ts`)

| Trigger | When Fired |
|---|---|
| `firePreToolUseHook` | Before each tool execution |
| `firePostToolUseHook` | After each successful tool execution |
| `firePostToolUseFailureHook` | After each failed tool execution |
| `firePermissionRequestHook` | When a permission decision is needed |
| `fireNotificationHook` | For general events |

### Hook Locations

Hooks can be defined in:
1. **Settings files** — JSONC configuration with hook definitions
2. **CLI commands** — `packages/cli/src/commands/hooks.tsx`
3. **Skills** — Bundled or user-installed skills
4. **Extensions** — Third-party extensions

### Hook Configuration (Settings)

Hooks are configured in the user's Qwen Code settings file:

```jsonc
{
  "hooks": {
    "preToolUse": [
      {
        "type": "http",
        "url": "https://example.com/pre-tool",
        "timeout": 5000
      },
      {
        "type": "function",
        "name": "myCustomHook",
        "source": "./path/to/hook.js"
      }
    ]
  }
}
```

### Safety Features

| Feature | File | Purpose |
|---|---|---|
| **SSRF Guard** | `ssrfGuard.ts` | Prevents SSRF attacks via hook URLs |
| **URL Validator** | `urlValidator.ts` | Validates hook endpoint URLs |
| **Trusted Hooks** | `trustedHooks.ts` | Manages trusted hook sources |
| **Env Interpolator** | `envInterpolator.ts` | Safe environment variable interpolation |

---

## Extension System

### Overview

The extension system (`packages/core/src/extension/`) provides a mechanism for discovering, installing, and managing extensions that extend Qwen Code's capabilities.

### Extension Sources

| Source | Module | Description |
|---|---|---|
| **GitHub** | `github.ts` | Install extensions from GitHub repos |
| **npm** | `npm.ts` | Install extensions from npm registry |
| **Marketplace** | `marketplace.ts` | Browse and install from extension marketplace |
| **Settings** | `settings.ts` | Configure extensions via settings |
| **Storage** | `storage.ts` | Persistent extension storage |

### Extension Manager

The `ExtensionManager` class (`extensionManager.ts`) handles:
- Installing extensions from sources
- Loading extension configurations
- Managing extension lifecycle (enable/disable)
- Variable resolution and schema validation

### Extension Converters

| Converter | File | Purpose |
|---|---|---|
| **Claude Converter** | `claude-converter.ts` | Convert Claude Code custom modes to extensions |
| **Gemini Converter** | `gemini-converter.ts` | Convert Gemini configurations |

### Extension Override System

The `Override` system (`override.ts`) allows extensions to:
- Override system prompts
- Override tool configurations
- Override model settings
- Override permission rules

---

## Skills System

### Overview

The skills system (`packages/core/src/skills/`) provides a mechanism for loading and executing skills — reusable, packaged capabilities.

### Skill Management

| Component | File | Description |
|---|---|---|
| `SkillManager` | `skill-manager.ts` | Central skill management |
| `SkillLoader` | `skill-load.ts` | Load skills from paths |
| `SkillActivator` | `skill-activation.ts` | Activate/deactivate skills |
| `SkillPaths` | `skill-paths.ts` | Skill path resolution |
| `SymlinkScope` | `symlinkScope.ts` | Symlink-based scope management |

### Bundled Skills

Located in `packages/core/src/skills/bundled/` — these are skills shipped with Qwen Code.

### Skill Integration

Skills integrate with:
- **Hooks** — via `registerSkillHooks.ts`
- **Commands** — via CLI command system
- **Tools** — via `SkillTool` (`tools/skill.ts`)
- **Memory** — via `skillReviewAgentPlanner.ts`

---

## MCP (Model Context Protocol) Integration

### Overview

MCP integration (`packages/core/src/mcp/`) allows Qwen Code to connect to MCP servers for additional tools and capabilities.

### Components

| Component | File | Description |
|---|---|---|
| **MCP OAuth Provider** | `oauth-provider.ts` | OAuth 2.0 authentication for MCP |
| **MCP OAuth Token Storage** | `oauth-token-storage.ts` | Persistent OAuth token storage |
| **OAuth Utils** | `oauth-utils.ts` | OAuth protocol utilities |
| **Keychain Token Storage** | `token-storage/keychain-token-storage.ts` | OS keychain integration |
| **Google Auth Provider** | `google-auth-provider.ts` | Google Cloud auth for MCP |
| **SA Impersonation** | `sa-impersonation-provider.ts` | Service account impersonation |

### CLI Commands

MCP servers are managed via CLI commands (`packages/cli/src/commands/mcp.ts`).

---

## Prompt System

### Overview

The prompt system (`packages/core/src/prompts/`) manages MCP prompts and prompt templates.

### Components

| Component | File | Description |
|---|---|---|
| `PromptRegistry` | `prompt-registry.ts` | Central prompt registry |
| `MCP Prompts` | `mcp-prompts.ts` | MCP prompt definitions |

### CLI Commands

MCP prompts are loaded and managed via `McpPromptLoader` (`packages/cli/src/services/McpPromptLoader.ts`).
