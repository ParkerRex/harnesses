# TUI Conversation Flow and Streaming (Codex CLI)

This file documents, in detail, how the Codex TUI handles conversation state,
streaming agent output, reasoning status headers, turn lifecycle, and interrupts.
All references are to the Codex code in this repository.

## Core UI State in ChatWidget
The conversation UI is driven by `ChatWidget` state and an event dispatcher.
Key fields (with why they exist):

- `active_cell`: a mutable in-progress history cell (often an exec cell) that
  streams and updates in place. `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:371`
- `stream_controller`: streams agent text by newline and animates commits.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:395`
- `agent_turn_running`: turn lifecycle flag, distinct from MCP startup.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:402`
- `reasoning_buffer` + `full_reasoning_buffer`: track reasoning deltas and
  transcript-only reasoning summaries. `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:415`
- `current_status_header`: current shimmer header text shown in the status bar.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:419`
- `queued_user_messages`: user inputs queued while a turn is running.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:430`

## Event Intake and Dispatch
The TUI consumes core `EventMsg` values via `handle_codex_event`, which calls
`dispatch_event_msg`.

- `handle_codex_event` is the live event entry point.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2245`
- `dispatch_event_msg` routes each `EventMsg` to the correct handler.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2250`

Replay behavior for resumed sessions:

- `replay_initial_messages` replays a safe subset of events from persisted
  history to render a transcript on resume.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2230`

## User Input Pipeline (Client side)
User input is collected in the UI and sent to core via `Op::UserInput`.

- Messages are queued if a turn is running (or review mode is active).
  `queue_user_message` in `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2151`
- Messages are submitted via `submit_user_message`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2160`

Key behaviors inside `submit_user_message`:

- Special case `!cmd`: executes a local shell command instead of calling the
  model. `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2168`
- Adds images and skill mentions to `UserInput` items before sending.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2186`
- Sends `Op::UserInput` and also `Op::AddToHistory` for cross-session history.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2204`
- Renders the user text in the transcript via `new_user_prompt`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2222`

## Agent Output Pipeline (Streaming)
Agent output can arrive as deltas or as a final message. The UI handles both.

### AgentMessageDelta
- Deltas are routed to `on_agent_message_delta` and then to
  `handle_streaming_delta`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:647`

`handle_streaming_delta` does the following:

- Flushes any active exec cell (`flush_active_cell`) before streaming text.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1300`
- Initializes `StreamController` on first delta.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1304`
- Pushes deltas into `StreamController` and starts a commit animation.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1317`

### StreamController details
`StreamController` manages newline-gated streaming and commit animation.

- It uses `MarkdownStreamCollector` to render markdown and only commit fully
  completed lines. `src/coding-agents/codex/codex-rs/tui/src/streaming/controller.rs:7`
- `MarkdownStreamCollector` appends deltas, commits only lines ending with
  newline, and finalizes on end-of-stream.
  `src/coding-agents/codex/codex-rs/tui/src/markdown_stream.rs:7`

Commit loop and animation:

- `StartCommitAnimation` and `CommitTick` are AppEvents.
  `src/coding-agents/codex/codex-rs/tui/src/app_event.rs:49`
- The periodic commit tick calls `StreamController::on_commit_tick`, which
  commits at most one line per tick.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1253`

### Final AgentMessage
- When a final `AgentMessage` arrives, the TUI calls `on_agent_message`. If a
  stream controller is active, the final message is redundant and is not
  streamed again.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:636`

## Reasoning Deltas and Status Header
Reasoning text is not streamed to the transcript. Instead it drives the shimmer
status header and optionally a reasoning summary cell.

- Reasoning deltas are appended to `reasoning_buffer`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:651`
- The first bold segment (Markdown `**...**`) is extracted and becomes the
  status header. `extract_first_bold`:
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:4231`
- On reasoning final, `new_reasoning_summary_block` creates a transcript-only
  reasoning summary cell (if applicable).
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:1622`

### Reasoning block lifecycle (full detail)
The TUI accumulates reasoning in two buffers:

- `reasoning_buffer`: the *current* reasoning block (used to extract headers).
- `full_reasoning_buffer`: the accumulated reasoning text across sections.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:415`

The reasoning flow is:

1) **Delta arrives** (`AgentReasoningDelta` or `AgentReasoningRawContentDelta`):
   - Append delta to `reasoning_buffer`.
   - If no unified-exec wait is in progress, extract the first bold header and
     update the status header.
   `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:651`

2) **Section break** (`AgentReasoningSectionBreak`):
   - Move `reasoning_buffer` into `full_reasoning_buffer`, add a blank line,
     clear `reasoning_buffer`.
   `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:685`

3) **Reasoning final** (`AgentReasoning` or `AgentReasoningRawContent`):
   - Append `reasoning_buffer` to `full_reasoning_buffer`.
   - Emit a reasoning summary/history cell via `new_reasoning_summary_block`.
   - Clear both buffers.
   `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:672`

### Reasoning summary rendering rules
`new_reasoning_summary_block` decides whether the summary appears on screen:

- If it finds a bold header plus additional content, it renders a visible
  `ReasoningSummaryCell`.
- Otherwise, it creates a **transcript-only** summary (not rendered on-screen).
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:1622`

