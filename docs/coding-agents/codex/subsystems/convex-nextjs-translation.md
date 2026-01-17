# Translation Guide: Codex TUI Concepts -> Convex + Next.js/React

This document maps Codex CLI conversation and task UI concepts to a Convex +
Next.js/React implementation. It is designed to be used alongside:

- `docs/coding-agents/codex/task-ui-and-tool-calls.md`
- `docs/coding-agents/codex/subsystems/tui-conversation-flow.md`
- `docs/coding-agents/codex/subsystems/server-to-tui-event-pipeline.md`

## Conceptual Mapping (Codex -> Your App)
| Codex concept | What it means | Convex/React analog |
| --- | --- | --- |
| `EventMsg` stream | Ordered event feed from core | Convex `events` table + subscription |
| `TurnStarted` / `TurnComplete` | Turn lifecycle boundaries | `turns` table + `status` field |
| `AgentMessageDelta` | Streaming text | `events` rows or `message_deltas` table |
| `PlanUpdate` | Task list updates | `plan_updates` events or materialized `plan_steps` |
| `ParsedCommand` | Tool-call summary metadata | Store `command_actions` on exec events |
| `active_cell` | In-flight tool call or exec UI | React local state for transient rendering |
| Status indicator (shimmer) | Task running + header | React component driven by turn status + reasoning header |

## Suggested Convex Data Model
This model mirrors Codex event-driven UI while remaining simple.

### Tables
1) `threads`
- `_id`
- `title` (optional)
- `createdAt`
- `updatedAt`

2) `turns`
- `_id`
- `threadId` (index)
- `status` (`running|completed|failed|aborted`)
- `startedAt`, `completedAt`

3) `events` (primary timeline source)
- `_id`
- `threadId` (index)
- `turnId` (index)
- `sequence` (monotonic integer or timestamp-based ordering)
- `type` (string enum)
- `payload` (json)

Event types you likely need:
- `user_message`
- **streaming**: `agent_message_delta`
- **final**: `agent_message`
- `reasoning_delta`
- `turn_started`, `turn_completed`, `turn_aborted`
- `plan_update`
- `exec_begin`, `exec_output_delta`, `exec_end`

You can also materialize a `thread_items` table if you want the server to do
aggregation, but it is optional.

## Client Subscription Strategy (React)
- Use a `useQuery` subscription to fetch all events for a thread, ordered by
  `sequence`.
- Reduce events into view models on the client:
  - Build a transcript list
  - Track `activeExec` and `activePlan`
  - Track status header from reasoning deltas

Pseudo-reducer outline:

```
for event in events:
  switch event.type:
    case turn_started: taskRunning = true
    case turn_completed: taskRunning = false
    case reasoning_delta: statusHeader = extractBoldHeader(event.payload)
    case plan_update: update plan steps
    case exec_begin/exec_end: update active exec cell
    case agent_message_delta: append to streaming buffer
```

## Streaming Text (Agent speaking)
Codex uses newline-gated streaming (only commits complete lines). You can mimic
this in React by:

- Maintaining a `streamBuffer` string for delta text
- Only render committed lines when you see a newline
- On final event, flush remaining buffer

This reduces text reflow and makes the UI feel stable while streaming.

## Status Bar (Shimmer + elapsed + interrupt)
Implement a React status bar that is driven by:

- `taskRunning` boolean
- `statusHeader` string (from reasoning bold header)
- `elapsed` timer (starts on `turn_started`, stops on completion)
- An interrupt button (or ESC handler) that sends a Convex mutation to abort
  the running turn

## Task List (Plan Updates)
Codex plan updates are separate events that update a list of steps with
`pending/in_progress/completed` statuses.

In Convex:
- Store the plan as a `plan_update` event with `{ steps: [...] }`
- Or materialize `plan_steps` table keyed by `turnId`

In React:
- Render a checkbox list with styles based on status

## Tool Call Summaries (Exploring/Explored)
Codex uses parsed command metadata to decide if a call is "exploring":

- All actions are `Read`, `ListFiles`, or `Search`
- Not a user shell command

In your system:
- Parse tool calls server-side if possible
- Store `command_actions` on `exec_begin` events
- In React, if all actions are `read/list/search`, show the "Explored" grouping

## Interrupts and Queueing
Codex queues user inputs while a turn is running and replays them after
completion.

