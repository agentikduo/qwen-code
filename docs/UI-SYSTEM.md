# UI System

Qwen Code uses **React** with **Ink** (React for terminals) to render a rich interactive terminal user interface.

---

## Technology Stack

| Technology | Purpose |
|---|---|
| **React 19.1** | Component framework |
| **Ink 6.2.3** | Terminal rendering engine |
| **Ink Testing Library** | Component testing |
| **@xterm/xterm** | Terminal emulation (pty output) |

---

## Component Hierarchy

```
<AppContainer>
  ├── <KeypressProvider>          — Global keybinding context
  ├── <VimModeProvider>           — Vim keybinding context
  ├── <SettingsContext.Provider>   — Settings context
  ├── <AgentViewProvider>         — Agent view context
  ├── <BackgroundTaskViewProvider> — Background task context
  └── <App>                       — Main application
        ├── <Layout>              — Layout (panels, tabs)
        │   ├── <Conversation>    — Chat/message history
        │   ├── <Input>           — User text input
        │   └── <StatusBar>       — Status bar
        ├── <OutputView>          — Tool output display
        ├── <PermissionDialog>    — Permission request dialogs
        ├── <AuthDialog>          — Authentication UI
        ├── <ModelSelector>       — Model management UI
        ├── <CommandPalette>      — Command palette
        └── <FeedbackDialog>      — User feedback dialog
```

---

## UI Source Structure (`packages/cli/src/ui/`)

### Components (`components/`)

Reusable UI components organized by category.

#### Shared Components (`components/shared/`)
- **MaxSizedBox** — Constrained box with max width/height
- Various layout primitives and styled wrappers

### Contexts (`contexts/`)

| Context | File | Purpose |
|---|---|---|
| **KeypressContext** | `KeypressContext.tsx` | Global keyboard event handling |
| **SettingsContext** | `SettingsContext.tsx` | Application settings |
| **SessionContext** | `SessionContext.tsx` | Session stats and state |
| **VimModeContext** | `VimModeContext.tsx` | Vim keybinding mode |
| **AgentViewContext** | `AgentViewContext.tsx` | Multi-agent view state |
| **BackgroundTaskViewContext** | `BackgroundTaskViewContext.tsx` | Background task display |

### Hooks (`hooks/`)

| Hook | File | Purpose |
|---|---|---|
| `useKittyKeyboardProtocol` | `useKittyKeyboardProtocol.ts` | Kitty keyboard protocol support |
| `useTerminalSize` | — | Responsive terminal layout |
| `useInput` | — | Input capture |

### Layouts (`layouts/`)

Defines the overall screen layout structure — panels, tabs, and responsive sizing.

### Themes (`themes/`)

| Component | File | Purpose |
|---|---|---|
| `ThemeManager` | `theme-manager.ts` | Theme loading and switching |
| `AUTO_THEME_NAME` | `theme-manager.ts` | Auto-detection of system theme |
| Theme definitions | — | Color palettes and style variables |

### State (`state/`)

UI state management — handles:
- Conversation history
- Current input state
- Permission dialogs
- Model selection
- Tool execution feedback

### Editors (`editors/`)

Text editor components for user input:
- Standard multiline input
- Vim-mode input handling
- Syntax highlighting

### Utils (`utils/`)

| Utility | File | Purpose |
|---|---|---|
| `kittyProtocolDetector.ts` | Auto-detect Kitty terminal protocol |
| `updateCheck.ts` | Version update notification |

### Models (`models/`)

Model management UI components — model selection, configuration display.

### NonInteractive (`noninteractive/`)

UI components for non-interactive mode display.

### Commands (`commands/`)

Command-related UI components.

### ManageModels (`manageModels/`)

Model management dialogs and configuration UI.

---

## Key UI Features

### Keyboard Input
- **Standard mode** — Normal text input
- **Vim mode** — Vim keybindings for navigation and editing
- **Kitty protocol** — Advanced keyboard protocol for better key detection

### Syntax Highlighting
- Uses `highlight.js` and `lowlight` for code rendering
- Supports multiple programming languages

### Status Bar
Shows:
- Current model
- Connection status
- Mode indicator
- Token usage
- System status

### Permission Dialogs
Interactive dialogs for:
- Approving/denying tool execution
- Shell command confirmation
- File operation confirmation

### Theme System
- Light and dark themes
- Auto-detection based on terminal theme
- Custom theme support via settings

---

## UI State Flow

```
User Input
    │
    ▼
KeypressProvider ──► VimModeProvider ──► Input Component
    │                                           │
    ▼                                           ▼
App State ────────► Tool Execution
    │                      │
    ▼                      ▼
Conversation Update    Output Display
    │                      │
    ▼                      ▼
React Re-render ───────► Ink Virtual DOM
                                │
                                ▼
                         Terminal Output
```

## Colors & Styling

| File | Purpose |
|---|---|
| `colors.ts` | Base color definitions |
| `semantic-colors.ts` | Semantic color mappings |
| `constants.ts` | Layout constants (margins, padding, etc.) |
| `textConstants.ts` | Text-related constants |
| `types.ts` | UI type definitions |

---

## Auto-Update Notifications

The UI checks for updates via `update-notifier` and displays a notification banner when a new version is available.
