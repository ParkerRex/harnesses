# Cline Project Snapshot

## 0. Snapshot metadata

- Snapshot UTC: 2026-01-15T16:36:41Z
- Snapshot source ref (commit or tag): 963abc190
- Snapshot cadence (e.g., every 3 days): every 3 days (or manual dispatch)
- Snapshot scope (what changed triggers a snapshot): doc updates in `docs/coding-agents/cline/`

## 1. Doc set (existing artifacts)

- `README.md` - TL;DR, entry points, and where to start reading.
- `architecture-overview.md` - System-level summary + primary data/control flows.
- `key-patterns.md` - Repeated architectural patterns, conventions, and invariants.
- `comparison.md` - How this project differs from other agents in the repo.
- `onboarding.md` - Legacy stub that points to reference docs.
- `repo-map.md` - Legacy stub that points to reference docs.
- `CHANGELOG-upstream.md` - Upstream project changes since last sweep.
- `CHANGELOG-docs.md` - Documentation updates made in this repo.
- `reference/directory-map.md` - Top-level folders + what they own.
- `reference/file-tree.md` - Top-level file tree (tracked files, depth-limited).
- `reference/key-files.md` - High-leverage files for core flows.
- `reference/mental-model.md` - Mental model + suggested reading path.
- `reference/project-snapshot.md` - This snapshot (overview + gaps).
- `subsystems/` - Deep dives:
  - `core-runtime.md`
  - `tools.md`
  - `models-prompting.md`
  - `context-compaction.md`
  - `persistence.md`
  - `safety-failure-modes.md`
  - `observability.md`
- `diagrams/` - Mermaid diagrams:
  - `system-overview.mmd`
  - `tools-lifecycle.mmd`

## 2. Tech stack

- Languages (primary + secondary): TypeScript/JavaScript (extension host + shared core + webview UI), Go (CLI), plus Markdown, JSON, and Proto assets.
- Runtimes/frameworks: VS Code extension host API, React + Vite for the webview UI, Node tooling, Go CLI runtime.
- Build/package tooling: npm, esbuild, TypeScript, Biome, Changesets, Vite/Vitest, Go modules (`go.work`, `cli/go.mod`), Buf/proto tooling (`buf.yaml` + `scripts/build-*-proto.mjs`).
- Infra/services (DB, queue, cache, external APIs): local task persistence under `~/.cline/data/tasks/<taskId>` plus global settings; external model providers configured per user settings; MCP server integrations.
- Observability (logs/metrics/tracing): telemetry and logging via `src/services/telemetry/TelemetryService.ts` and `src/services/logging/Logger.ts`, with PostHog/OpenTelemetry providers.

## 3. Repo stats (as of 2026-01-15)

- Commits: 4491
- Contributors: 277
- Default branch: main
- License: Apache 2.0
- Release cadence (tags or releases): tags present; latest reachable tag: `v3.49.1`

## 4. Repo shape & entry points (10-15 min)

- Entry points (where execution starts):
  - VS Code extension activation: `src/extension.ts`.
  - Webview bridge: `src/core/webview/WebviewProvider.ts`.
  - Controller orchestration: `src/core/controller/index.ts`.
  - Task loop: `src/core/task/index.ts`.
  - CLI entry: `cli/cmd/cline/main.go`.
- "Happy path" flow (what a typical run does):
  - Webview input -> controller/task loop -> prompt assembly -> model response -> tool execution + approvals -> state persisted -> UI updated.
- Monorepo or single-purpose: monorepo (extension host, webview UI, CLI, test tooling).
- Key top-level dirs and why they matter:
  - `src/` (extension host runtime, controller, task loop, tools, storage)
  - `webview-ui/` (React webview UI)
  - `cli/` (Go CLI + packaging)
  - `proto/` (protocol definitions)
  - `testing-platform/` (test harness utilities)
- Build/run files (README, package.json, go.mod, Makefile, etc.): `README.md`, `package.json`, `esbuild.mjs`, `webview-ui/package.json`, `go.work`, `cli/go.mod`.

## 5. Repo file tree (top-level)

- `reference/file-tree.md` (tracked files, depth-limited)

## 6. Architecture in one whiteboard diagram

- Diagram: `diagrams/system-overview.mmd`
- Flow: Webview UI -> Extension Host (controller/task) -> Model Provider -> Tool handlers/MCP -> Persistence -> UI
- Boundaries (where data changes shape):
  - Webview messages <-> extension host controller/task state.
  - Extension host <-> model provider request/response payloads.
  - Extension host <-> tool handlers/MCP servers.
  - Extension host <-> disk persistence.
- Stateful vs stateless parts:
  - Stateful: task/controller state, context manager, persistence state.
  - Stateless-ish: tool handlers and provider calls per turn.
- Sync vs async:
  - Task loop is sequential; tool and provider IO is async within a turn.

## 7. Data flow > code flow

- Where data enters the system: webview chat input, commands, and added context attachments.
- Validation -> transformation -> persistence -> response:
  - Input normalized -> prompt assembly -> model output parsed -> tool execution -> message updates -> persisted task state -> UI render.
- Domain objects vs DTOs:
  - Domain: task/controller state, tool calls, context window management.
  - DTOs: webview message payloads, provider request/response structs.
- Where schemas live: shared proto models in `src/shared/proto/` and task models in `src/core/`.
- Ownership (who "owns" each piece of data): extension host owns task state; webview UI owns presentation state; providers own remote responses.
- Where data mutates: `ContextManager` trimming/compaction and `ToolExecutor` tool updates.
- What assumptions are baked in: explicit approvals for tool execution, bounded context windows, per-task persistence.

## 8. Tests tell you what matters

- What has tests vs what doesn't: VS Code extension tests under `tests/`, unit tests via mocha, webview UI tests via Vitest, and Playwright E2E coverage.
- Edge cases covered: tool execution paths, provider routing, and UI interactions (see subsystem docs).
- Mock-heavy zones (signals slowness/instability): provider and tool tests frequently mock external APIs.
- Snapshot vs behavioral tests: primarily behavioral tests, with some snapshot-style UI assertions.
- What breaks most often (if known): tool integration edges and provider configuration mismatches.

## 9. Error handling & logging

- Structured logging or ad-hoc logs: structured logs via `src/services/logging/Logger.ts`.
- Centralized error handling: task/controller loop captures tool and provider errors and surfaces them in the UI.
- Retry logic / timeouts / circuit breakers: handled per tool/provider and configured by the runtime.
- If this breaks in prod, what do we see?
  - Task errors in the UI, plus telemetry counters for failures.

## 10. Observability & operations (optional)

- Metrics/tracing tooling: telemetry service with PostHog and OpenTelemetry providers.
- Feature flags / env config: VS Code settings, per-task state, and optional cline rules/hooks stored on disk.
- Deployment shape (local, cloud, etc.): distributed as a VS Code extension with optional CLI and standalone builds.

## 11. Sharp edges & footguns

- Global state: tasks persist to disk; deleting task state loses history.
- Hidden side effects: approved tools and MCP servers can mutate external systems.
- Implicit conventions: webview message schemas and tool state transitions must stay in sync.
- Magic env flags: provider API keys and runtime settings are required for tool/model access.
- Temporal coupling (this must run before that): compaction and context trimming must occur before the next model call.
