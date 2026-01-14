# Memory and Persistence

Codex has two kinds of memory: in-memory transcript state and durable rollouts
written to disk.

## In-memory transcript (fast, volatile)

- `ContextManager` stores the live `ResponseItem` history.
- Used to build prompts every turn.
- Editable via undo / rollback operations.

Key files:

- `codex-rs/core/src/context_manager/history.rs`
- `codex-rs/core/src/state/session.rs`

## Persistent rollouts (durable)

- Each session is persisted as JSONL under `~/.codex/sessions/`.
- Enables resume, history inspection, and UI replay.

Key files:

- `codex-rs/core/src/rollout/recorder.rs`
- `codex-rs/core/src/rollout/list.rs`
- `codex-rs/protocol/src/protocol.rs` (`InitialHistory`, `ResumedHistory`)

## Ghost snapshots (undo safety)

Codex snapshots the repo (ghost commits) so it can undo tool-driven file edits.

- `codex-rs/core/src/tasks/ghost_snapshot.rs`
- `codex-rs/core/src/tasks/undo.rs`
