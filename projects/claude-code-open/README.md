# Claude Code Open Architecture Docs (Manual)

TL;DR: This directory is a manual, per-subsystem breakdown of Claude Code Open
(the `claude-code-open` submodule). It covers the CLI/TUI loop, tool execution
with permissions and sandboxing, prompt assembly, compaction, and on-disk
session storage. Paths below are relative to the `claude-code-open/` submodule.

## Start here

- Repo map: `repo-map.md`
- Core runtime (loop + sessions): `subsystems/core-runtime.md`
- Tools + permissions + sandbox: `subsystems/tools.md`
- Model selection + prompting: `subsystems/models-prompting.md`
- Context window + compaction: `subsystems/context-compaction.md`
- Persistence + memory: `subsystems/persistence-memory.md`
- Safety + failure modes: `subsystems/safety-failure-modes.md`
- Observability: `subsystems/observability.md`
- Onboarding checklist: `onboarding.md`

## System overview

```mermaid
flowchart LR
  subgraph Client
    CLI[CLI Entry (Commander)]
    TUI[Ink UI]
    Text[Text/Print Mode]
  end

  subgraph Core
    Loop[ConversationLoop]
    Session[Session State + Messages]
    Prompt[SystemPromptBuilder]
    Tools[ToolRegistry]
    Permissions[Permission Manager]
    Hooks[Hook Runner]
  end

  subgraph Storage
    Sessions[(~/.claude/sessions/*.json)]
    Summaries[(~/.claude/sessions/summaries)]
    Memory[(~/.claude/memory + ./.claude/memory)]
    Telemetry[(~/.claude/telemetry)]
    ToolOutputs[(~/.claude/tasks)]
  end

  subgraph External
    Anthropic[Anthropic Messages API]
    OAuth[OAuth Endpoints]
    MCP[MCP Servers]
    Web[WebFetch/WebSearch]
  end

  CLI --> Loop
  TUI --> Loop
  Text --> Loop
  Loop --> Prompt
  Loop --> Session
  Loop --> Tools
  Tools --> Permissions
  Tools --> Hooks
  Tools --> MCP
  Loop --> Anthropic
  Loop --> OAuth
  Session --> Sessions
  Session --> Summaries
  Session --> Memory
  Loop --> Telemetry
  Tools --> ToolOutputs
  Tools --> Web
```
