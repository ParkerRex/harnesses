# Eigent Project Snapshot

## 0. Snapshot metadata

- Snapshot UTC: 2026-01-15T16:33:33Z
- Snapshot source ref (commit or tag): 6d83cb7
- Snapshot cadence (e.g., every 3 days): every 3 days (or manual dispatch)
- Snapshot scope (what changed triggers a snapshot): doc updates in `docs/coding-agents/eigent/`

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
  - `models-prompting.md`
  - `tools.md`
  - `context-compaction.md`
  - `persistence-sharing.md`
  - `safety-failure-modes.md`
- `diagrams/` - Mermaid diagrams:
  - `system-overview.mmd`

## 2. Tech stack

- Languages (primary + secondary): TypeScript/JavaScript (Electron + renderer UI), Python (FastAPI backend + server), plus HTML/CSS (Tailwind) and Markdown/JSON.
- Runtimes/frameworks: Electron desktop shell, React/Zustand + Vite renderer, FastAPI + Uvicorn backend runtime, optional FastAPI server for persistence/sharing.
- Build/package tooling: npm scripts, Vite, TypeScript, Electron Builder, Tailwind/PostCSS, Vitest; Python backend uses `uv` with `pyproject.toml`.
- Infra/services (DB, queue, cache, external APIs): local config under `~/.eigent` (MCP config), optional server SQL store for chat persistence, external model APIs wired via `ModelFactory`.
- Observability (logs/metrics/tracing): SSE step events streamed to the renderer; no dedicated metrics/tracing doc in this manual set.

## 3. Repo stats (as of 2026-01-15)

- Commits: 1674
- Contributors: 39
- Default branch: main
- License: Apache License 2.0 (LICENSE); `package.json` still lists MIT.
- Release cadence (tags or releases): tags present; latest reachable tag: `v0.0.77`

## 4. Repo shape & entry points (10-15 min)

- Entry points (where execution starts):
  - Electron main boot sequence + backend spawn: `electron/main/init.ts`.
  - Renderer entry: `src/main.tsx`.
  - Backend app entry: `backend/main.py` (FastAPI).
  - Optional server API: `server/app/__init__.py` (FastAPI).
- "Happy path" flow (what a typical run does):
  - User prompt -> renderer store -> backend task execution -> model/tool calls -> SSE step events -> UI updates -> optional server persistence.
- Monorepo or single-purpose: single-purpose desktop app repo with bundled backend runtime + optional server.
- Key top-level dirs and why they matter:
  - `src/` (renderer UI, chat state, SSE client)
  - `electron/` (main process, backend spawn, OS integration)
  - `backend/` (FastAPI runtime, CAMEL workforce, toolkits)
  - `server/` (optional persistence + sharing backend)
  - `resources/`, `config/`, `public/` (app assets/config)
- Build/run files (README, package.json, go.mod, Makefile, etc.):
  - `README.md`, `package.json`, `vite.config.ts`, `electron-builder.json`, `tsconfig.json`, `tailwind.config.js`, `backend/pyproject.toml`.

## 5. Repo file tree (top-level)

- `reference/file-tree.md` (tracked files, depth-limited)

## 6. Architecture in one whiteboard diagram

- Diagram: `diagrams/system-overview.mmd`
- Flow: User/Event -> Renderer -> Electron main -> Backend runtime -> Model/Tool execution -> SSE steps -> Renderer (+ optional server persistence).
- Boundaries (where data changes shape):
  - Renderer <-> backend via HTTP + SSE (`chat_controller`).
  - Backend <-> toolkits via agent/toolkit listeners.
  - Backend <-> server persistence via chat step records.
- Stateful vs stateless parts:
  - Stateful: project/task state in backend services; UI chat store; server DB records.
  - Stateless-ish: per-call toolkits and model adapters (still side-effectful).
- Sync vs async:
  - Backend streams SSE updates asynchronously while tasks/tools run.

## 7. Data flow > code flow

- Where data enters the system: UI chat input + model selection (`src/store/chatStore.ts`, `src/store/authStore.ts`).
- Validation -> transformation -> persistence -> response:
  - Input becomes a task (`backend/app/service/task.py`) -> tool/model execution -> step events -> UI renders -> optional server persistence (`server/app/model/chat/*`).
- Domain objects vs DTOs:
  - Domain: `project_id`/`task_id` sessions, task state, toolkits.
  - DTO-ish: SSE step events + server chat models (ChatHistory/ChatStep/ChatSnapshot).
- Where schemas live: `backend/app/model/chat.py` and `server/app/model/chat/*`.
- Ownership (who "owns" each piece of data):
  - Backend owns task state + conversation history; UI owns presentation state; server owns persisted step logs.
- Where data mutates:
  - Conversation history updates in `backend/app/service/chat_service.py`.
  - Tool events attach `process_task_id` in `backend/app/utils/listen/toolkit_listen.py`, resolved in `src/store/chatStore.ts`.
- What assumptions are baked in:
  - Context capped at 100,000 chars with no automatic compaction.
  - Tool events are keyed by task/process IDs for UI mapping.

## 8. Tests tell you what matters

- What has tests vs what doesn't: renderer tests under `test/` (unit/integration/mocks); backend tests under `backend/tests`.
- Edge cases covered: tool execution + UI flow tests live under `test/integration`; backend tests validate service logic.
- Mock-heavy zones (signals slowness/instability): UI tests rely on `test/mocks`; backend uses fixtures in `backend/tests`.
- Snapshot vs behavioral tests: primarily behavioral/unit tests; no snapshot tests called out in the doc set.
- What breaks most often (if known): not documented in this manual set.

## 9. Error handling & logging

- Structured logging or ad-hoc logs: errors are surfaced via SSE step events; logging paths are not documented here.
- Centralized error handling:
  - SSE timeout after 10 minutes idle (`backend/app/controller/chat_controller.py`).
  - `budget_not_enough` and tool failures surfaced from agent execution (`backend/app/utils/agent.py`).
  - `context_too_long` emitted when history exceeds 100,000 chars (`backend/app/service/chat_service.py`).
- Retry logic / timeouts / circuit breakers: SSE idle timeout; tool retry logic not documented here.
- If this breaks in prod, what do we see?
  - UI receives failure events (timeouts/budget/context), and input can be disabled for overlong context.

## 10. Observability & operations (optional)

- Metrics/tracing tooling: not documented in the manual set.
- Feature flags / env config: backend env loader in `backend/app/component/environment.py`; MCP config in `~/.eigent/mcp.json`.
- Deployment shape (local, cloud, etc.): local Electron desktop app with local FastAPI runtime; optional server for persistence/sharing.

## 11. Sharp edges & footguns

- Global state: project/task state lives in backend services and renderer stores; server DB holds shared history.
- Hidden side effects: toolkits can mutate local files/apps and remote services.
- Implicit conventions: tool events must carry consistent `process_task_id` to map into UI subtasks.
- Magic env flags: MCP config path under `~/.eigent/mcp.json` is assumed.
- Temporal coupling (this must run before that):
  - Backend must be running before UI can stream SSE steps.
  - Context overflow disables input without compaction.
