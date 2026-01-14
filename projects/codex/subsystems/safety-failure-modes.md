# Safety and Failure Modes

This subsystem covers exec policy, sandboxing, and the failure modes the runtime
handles gracefully.

## Safety / sandbox / approvals

- Exec policy plus sandbox enforcement.
- Explicit approval workflows for risky commands.

Key files:

- `codex-rs/core/src/exec_policy.rs`
- `codex-rs/core/src/tools/sandboxing/`
- `codex-rs/execpolicy/` and `codex-rs/execpolicy-legacy/`
- `shell-tool-mcp/` (sandboxed shell MCP server)

## Common runtime failure cases

1) Context window exceeded
   - `codex-rs/core/src/compact.rs` trims history and retries; may emit errors if
     compaction cannot fit.

2) Tool failures
   - `ToolRouter` generates a failure output item instead of crashing the turn.
   - `codex-rs/core/src/tools/router.rs` (`failure_response`)

3) Invalid image payloads
   - `run_turn` sanitizes and warns if image input is invalid.
   - `codex-rs/core/src/codex.rs` (invalid image branch)

4) MCP startup or call failures
   - Managed in `codex-rs/core/src/mcp_connection_manager.rs`, errors surface as events.
   - Tool names sanitized and truncated to avoid API constraints.

5) Undo / ghost snapshot issues
   - If repo is not git or snapshot fails, undo is unavailable.
   - Warnings emitted if snapshots are slow (large untracked files).