In React:
- Keep a local queue of messages attempted during `taskRunning`
- When `taskRunning` becomes false, send the next queued message

## Minimal Implementation Checklist
1) Convex `events` table + subscriptions
2) Reducer to construct transcript + status + plan from events
3) Streaming text component (newline-gated)
4) Status bar with elapsed timer and interrupt action
5) Tool call summary view using `command_actions`
6) Task list UI from plan updates

## React/Convex Implementation Recipe (Reducer + Components)
This section provides a concrete reducer spec and component props list so you
can wire up a working UI quickly.

### 1) Reducer spec (event -> UI state)
Create a reducer that folds the event stream into a single UI state object.

**State shape (suggested):**
```ts
type UIState = {
  taskRunning: boolean;
  statusHeader: string;
  statusDetails?: string;
  elapsedMs: number;
  plan: { step: string; status: "pending" | "in_progress" | "completed" }[];
  activeExec?: {
    callId: string;
    command: string[];
    commandActions?: Array<{
      type: "read" | "list_files" | "search" | "unknown";
      command: string;
      path?: string;
      query?: string;
      name?: string;
    }>;
    output?: string;
    exitCode?: number;
    done?: boolean;
  };
  transcript: Array<
    | { kind: "user"; text: string }
    | { kind: "agent"; lines: string[] }
    | { kind: "plan"; steps: UIState["plan"] }
    | { kind: "exec"; summary: string[]; output?: string }
  >;
  streamBuffer: string;
  queuedInputs: string[];
};
```

**Reducer rules (pseudocode):**
```ts
switch (event.type) {
  case "turn_started":
    taskRunning = true;
    statusHeader = statusHeader || "Working";
    start timer
    break;
  case "turn_completed":
  case "turn_aborted":
    taskRunning = false;
    stop timer
    flush streamBuffer into transcript
    drain queuedInputs (send next message)
    break;
  case "reasoning_delta":
    statusHeader = extractBoldHeader(event.payload.delta) ?? statusHeader;
    break;
  case "plan_update":
    plan = event.payload.steps;
    transcript.push({ kind: "plan", steps: plan });
    break;
  case "exec_begin":
    activeExec = { callId, command, commandActions };
    break;
  case "exec_output_delta":
    append to activeExec.output;
    break;
  case "exec_end":
    finalize activeExec into transcript;
    activeExec = undefined;
    break;
  case "agent_message_delta":
    streamBuffer += delta;
    if (delta includes "\\n") commit completed lines to transcript;
    break;
  case "agent_message":
    flush streamBuffer + message into transcript;
    break;
  case "user_message":
    transcript.push({ kind: "user", text });
    break;
}
```

### 2) Component map (what to build)
Build small focused components; the reducer supplies props.

**`ConversationView`**
- Props: `transcript`, `activeExec`, `streamBuffer`
- Renders transcript cells in order
- Appends a streaming agent bubble for partial output

**`PlanList`**
- Props: `steps`
- Renders checkbox list with styles for status

**`ExecSummary`**
- Props: `commandActions`, `output`, `done`
- If all actions are read/list/search -> show “Exploring/Explored” grouped rows
- Else show command header + truncated output

**`StatusBar`**
- Props: `taskRunning`, `statusHeader`, `elapsedMs`, `onInterrupt`
- Applies shimmer when `taskRunning` is true

### 3) Streaming rules (newline-gated)
Match Codex behavior by committing only full lines. This avoids jitter:

```ts
function commitStreamBuffer(buffer: string): { committed: string[]; rest: string } {
  const lastNewline = buffer.lastIndexOf("\\n");
  if (lastNewline === -1) return { committed: [], rest: buffer };
  const committed = buffer.slice(0, lastNewline + 1).split("\\n").filter(Boolean);
  const rest = buffer.slice(lastNewline + 1);
  return { committed, rest };
}
```

### 4) Queued user input
If `taskRunning` is true, do not send the message immediately:

```ts
if (taskRunning) queueInput(text);
else sendInput(text);
```

On completion, dequeue and send one message to start the next turn.
## Notes
If you want the backend to do more work, you can add a `thread_items` table
that materializes items (like Codex v2 `ThreadItem`). But starting from raw
`events` keeps the system flexible and simple.
