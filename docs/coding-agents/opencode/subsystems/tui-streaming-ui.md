# TUI Streaming + Thinking/Todos UI

This doc captures how the CLI TUI streams events and renders reasoning
("Thinking") and todos. Paths are relative to the
`src/coding-agents/opencode/` submodule.

Diagram: `../diagrams/tui-streaming-ui.mmd`.

## Streaming pipeline (CLI TUI)

- TUI thread spawns a worker; the worker hosts the server and SDK client.
  - `packages/opencode/src/cli/cmd/tui/thread.ts`
  - `packages/opencode/src/cli/cmd/tui/worker.ts`
- Worker uses `createOpencodeClient` with in-process `Server.App().fetch`
  and subscribes to `sdk.event.subscribe`. Each event is forwarded to the UI
  thread over RPC (`Rpc.emit("event", ...)`).
- If a server is started (port/hostname set), the TUI uses HTTP + SSE
  instead of RPC events.
- SDK events are batched in 16ms windows to avoid render thrash and keep
  latency low.
  - `packages/opencode/src/cli/cmd/tui/context/sdk.tsx`
- Server SSE endpoints:
  - `/event` streams Bus events (used by the SDK).
  - `/global/event` streams GlobalBus events (includes directory context).
  - Both send a 30s heartbeat.
  - `packages/opencode/src/server/server.ts`
  - `packages/opencode/src/bus/index.ts`
  - `packages/opencode/src/bus/global.ts`

## Store sync (events -> UI state)

- `context/sync.tsx` listens to SDK events and updates the Solid store.
- Key events for streaming UI:
  - `message.updated`
  - `message.part.updated`
  - `message.part.removed`
  - `todo.updated`
  - `session.updated`, `session.status`
- Todo state is stored per-session in `store.todo[sessionID]`.
  - `packages/opencode/src/cli/cmd/tui/context/sync.tsx`

## Part rendering (Thinking + text)

- Parts are mapped by type: `text`, `tool`, `reasoning`.
  - `packages/opencode/src/cli/cmd/tui/routes/session/index.tsx`
- Reasoning part ("Thinking")
  - Renders only when `thinking_visibility` is enabled.
  - Uses a left border via `SplitBorder.customBorderChars`.
  - Markdown renderer with `streaming={true}` and subtle syntax theme.
  - Prefixes content with `_Thinking:_` and strips `[REDACTED]` tokens.
- Text part
  - Markdown renderer with `streaming={true}` and standard syntax theme.
  - Trimmed text, left padding, and a small top margin.

## Todos UX

- `todowrite` tool updates the per-session todo list and publishes
  `todo.updated`.
  - `packages/opencode/src/tool/todo.ts`
  - `packages/opencode/src/session/todo.ts`
- Inline (message stream)
  - `TodoWrite` renders a `BlockTool` titled `# Todos` when metadata is
    available.
  - While pending, it renders an inline "Updating todos..." row.
  - `packages/opencode/src/cli/cmd/tui/routes/session/index.tsx`
- Sidebar
  - Todo section appears when there are non-completed items.
  - Collapses/expands when more than 2 items exist.
  - `packages/opencode/src/cli/cmd/tui/routes/session/sidebar.tsx`
- Todo item rendering
  - Status values: `pending`, `in_progress`, `completed`, `cancelled`.
  - Glyphs: in_progress uses a bullet dot (U+2022), completed uses a
    checkmark (U+2713), others render a blank.
  - Color: in_progress uses `theme.warning`; others use `theme.textMuted`.
  - Word-wrap enabled for long items.
  - `packages/opencode/src/cli/cmd/tui/component/todo-item.tsx`

## BlockTool layout (todos and other tool panels)

- Left border + panel background (`backgroundPanel`), hover swaps to
  `backgroundMenu`.
- Padding top/bottom 1, left 2; top margin 1; title in `textMuted`.
- Errors render below the content in `theme.error`.
- `packages/opencode/src/cli/cmd/tui/routes/session/index.tsx`
