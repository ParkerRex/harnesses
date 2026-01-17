# Server to TUI Event Pipeline (Codex CLI)

This file documents the full server-side pipeline that drives the TUI:
from tool invocation, through protocol events, to TUI renderable cells.

## 1) Core tool invocation -> EventMsg
Tool and exec calls are emitted by codex-core as `EventMsg` values.

### update_plan tool
- Tool schema and handler:
  `src/coding-agents/codex/codex-rs/core/src/tools/handlers/plan.rs:20`
- Emits `EventMsg::PlanUpdate` with parsed args:
  `src/coding-agents/codex/codex-rs/core/src/tools/handlers/plan.rs:100`
- Plan arg types are in:
  `src/coding-agents/codex/codex-rs/protocol/src/plan_tool.rs:6`

### Exec commands
- `ToolEmitter::shell/unified_exec` attaches parsed command metadata and emits
  ExecCommand events.
  `src/coding-agents/codex/codex-rs/core/src/tools/events.rs:90`
- Parsing is done by `parse_command` in:
  `src/coding-agents/codex/codex-rs/core/src/parse_command.rs:30`

### EventMsg definitions
The canonical `EventMsg` enum is defined in:

- `src/coding-agents/codex/codex-rs/protocol/src/protocol.rs:720`

This is the event stream the TUI consumes.

## 2) Exec event processor -> Thread events
The `EventProcessorWithJsonOutput` converts protocol events into thread items
and item lifecycle events.

- Plan updates become `TodoListItem` thread items:
  `src/coding-agents/codex/codex-rs/exec/src/event_processor_with_jsonl_output.rs:438`
- MCP tool calls become `McpToolCallItem` thread items:
  `src/coding-agents/codex/codex-rs/exec/src/event_processor_with_jsonl_output.rs:250`

Thread item types live in:

- `src/coding-agents/codex/codex-rs/exec/src/exec_events.rs:233`

## 3) TUI consumption of EventMsg
The TUI consumes `EventMsg` directly (from codex-core) rather than thread items
(used by the app-server protocol).

- The dispatch table is in `dispatch_event_msg`:
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2250`

Examples:

- `TurnStarted` -> `on_task_started` (status bar + running state)
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2287`
- `PlanUpdate` -> `on_plan_update` (PlanUpdateCell in history)
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2310`
- `ExecCommandBegin/End` -> `on_exec_command_begin/end` (ExecCell grouping)
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2321`

## App-server Protocol (JSON-RPC) for non-TUI clients
If you are building a non-TUI client, the app-server protocol exposes
notifications with typed payloads that mirror the same concepts.

### Server notifications
- The server notification enum and wire names live in:
  `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/common.rs:405`

Key notifications for UI:

- `turn/started`, `turn/completed`, `thread/started`
- `turn/plan/updated`
- `item/started`, `item/completed`
- `item/agentMessage/delta`, `item/reasoning/textDelta`
- `item/commandExecution/outputDelta`, `item/commandExecution/terminalInteraction`

These are defined in:

- `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/common.rs:544`

### Thread item schema (what clients render)
The v2 protocol defines a unified `ThreadItem` with:

- `CommandExecution` (exec + parsed command actions)
- `McpToolCall`
- `FileChange`, `WebSearch`, `ImageView`, `AgentMessage`, `UserMessage`

`ThreadItem` definition:

- `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/v2.rs:1577`

### Parsed command mapping for UI summaries
`CommandAction` mirrors `ParsedCommand` and is used by
`CommandExecution.command_actions`.

- `CommandAction` definition + mapping:
  `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/v2.rs:653`

### Plan updates in v2 protocol
Plan updates for client UIs are represented as
`TurnPlanUpdatedNotification`:

- `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/v2.rs:1793`

### Approval flow (for exec/patches)
The JSON-RPC protocol defines approval request/response flows:

- `item/commandExecution/requestApproval`
- `item/fileChange/requestApproval`

Definition:

- `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/common.rs:488`

## Persistence and Resume
The system persists a subset of events and rebuilds turn history on resume.

- Persistence policy:
  `src/coding-agents/codex/codex-rs/core/src/rollout/policy.rs:36`
- Rebuild turns from persisted events:
  `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/thread_history.rs:13`

## Notes for Translation to Convex + Next/React
For a Convex-backed app, you will likely:

- Store events in a Convex table keyed by thread and turn.
- Stream new events to the client via Convex subscriptions.
- Build UI from the event stream (or materialized thread items) similar to
  `ThreadItem` in the v2 protocol.
- Use a local `activeCell` for in-flight exec/tool entries to minimize reflow.
