# OpenCode Architecture Docs (Manual)

TL;DR: This directory is a manual, per-subsystem breakdown of the OpenCode
codebase (the `src/coding-agents/opencode` submodule). It focuses on the client/server runtime,
sessions and message parts, tool routing, model prompting, context compaction,
and sharing/sync. Paths below are relative to the `src/coding-agents/opencode/` submodule.

## Start here

- Architecture overview: `architecture-overview.md`
- Key patterns: `key-patterns.md`
- Core runtime (sessions, messages, processor): `subsystems/core-runtime.md`
- Tools and tool state: `subsystems/tools.md`
- Model selection + prompting: `subsystems/models-prompting.md`
- Context window + compaction: `subsystems/context-compaction.md`
- TUI streaming + thinking/todos UI: `subsystems/tui-streaming-ui.md`
- Persistence + sharing: `subsystems/persistence-sharing.md`
- Reference map: `reference/directory-map.md`
- Repo file tree: `reference/file-tree.md`
- Key files: `reference/key-files.md`
- Mental model: `reference/mental-model.md`
- Project snapshot: `reference/project-snapshot.md`
- Comparison notes: `comparison.md`
- Changelogs: `CHANGELOG-upstream.md`, `CHANGELOG-docs.md`
