# Safety and Failure Modes

Claude Code Open handles tool errors, permission denials, and model failures
inside the main loop.

Common failure cases:

- API errors and overload retries:
  `src/core/client.ts`, `src/models/fallback.ts`.
- Tool execution errors and timeouts:
  `src/tools/base.ts`.
- Permission denials short-circuit tool execution:
  `src/permissions/index.ts`.
- Compaction failures log warnings:
  `src/core/loop.ts`.
- Session load failures handled in:
  `src/core/session.ts`.
