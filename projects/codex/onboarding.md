# Codex Onboarding

This page is the quick orientation for new engineers. Paths are relative to the
`codex/` submodule.

## Start here file list (most important)

- `codex-rs/core/src/codex.rs` - main agent loop, session/turn orchestration
- `codex-rs/core/src/thread_manager.rs` - thread creation + session wiring
- `codex-rs/core/src/context_manager/history.rs` - transcript + token accounting
- `codex-rs/core/src/tools/router.rs` - tool call mapping
- `codex-rs/core/src/tools/spec.rs` - tool definitions + registration
- `codex-rs/core/src/models_manager/model_info.rs` - model prompts and behavior
- `codex-rs/core/src/project_doc.rs` - AGENTS.md discovery + user instructions
- `codex-rs/core/src/compact.rs` - summary-based compaction
- `codex-rs/core/src/rollout/recorder.rs` - persistent session logs
- `codex-rs/protocol/src/protocol.rs` - public ops/events contract

## How to explain Codex to a new engineer

Codex CLI is a Rust-first agent runtime. A thread owns a session that holds all
state (history, features, tool configs). Each user input starts a turn: Codex
builds a prompt from history plus instructions, sends it to a model, and either
gets a message or a tool call. Tool calls are routed through a registry and
executed in a sandbox; outputs are recorded back into history and fed into the
next model turn. The system keeps context bounded via truncation and compaction,
and it persists every session as JSONL so it can be resumed or audited.

## Suggested onboarding exercises

1) Tool path walk
   - Follow a tool call from `ResponseItem::FunctionCall` -> `ToolRouter`
     -> `ToolHandler` -> output.

2) Compaction walk
   - Read `compact.rs` and `compact_remote.rs` and map to the `run_turn` loop.

3) Session persistence walk
   - Trace `RolloutRecorder` and how a session is resumed.

4) Prompt assembly walk
   - Map `ModelInfo` -> `Prompt` -> `ModelClient` -> response items.
