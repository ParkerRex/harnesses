# Cline Architecture Docs (Manual)

TL;DR: This directory is a manual, per-subsystem breakdown of the Cline VS Code
extension. It focuses on the controller/task loop, tool execution with approvals,
prompt assembly, context/compaction, and disk persistence. Paths below are
relative to the `src/coding-agents/cline/` submodule.

## Start here

- Architecture overview: `architecture-overview.md`
- Key patterns: `key-patterns.md`
- Core runtime (controller, task, messages): `subsystems/core-runtime.md`
- Tools + approvals: `subsystems/tools.md`
- Model selection + prompting: `subsystems/models-prompting.md`
- Context window + compaction: `subsystems/context-compaction.md`
- Persistence + history: `subsystems/persistence.md`
- Safety + failure modes: `subsystems/safety-failure-modes.md`
- Observability: `subsystems/observability.md`
- Reference map: `reference/directory-map.md`
- Repo file tree: `reference/file-tree.md`
- Key files: `reference/key-files.md`
- Mental model: `reference/mental-model.md`
- Project snapshot: `reference/project-snapshot.md`
- Comparison notes: `comparison.md`
- Changelogs: `CHANGELOG-upstream.md`, `CHANGELOG-docs.md`
