# Context Window, Token Usage, and Compaction

This document explains how Codex tracks token usage, renders the "X% context left"
indicator, and compacts conversation history (manual and automatic). It also calls
out the exact event flow from core -> app-server -> TUI so you can reproduce the
behavior in another stack.

## Token usage data model (core)

- `TokenUsage` holds the raw counts for a single usage report.
  `src/coding-agents/codex/codex-rs/protocol/src/protocol.rs:976`
- `TokenUsageInfo` holds:
  - `total_token_usage`: cumulative total across the session
  - `last_token_usage`: the most recent usage snapshot
  - `model_context_window`: optional context size used for percent display
  `src/coding-agents/codex/codex-rs/protocol/src/protocol.rs:990`

Percent calculation logic:

- `TokenUsage::percent_of_context_window_remaining` computes the percent remaining
  using a fixed baseline of 12,000 tokens so the user-visible percent reflects the
  user-controllable portion of the prompt.
  `src/coding-agents/codex/codex-rs/protocol/src/protocol.rs:1117`
- `BASELINE_TOKENS` = 12,000 is subtracted from both the window size and the
  used tokens before calculating the percentage.
  `src/coding-agents/codex/codex-rs/protocol/src/protocol.rs:1110`

## Effective context window

Codex does not use the raw model context window directly. It applies an
"effective context window" percentage (default 95%) to reserve headroom:

- `ModelClient::get_model_context_window()` returns
  `model_info.context_window * effective_context_window_percent / 100`.
  `src/coding-agents/codex/codex-rs/core/src/client.rs:120`
- The default effective percent is 95 in model metadata.
  `src/coding-agents/codex/codex-rs/core/src/models_manager/model_info.rs:55`

So the "X% context left" UI is calculated against a *smaller* effective window,
not the full model capacity.

## TokenCount event flow (core -> app-server -> TUI)

Core emits a TokenCount event whenever token usage or rate limits change:

- `codex::send_token_count_event` wraps `TokenCountEvent { info, rate_limits }`.
  `src/coding-agents/codex/codex-rs/core/src/codex.rs:1487`
- Token info updates come from:
  - `update_token_usage_info` (after API responses)
  - `recompute_token_usage` (after compaction)
  - `set_total_tokens_full` (context window exceeded)
  `src/coding-agents/codex/codex-rs/core/src/codex.rs:1425`

In app-server, `TokenCountEvent` becomes a notification:

- `ThreadTokenUsageUpdatedNotification` carries `ThreadTokenUsage` with
  `total`, `last`, and `model_context_window`.
  `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/v2.rs:1345`
- The mapping is done in `bespoke_event_handling`.
  `src/coding-agents/codex/codex-rs/app-server/src/bespoke_event_handling.rs:1016`

## "X% context left" in the TUI

TUI rendering uses the most recent token info snapshot:

- `ChatWidget::apply_token_info` computes percent and passes it to the bottom pane.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:733`
- `context_remaining_percent` uses `info.last_token_usage`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:746`
- The footer line renders as:
  - `"{percent}% context left"` when a percent is available
  - `"{tokens} used"` if percent is not available
  - default `"100% context left"` if no data yet
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/footer.rs:259`

In `/status`, the UI shows a richer context breakdown:

- `StatusHistoryCell` builds `StatusContextWindowData` using
  `info.last_token_usage` (or config fallback) and renders
  `"X% left (used / window)"`.
  `src/coding-agents/codex/codex-rs/tui/src/status/card.rs:69`

## Context window exceeded handling

If the model reports context window exceeded:

- `run_model_turn` calls `set_total_tokens_full`, which fills the token usage
  to the context window and emits a TokenCount event.
  `src/coding-agents/codex/codex-rs/core/src/codex.rs:2719`

This ensures the UI shows 0% left (or a full window) even when the request
fails, so the user can see why the turn failed.

## Auto compaction (when it triggers)

Auto compaction runs inside `run_turn` when usage crosses a limit:

- The limit is `model_info.auto_compact_token_limit`, which can be overridden
  by config `model_auto_compact_token_limit`.
  `src/coding-agents/codex/codex-rs/core/src/models_manager/model_info.rs:73`
- The check happens before the turn starts and again if a follow-up is needed
  and the limit is exceeded.
  `src/coding-agents/codex/codex-rs/core/src/codex.rs:2515`

The "total usage" used for the check is *not* the sum of all historical tokens.
`ContextManager::get_total_token_usage` uses `last_token_usage.total_tokens` plus
an estimate for reasoning bytes prior to the last user message.
`src/coding-agents/codex/codex-rs/core/src/context_manager/history.rs:233`

## Manual compaction (/compact)

- The TUI slash command `/compact` clears token usage locally, then sends
  `Op::Compact`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1915`
