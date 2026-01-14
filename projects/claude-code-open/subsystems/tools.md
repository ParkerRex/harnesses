# Tools, Permissions, and Sandbox

Tools are executed locally with permission checks and optional sandboxing.

## Tool registry and execution

- Tool base classes and schemas: `src/tools/base.ts`.
- Tool registration: `src/tools/index.ts`.
- Tool interfaces: `src/types/tools.ts`.
- Tool execution and result injection: `src/core/loop.ts`.

## Permissions

- Permission manager and audit rules: `src/permissions/index.ts`.
- Tool permission checks: `BaseTool.checkPermissions()`.

## Sandboxing

- Sandbox entry: `src/tools/sandbox.ts`.
- Linux bubblewrap: `src/sandbox/bubblewrap.ts`.
- macOS seatbelt: `src/sandbox/seatbelt.ts`.
- Filesystem and network policies: `src/sandbox/filesystem.ts`,
  `src/sandbox/network.ts`.
