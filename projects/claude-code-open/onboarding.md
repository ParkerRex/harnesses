# Claude Code Open Onboarding

This page is the quick orientation for new engineers. Paths are relative to the
`claude-code-open/` submodule.

## Start here file list (most important)

- `src/cli.ts` - CLI flags and startup flow
- `src/core/loop.ts` - orchestration loop + tool execution + compaction
- `src/core/session.ts` - session state + persistence
- `src/session/index.ts` - session list/load/fork/merge
- `src/tools/base.ts` - tool base class + registry hooks
- `src/tools/index.ts` - tool registration
- `src/prompt/builder.ts` - system prompt assembly
- `src/models/config.ts` - model registry and context windows
- `src/context/index.ts` - token estimation + compression
- `src/permissions/index.ts` - permission modes + audit logging
- `src/auth/index.ts` - OAuth + API keys
- `src/ui/App.tsx` - Ink-based TUI

## Mental model cheat sheet

- Loop = build system prompt -> call model -> run tools -> append tool results.
- Sessions are persisted JSON with usage stats and metadata.
- Permissions gate tools and can remember allow/deny decisions.
- Context pressure triggers auto-compaction.
- Storage is local by default: sessions, memory, telemetry, and tool outputs.
