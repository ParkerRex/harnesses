# TUI Task UI Elements (Tool Calls, Plans, Status)

This file documents the TUI components that render tool calls, task lists,
output truncation, and the status indicator. It focuses on "what shows up" and
"why" from a UI perspective.

## History Cells (Renderable Units)
The TUI renders conversation and tool activity as a stream of `HistoryCell`
objects. Key cell types:

- `UserHistoryCell` for user prompts.
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:163`
- `AgentMessageCell` for streamed agent text.
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:261`
- `ReasoningSummaryCell` for transcript-only reasoning summaries.
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:190`
- `ExecCell` for tool calls and command outputs.
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/model.rs:28`
- `PlanUpdateCell` for update_plan checkbox lists.
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:1506`
- `McpToolCallCell` for MCP tool call rows.
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:1041`

## Tool Call Summaries ("Exploring/Explored")
The TUI groups tool calls into a single cell when they are considered
"exploring" (read/list/search actions).

Detection:

- A call is "exploring" if it is not a user shell command and all parsed
  commands are `Read`, `ListFiles`, or `Search`.
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/model.rs:140`

Rendering:

- The header line is "Exploring" while active, and "Explored" when complete.
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/render.rs:255`
- Consecutive read calls are coalesced and rendered as a single "Read" line
  with a comma-separated list of file names.
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/render.rs:271`

This is the exact UI you see in the screenshots:

- `• Explored` with `Read`/`List`/`Search` rows below.

### How "Exploring" vs "Explored" is chosen
The label is chosen purely by whether the grouped exec calls are still running:

- `ExecCell::is_active()` returns true if any call has no output yet.
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/model.rs:109`
- The header string uses `is_active()`:
  - Active => "Exploring"
  - Completed => "Explored"
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/render.rs:255`

### Reading vs writing/generating tools
Only `Read`, `ListFiles`, and `Search` commands are treated as "exploring."
Anything else (write/generate/compile/build/etc.) renders as a normal exec
command cell (the "Running"/"Ran" layout).

This behavior comes from two rules:

1) "Exploring" grouping requires **all parsed commands** to be one of
   `Read|ListFiles|Search` and the exec source is not `UserShell`.
   `src/coding-agents/codex/codex-rs/tui/src/exec_cell/model.rs:140`
2) If the parser yields `Unknown` or an empty parsed list, the UI uses the
   standard command display renderer instead of the "Exploring" grouping.
   `src/coding-agents/codex/codex-rs/tui/src/exec_cell/render.rs:358`

## Command Display (Non-exploring exec calls)
When a tool call is not "exploring," the TUI renders a command display cell.

- The command header text varies by source:
  - "Running" if active
  - "You ran" for user shell commands
  - "Ran" for others
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/render.rs:360`

Output handling:

- Output is wrapped and truncated to avoid flooding the viewport.
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/render.rs:399`
- Tool calls are truncated to `TOOL_CALL_MAX_LINES`, while user shell commands
  can show more lines (`USER_SHELL_TOOL_CALL_MAX_LINES`).
  `src/coding-agents/codex/codex-rs/tui/src/exec_cell/render.rs:27`

## Plan/Todo List UI
The plan update UI is rendered as a checkbox list:

- `PlanUpdateCell` produces the header "Updated Plan" and lines with
  checkboxes based on step status.
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:1506`

Status mapping:

- Completed -> "✔" and crossed-out style.
- In progress -> "□" with cyan/bold style.
- Pending -> dimmed "□".
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:1528`

## Status Indicator (Shimmer + Elapsed + Interrupt)
The status indicator is a dedicated widget in the bottom pane.

- The widget is `StatusIndicatorWidget`.
  `src/coding-agents/codex/codex-rs/tui/src/status_indicator_widget.rs:33`
- It renders:
  - Spinner
  - Shimmered header text
  - Elapsed time
  - "esc to interrupt" hint
  `src/coding-agents/codex/codex-rs/tui/src/status_indicator_widget.rs:198`

Shimmer implementation:

- `shimmer_spans` computes a moving highlight band over the header text.
  `src/coding-agents/codex/codex-rs/tui/src/shimmer.rs:21`

Elapsed formatting:

- `fmt_elapsed_compact` formats time (e.g., 1m 03s).
  `src/coding-agents/codex/codex-rs/tui/src/status_indicator_widget.rs:49`

## Final Message Separator
When switching between tool calls and streaming agent messages, a final message
separator is inserted to clearly divide transcript sections:

- `FinalMessageSeparator` is in `history_cell.rs`.
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:1648`
- Insertion logic is in `handle_streaming_delta`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1304`

## UI Snapshot References (golden outputs)
These snapshots show the exact expected output for core UI elements:

- Plan update: `codex_tui__history_cell__tests__plan_update_without_note_snapshot.snap:5`
- Exploring summary: `codex_tui__chatwidget__tests__exploring_step2_finish_ls.snap:5`
- MCP tool row: `codex_tui__history_cell__tests__active_mcp_tool_call_snapshot.snap:6`
- Status bar: `codex_tui__status_indicator_widget__tests__renders_with_working_header.snap:5`

## Notes for Translation to React/Convex
- Model the same cell taxonomy as a discriminated union (User, Agent, Exec,
  PlanUpdate, Status, etc.).
- Treat exec calls as either a grouped "exploring" list or a command/output
  block, based on parsed command metadata.
- Use CSS or canvas for shimmer; keep elapsed time logic separate so it can
  be paused/resumed with task state.
