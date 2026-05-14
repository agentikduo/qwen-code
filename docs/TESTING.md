# Testing Strategy

Qwen Code employs a multi-layered testing strategy covering unit, integration, end-to-end, and performance testing.

---

## Testing Layers

### 1. Unit Tests

**Framework:** Vitest (v3.2.4)
**Location:** Collocated with source files as `*.test.ts` or `*.test.tsx`

Each package has its own vitest configuration:

| Package | Config File |
|---|---|
| Root | `vitest.workspace.ts` (workspace config) |
| `packages/core` | `vitest.config.ts` |
| `packages/cli` | `vitest.config.ts` |
| `packages/sdk-typescript` | `vitest.config.ts` |
| Integration tests | `integration-tests/vitest.config.ts` |

**Running Tests:**
```bash
# Single file (preferred)
cd packages/core && npx vitest run src/path/to/file.test.ts
cd packages/cli && npx vitest run src/path/to/file.test.ts

# All tests in a package
cd packages/core && npx vitest run

# Parallel across workspaces (from root)
npm run test
```

**Key Testing Patterns:**
- **`msw`** — HTTP mocking for API-dependent tests
- **`memfs`** / **`mock-fs`** — In-memory filesystem for file operation tests
- **`vi.hoisted()`** — For mocks consumed by `vi.mock()` in CLI tests
- **`ink-testing-library`** — React/Ink component testing in CLI package
- **`@testing-library/react`** — React component testing
- **Collocated snapshots** — `__snapshots__/` directories next to test files

### 2. Integration Tests

**Location:** `integration-tests/`

```
integration-tests/
├── cli/                  # CLI integration tests (non-interactive)
├── interactive/          # Interactive mode tests (tmux-based)
├── sdk-typescript/       # TypeScript SDK integration tests
├── concurrent-runner/    # Concurrent execution tests
├── fixtures/             # Test fixtures
├── hook-integration/     # Hook system integration tests
├── terminal-bench/       # Terminal benchmark tests
├── terminal-capture/     # Terminal screenshot tests
├── globalSetup.ts        # Global test setup
├── test-helper.ts        # Test utilities
├── test-mcp-server.ts    # Test MCP server
├── vitest.config.ts      # Vitest config for integration tests
└── vitest.terminal-bench.config.ts  # Benchmark test config
```

**Running Integration Tests:**
```bash
# CLI tests (no sandbox)
npm run test:integration:cli:sandbox:none

# Interactive tests (no sandbox)
npm run test:integration:interactive:sandbox:none

# All integration tests
npm run test:integration:all
```

**Sandbox variants:**
| Script | Sandbox |
|---|---|
| `sandbox:none` | No sandbox (local execution) |
| `sandbox:docker` | Docker sandbox |
| `sandbox:podman` | Podman sandbox |

### 3. SDK Tests

| SDK | Framework | Command |
|---|---|---|
| TypeScript | Vitest | `cd packages/sdk-typescript && npm test` |
| Python | pytest | `npm run test:sdk:python` |
| Java | JUnit 5 | Via Maven (`mvn test`) |

### 4. Terminal Benchmarks

**Config:** `vitest.terminal-bench.config.ts`
**Purpose:** Measure terminal rendering performance for CLI commands.

```bash
npm run test:terminal-bench
npm run test:terminal-bench:oracle    # Oracle benchmark
npm run test:terminal-bench:qwen      # Qwen benchmark
```

### 5. E2E Tests

```bash
npm run test:e2e
```

Runs interactive integration tests verbosely with output kept for analysis.

---

## Test Utilities

### Core Package (`packages/core/src/test-utils/`)
- `makeFakeConfig()` — Create a fake `Config` object for testing
- Various mocks and stubs

### CLI Package (`packages/cli/src/test-utils/`)
- Ink testing helpers
- CLI-specific mock utilities

### Integration Tests (`integration-tests/test-helper.ts`)
- Test harness for spawning Qwen Code processes
- Fixture loading
- Result assertion helpers

---

## Mocking Strategy

| Mocking Need | Tool |
|---|---|
| HTTP APIs | **msw** (Mock Service Worker) |
| File system | **memfs** / **mock-fs** |
| ES module mocking | `vi.mock()` |
| Clock/timers | `vi.useFakeTimers()` |
| PTY (pseudo-terminal) | Mocked via testing utilities |
| Git operations | In-memory git repos (simple-git+mock) |

---

## Linting & Type Checking

```bash
npm run lint              # ESLint (flat config)
npm run lint:fix          # Auto-fix
npm run format            # Prettier
npm run typecheck         # TypeScript --noEmit
npm run preflight         # Full quality gate
```

### Lint Rules Summary

- **No `any` types** — `@typescript-eslint/no-explicit-any: error`
- **No `console`** — `no-console: error` (with allowlists for specific files)
- **No `require()`** — ESM import preferred
- **No internal module imports** across packages
- **Consistent type imports** — `@typescript-eslint/consistent-type-imports`
- **No default exports** — Prefer named exports

---

## CI Pipeline

The `preflight` script represents the full CI quality gate:
```
clean → npm ci → format → lint → build → typecheck → test
```
