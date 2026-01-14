# Codex Repo Map (Top-Level)

At the top level of the `codex` submodule:

- `codex-rs/` - Rust workspace containing the core agent, TUI, exec CLI, protocol,
  tool runtimes, and supporting crates.
- `codex-cli/` - JS/TS packaging wrapper for CLI distribution (Rust binary is the core).
- `sdk/` - SDKs (for example `sdk/typescript/`) for programmatic use.
- `shell-tool-mcp/` - MCP server providing a sandboxed `shell` tool.
- `docs/` - usage, configuration, and TUI docs.

Key emphasis: `codex-rs/core/` is the business logic (the agent brain). All UIs
(TUI, exec, SDKs) delegate into core.
