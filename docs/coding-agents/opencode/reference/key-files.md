# OpenCode Key Files

Paths are relative to the `src/coding-agents/opencode/` submodule.

## Start here file list

- `packages/opencode/src/index.ts` - entry + CLI commands
- `packages/opencode/src/server/server.ts` - HTTP API surface
- `packages/opencode/src/session/index.ts` - sessions (create, fork, children)
- `packages/opencode/src/session/message-v2.ts` - message parts model
- `packages/opencode/src/session/prompt.ts` - prompt orchestration loop
- `packages/opencode/src/session/processor.ts` - stream event handling
- `packages/opencode/src/session/llm.ts` - LLM stream execution
- `packages/opencode/src/session/system.ts` - system prompt selection
- `packages/opencode/src/cli/cmd/tui/worker.ts` - worker event stream + RPC bridge
- `packages/opencode/src/cli/cmd/tui/thread.ts` - UI thread + worker wiring
- `packages/opencode/src/cli/cmd/tui/context/sdk.tsx` - SDK event batching + emitter
- `packages/opencode/src/cli/cmd/tui/context/sync.tsx` - event-to-store sync
- `packages/opencode/src/cli/cmd/tui/routes/session/index.tsx` - part rendering (thinking, tools, todos)
- `packages/opencode/src/cli/cmd/tui/component/todo-item.tsx` - todo item glyphs + styles
- `packages/opencode/src/tool/tool.ts` - tool definition contract
- `packages/opencode/src/tool/registry.ts` - tool registry
- `packages/plugin/src/index.ts` - plugin hooks
- `packages/sdk/openapi.json` - OpenAPI schema for SDK