- The core runs a `CompactTask` (TaskKind::Compact).
  `src/coding-agents/codex/codex-rs/core/src/tasks/compact.rs:12`

## Local compaction algorithm

Local compaction generates a summary and rebuilds history:

- Prompt templates live in:
  - `core/src/templates/compact/prompt.md`
  - `core/src/templates/compact/summary_prefix.md`
  `src/coding-agents/codex/codex-rs/core/src/compact.rs:25`
- `collect_user_messages` filters user messages and removes prior summaries
  (prefixed by `SUMMARY_PREFIX`) plus injected instructions.
  `src/coding-agents/codex/codex-rs/core/src/compact.rs:206`
- `build_compacted_history` keeps the most recent user messages up to
  `COMPACT_USER_MESSAGE_MAX_TOKENS` (20,000), truncating the oldest oversized
  message if needed, then appends the summary as a **user** message.
  `src/coding-agents/codex/codex-rs/core/src/compact.rs:231`
- Ghost snapshots are preserved so `/undo` can still work after compaction.
  `src/coding-agents/codex/codex-rs/core/src/compact.rs:162`

After rebuilding history, Codex recomputes token usage:

- `sess.recompute_token_usage` calls `ContextManager::estimate_token_count` and
  emits a new TokenCount event.
  `src/coding-agents/codex/codex-rs/core/src/codex.rs:1453`

## Remote compaction

Remote compaction delegates summary generation to the API:

- `ModelClient::compact_conversation_history` calls the Compact endpoint and
  returns a new `Vec<ResponseItem>`.
  `src/coding-agents/codex/codex-rs/core/src/client.rs:183`
- The returned history is stored in `CompactedItem.replacement_history` so
  resume/fork can reconstruct without re-running the summarizer.
  `src/coding-agents/codex/codex-rs/core/src/compact_remote.rs:69`

Remote compaction is only used when:

- The provider is OpenAI **and** `Feature::RemoteCompaction` is enabled.
  `src/coding-agents/codex/codex-rs/core/src/compact.rs:35`

## Compaction events and UI

Events emitted after compaction:

- `EventMsg::ContextCompacted` is always emitted.
  `src/coding-agents/codex/codex-rs/core/src/compact.rs:183`
- Local compaction also emits a Warning telling the user to keep threads small.
  `src/coding-agents/codex/codex-rs/core/src/compact.rs:186`

App-server forwards this as:

- `ContextCompactedNotification` (`thread/compacted`).
  `src/coding-agents/codex/codex-rs/app-server-protocol/src/protocol/common.rs:568`

The TUI responds by inserting a plain agent message:

- `EventMsg::ContextCompacted` -> `on_agent_message("Context compacted")`.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:2364`

## Persistence and resume

Compaction is persisted in the rollout log:

- `RolloutItem::Compacted` stores either the summary text (local) or the
  replacement history (remote).
  `src/coding-agents/codex/codex-rs/protocol/src/protocol.rs:1419`

When resuming, Codex reconstructs history using these entries.
`src/coding-agents/codex/codex-rs/core/src/codex.rs:1307`

## Porting notes (Convex + Next.js)

Key behaviors to replicate:

- Keep a single source of truth for `TokenUsageInfo` on the server and emit a
  token-usage event after any model response, compaction, or error that affects
  context.
- Compute "context left" from a stable *effective* window and subtract a fixed
  baseline before showing percentages.
- Treat compaction as a distinct task/turn so the UI can show "working" state
  and emit an explicit "context compacted" message.
- Persist compaction summaries in history so resume/fork works without
  recomputing the summary.