The `ReasoningSummaryCell` rendering behavior is defined here:

- `ReasoningSummaryCell` is transcript-only if `transcript_only == true`.
  `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:234`

### Status header precedence
Reasoning-derived headers are skipped when unified-exec waiting is active:

- When `unified_exec_wait_streak` is set, the status header is left unchanged.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:657`

### Buffer resets
Reasoning buffers are cleared at key points:

- On task start: `full_reasoning_buffer` and `reasoning_buffer` cleared.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:701`
- On reasoning final: both buffers cleared after summary emission.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:680`

## Turn Lifecycle and Status Bar Visibility
- `TurnStarted` triggers `on_task_started`, which sets running state and shows
  the status indicator with a default header "Working".
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:694`
- `TurnComplete` triggers `on_task_complete`, which stops running state, clears
  streaming buffers, and may start the next queued input.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:706`

The bottom pane owns the status indicator:

- Running state toggles the status indicator on/off.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/mod.rs:360`

## When the Todo/Plan UI Appears Inline
Plan/todo UI is rendered inline in the transcript when the TUI receives a
`PlanUpdate` event. The flow is direct and synchronous, not delayed until the
turn ends:

1) The model calls `update_plan` -> core emits `EventMsg::PlanUpdate`.
   `src/coding-agents/codex/codex-rs/core/src/tools/handlers/plan.rs:100`
2) The TUI receives `EventMsg::PlanUpdate` and immediately calls
   `on_plan_update`, which inserts a `PlanUpdateCell` into history.
   `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2310`
3) `add_to_history` flushes any in‑flight exec cell and inserts the plan cell,
   so the todo list appears inline in the transcript, in order.
   `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2131`
4) The cell renders inline with other history cells as soon as it is inserted.
   `src/coding-agents/codex/codex-rs/tui/src/history_cell.rs:1506`

This means the todo list shows up *during* the turn (as soon as the tool call
is emitted), not only after the final agent message.

## Interaction with New User Messages
If the user types a new message while a turn is running, the TUI queues it
instead of sending immediately:

- The queueing path is `queue_user_message`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2151`
- While queued, the UI shows the message in the “queued” preview area.
- When the running turn completes, `maybe_send_next_queued_input` submits the
  next queued message.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2476`

This behavior prevents user input from interleaving with an in-progress turn
and keeps the transcript ordering deterministic.

## Active Cell and History Flow
The TUI uses an "active cell" to show in-progress exec/tool calls and flushes
it into history at the right times.

- `flush_active_cell` sends the active cell to history and sets
  `needs_final_message_separator`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2131`
- Any time a new history cell is added, the active cell is flushed first.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2142`

## Interrupt Queue (Ordering Guarantees)
To avoid reordering events during streaming, the TUI uses an interrupt queue:

- When streaming, event handlers are deferred to preserve FIFO ordering.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1274`
- When streaming finishes, the queue is flushed.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1290`

## Turn Interrupt (Esc) Behavior
When a turn is interrupted (ESC), the UI:

- Finalizes active cells as failed (red X), resets running state, and inserts a
  gentle error prompt.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:939`

## Summary: Event to UI Mapping (Conversation)
Below is the key event mapping for conversation flow:

- `AgentMessageDelta` -> `handle_streaming_delta` -> StreamController ->
  `AgentMessageCell` lines.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:647`
- `AgentMessage` -> `on_agent_message` -> finalize stream (if any) and flush.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:636`
- `AgentReasoningDelta` -> update status header from bold segment.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:651`
- `TurnStarted` -> running state + "Working" header.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:694`
- `TurnComplete` -> stop running + clear state + queue next input.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:706`

## Notes for Translation to React/Convex
If you are implementing this in React/Convex:

- Model an `activeCell` in client state for in-flight exec/tool UI.
- Use a streaming buffer with newline-gated markdown rendering (similar to
  `MarkdownStreamCollector`) to avoid reflow on partial lines.
- Treat reasoning deltas as a status header source, not transcript content.
- Maintain a `taskRunning` flag to show/hide the status bar.
- Queue user messages while `taskRunning` is true and flush them in order on
  completion.
