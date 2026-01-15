# Eigent Architecture Docs (Manual)

TL;DR: This directory is a manual, per-subsystem breakdown of Eigent (the
`src/coding-agents/eigent` submodule). Eigent is a desktop Electron app that spawns a local FastAPI
runtime and streams SSE events to the UI, with optional server persistence and
sharing. Paths below are relative to the `src/coding-agents/eigent/` submodule.

## Start here

- Architecture overview: `architecture-overview.md`
- Key patterns: `key-patterns.md`
- Core runtime (Electron + backend + SSE): `subsystems/core-runtime.md`
- Tools and toolkits: `subsystems/tools.md`
- Model selection + prompting: `subsystems/models-prompting.md`
- Context window + compaction: `subsystems/context-compaction.md`
- Persistence + sharing: `subsystems/persistence-sharing.md`
- Safety + failure modes: `subsystems/safety-failure-modes.md`
- Reference map: `reference/directory-map.md`
- Repo file tree: `reference/file-tree.md`
- Key files: `reference/key-files.md`
- Mental model: `reference/mental-model.md`
- Project snapshot: `reference/project-snapshot.md`
- Comparison notes: `comparison.md`
- Changelogs: `CHANGELOG-upstream.md`, `CHANGELOG-docs.md`
