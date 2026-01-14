# Persistence and Memory

Sessions, summaries, memory, and telemetry are stored on disk under `~/.claude`.

## Session storage

- Session state and JSON persistence: `src/core/session.ts`.
- Session list/load/fork/merge: `src/session/index.ts`.
- Resume logic: `src/session/resume.ts`.
- Cleanup policies: `src/session/cleanup.ts`.

## Memory

- Global and project memory: `src/memory/index.ts`.

## Telemetry and tool outputs

- Telemetry events: `src/telemetry/index.ts`.
- Tool output persistence: `src/tools/output-persistence.ts`.
