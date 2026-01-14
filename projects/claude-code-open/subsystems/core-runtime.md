# Core Runtime (Loop + Sessions)

Claude Code Open runs a local CLI/TUI that drives a ConversationLoop around a
persistent session model.

## Core components

- CLI entry and flags: `src/cli.ts`.
- Conversation loop: `src/core/loop.ts`.
- Session state and usage tracking: `src/core/session.ts`.
- Session list/load/fork/merge: `src/session/index.ts`.

## Message lifecycle (high level)

1) CLI/TUI input appends a user message to the session.
2) System prompt is built from templates and attachments.
3) LLM request sent via Anthropic SDK.
4) Tool calls emitted as `tool_use` blocks.
5) Tools execute via the ToolRegistry with permission checks.
6) Tool results are injected as `tool_result` messages.
7) Session auto-saves to disk.

Key files:

- `src/core/loop.ts`
- `src/core/client.ts`
- `src/core/session.ts`
