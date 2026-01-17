# Slash Commands (TUI)

This document explains how slash commands work in Codex TUI: discovery,
parsing, dispatch, and how commands interact with tasks. It is intended as a
blueprint for recreating the same UX in a React client.

## Command inventory

Built-in commands are defined in a single enum, and the enum order is the
presentation order in the popup:

- `SlashCommand` enum and descriptions live in:
  `src/coding-agents/codex/codex-rs/tui/src/slash_command.rs:8`
- `SlashCommand::description()` provides the tooltip text.
  `src/coding-agents/codex/codex-rs/tui/src/slash_command.rs:27`
- `SlashCommand::available_during_task()` gates commands while a task runs.
  `src/coding-agents/codex/codex-rs/tui/src/slash_command.rs:63`
- Debug-only commands (`/rollout`, `/test-approval`) are hidden unless
  `debug_assertions` is enabled.
  `src/coding-agents/codex/codex-rs/tui/src/slash_command.rs:97`

## Parsing and recognition

Only the **first line** is considered for slash commands.

- `parse_slash_name` extracts the command name and remaining text.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/prompt_args.rs:62`

Rules that determine whether text is treated as a command:

- Leading space disables command handling (treat as literal text).
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs:1310`
- Names containing `/` are treated as literal text to avoid path collisions.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs:1316`
- If the command is unknown and not a custom prompt, the UI inserts an info
  message and restores the original text.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs:1328`

## Command popup behavior

The command popup activates when the caret is inside the `/name` token and
that token looks like a prefix of a known command or custom prompt:

- Prefix detection uses fuzzy matching against built-ins and custom prompts.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs:1909`
- The popup is suppressed when the caret is inside an `@file` token (file
  search takes priority).
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs:1951`

The popup list itself is built in `CommandPopup`:

- Built-ins are loaded in enum order and can be filtered out when disabled
  (skills off, elevated sandbox not allowed).
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/command_popup.rs:35`
- Custom prompts are appended after built-ins and excluded if they collide
  with built-in names.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/command_popup.rs:49`
- Filtering uses fuzzy match and sorts by match score, then name.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/command_popup.rs:98`

## Submission and dispatch rules

The composer distinguishes between:

- Bare commands (no args): `try_dispatch_bare_slash_command()`.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs:1439`
- Commands with args (only `/review` currently):
  `try_dispatch_slash_command_with_args()`.
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs:1457`

If a command is dispatched, the input is cleared and `InputResult::Command` or
`InputResult::CommandWithArgs` is returned to the ChatWidget.
`src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1781`

## Command execution

ChatWidget enforces task gating before executing a command:

- If `available_during_task()` is false and a task is running, the UI emits an
  error cell and skips the command.
  `src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1872`

Command actions include:

- `/new`, `/resume`, `/fork`: send session lifecycle events.
- `/init`: inserts a canned prompt into the chat input.
- `/compact`: clears token usage and sends `Op::Compact`.
- `/review`: opens review flow; `/review <args>` passes args to review.
- `/model`, `/approvals`, `/experimental`, `/skills`: open popups.
- `/diff`, `/mention`, `/status`, `/ps`, `/mcp`: trigger UI flows or data fetches.
- `/feedback`: opens feedback UI or a disabled view.
- `/quit`/`/exit`: request app exit.

Entry point:
`src/coding-agents/codex/codex-rs/tui/src/chatwidget.rs:1864`

## Custom prompts (`/prompts:`)

Custom prompts are treated like commands and expanded before submission:

- Prefix constant: `PROMPTS_CMD_PREFIX` ("prompts").
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/prompt_args.rs:4`
- `expand_custom_prompt` supports:
  - Named placeholders: `$NAME` with `key=value` inputs
  - Positional placeholders: `$1..$9` and `$ARGUMENTS`
  - Shlex-style quoting for values with spaces
  `src/coding-agents/codex/codex-rs/tui/src/bottom_pane/prompt_args.rs:111`

Expansion errors are surfaced as error history cells and the original input is
restored.
`src/coding-agents/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs:1341`

## Porting notes (Convex + Next.js)

To reproduce the UX:

- Keep a centralized command registry (list + description + availability).
- Use fuzzy match on the first `/token` to decide whether to show a command
  picker. Prefer file-mention popups when the caret is inside `@path`.
- Treat leading space as an escape hatch (submit literal text).
- Split command dispatch from message submission (command-only messages should
  not be sent to the model).
- Support custom prompt expansion with quoted args and error recovery.
