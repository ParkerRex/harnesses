# OSS Codebase Architecture Documentation Generator

## Purpose
Generate comprehensive onboarding documentation for agentic codebases and AI-powered developer tools. This prompt produces detailed, diagram-rich documentation suitable for team onboarding.

## Target Output
A markdown file (`{project-name}-architecture.md`) in the repo root with:
- Mermaid flowcharts, sequence diagrams, state charts, and swimlanes
- Plain English explanations alongside technical depth
- Specific file/code callouts for easy navigation
- Focus on agentic patterns: sessions, tools, context, orchestration
- Include infrastructure and product surfaces when relevant: sandboxing, prebuilds, clients, auth/PR flow, observability

---

## Prompt

Analyze the `{SUBMODULE_PATH}` directory and generate a comprehensive architecture onboarding document.

### Focus Areas (prioritized)
1. **Sessions & Threads** - How conversations/threads are modeled, stored, and managed
2. **Tool Call Mapping** - How tools are defined, registered, invoked, and tracked through execution
3. **Model Selection & System Prompting** - Provider abstraction, model routing, prompt composition
4. **Building Tools** - The canonical path to add a new tool (contract, registration, hooks)
5. **Context Window Management** - Token tracking, pruning strategies, overflow handling
6. **Compaction/Summarization** - How context is compressed when limits are hit
7. **Message Lifecycle** - From user input through LLM response to persistence
8. **Failure Modes** - Error handling, retries, graceful degradation patterns
9. **Memory/State** - How state persists across turns, sessions, restarts
10. **Multi-Agent Orchestration** - If applicable: coordinators, workers, task decomposition
11. **Execution Environment** - Sandboxes, isolation, base images, snapshots, warm pools, TTFT goals
12. **Session Lifecycle** - Create → warm → sync → run → verify → finalize → snapshot → resume/expire
13. **Clients & Surfaces** - Slack/web/IDE/extension, request routing, status UI
14. **Collaboration / Multiplayer** - Multi-user sessions, attribution, concurrency rules
15. **AuthN/AuthZ + Git/PR Flow** - User tokens, git identity, PR creation, webhooks
16. **Observability & Metrics** - TTFT, latency, success/merge rate, dashboards
17. **Adoption/Workflow Fit** - How usage is encouraged (virality loops, public spaces, UX nudges)

### Required Sections

```markdown
# {Project Name} Architecture - Onboarding Doc

## TL;DR
<!-- 2-3 sentence summary: what is this, who is it for, core value prop -->

## System Map
<!-- High-level component diagram showing major pieces and their relationships -->
<!-- Include: clients, core runtime, storage, external services -->

## Execution Environment
<!-- Sandbox/VM/container model, isolation strategy, base image/prebuilds -->
<!-- Startup latency/TTFT goals, snapshot/restore, read-before-sync/write-gates -->

## Core Data Model
<!-- Sessions, messages, parts/chunks - how conversations are structured -->
<!-- Include entity relationship or class diagram -->

## Message Lifecycle
<!-- Step-by-step: user input → processing → LLM → response → storage -->

## Sequence: User → Tool → Response
<!-- Mermaid sequence diagram showing the happy path -->

## Tool System

### Tool Call Mapping
<!-- How tool invocations are tracked: pending → running → completed/error -->
<!-- State diagram for tool lifecycle -->

### How to Build a New Tool
<!-- Step-by-step canonical path with code snippets -->

## Model Selection & System Prompting
<!-- How models are chosen, how system prompts are composed -->
<!-- Include: provider abstraction, config hierarchy, prompt assembly -->

## Context Window Management
<!-- Token tracking, soft pruning, hard limits -->

## Compaction
<!-- When/how context is compressed -->
<!-- Flow diagram for compaction trigger → summary → continuation -->

## Sessions & Threading
<!-- How threads/sessions relate, forking, parent-child relationships -->

## Session Lifecycle
<!-- Create → warm → sync → run → verify → finalize → snapshot → resume/expire -->
<!-- Prompt handling (queue vs interrupt), stop/cancel, child sessions/status polling -->

## Collaboration / Multiplayer
<!-- Shared session model, attribution, conflict policy, edit ordering -->

## Clients & Surfaces
<!-- Slack/web/IDE/extension, routing/classification, status signaling -->

## AuthN/AuthZ + Git/PR Flow
<!-- Auth model, token use, git identity, PR creation, webhook sync, review guardrails -->

## Failure Modes & Error Handling
<!-- Common failure patterns and how they're handled -->

## Swimlane: End-to-End Request
<!-- Full swimlane diagram showing all actors: User, Client, Server, LLM, Tools, Storage -->

## Observability & Metrics
<!-- TTFT, end-to-end latency, success rate, PR merge rate, dashboards -->

## Cost & Performance Tradeoffs
<!-- Prebuild vs on-demand, warm pools, caching, scaling bottlenecks -->

## Key Files to Read First
<!-- Prioritized list with brief descriptions -->
<!-- Format: `path/to/file.ts` — what it does, why it matters -->

## Quick "What Lives Where"
<!-- Directory → responsibility mapping -->

## Mental Model Cheat-Sheet
<!-- Plain English summary of core concepts for quick reference -->
```

### Diagram Requirements

Include these diagram types where applicable:

1. **System/Component Flowchart** - `flowchart LR` showing major components
2. **Sequence Diagram** - `sequenceDiagram` for message flow
3. **State Diagram** - `stateDiagram-v2` for tool call states
4. **Swimlane** - `flowchart LR` with subgraphs for each actor
5. **Entity Relationship** - For data model relationships
6. **Lifecycle Diagram** - Session lifecycle warm/sync/verify/snapshot (if applicable)

### Style Guidelines

- **Plain English first** - Explain concepts simply before diving into code
- **Specific file callouts** - Always include `path/to/file.ts` references
- **Code snippets** - Show minimal examples of key patterns (tool definition, message creation)
- **No assumptions** - Don't assume reader knows the framework or libraries used
- **Consistent terminology** - Define terms once, use consistently throughout
- **Bullet points for lists** - Use bullets for file lists, steps, feature lists
- **Tables for mappings** - Use tables for config options, event types, etc.

### Information to Extract

For each focus area, find and document:

| Area | What to Find | Where to Look |
|------|--------------|---------------|
| Sessions | Session model, storage, lifecycle | `**/session/**`, `**/store/**`, `**/thread/**` |
| Tools | Tool interface, registry, execution | `**/tool/**`, `**/toolkit/**`, `**/function/**` |
| Models | Provider abstraction, model config | `**/provider/**`, `**/model/**`, `**/llm/**` |
| Context | Token counting, truncation, compaction | `**/context/**`, `**/compaction/**`, `**/token/**` |
| Messages | Message types, parts, lifecycle | `**/message/**`, `**/chat/**`, `**/conversation/**` |
| Orchestration | Agents, workers, coordination | `**/agent/**`, `**/workforce/**`, `**/coordinator/**` |
| Config | Settings, environment, runtime options | `**/config/**`, `**/settings/**`, `**/.env*` |
| API | Endpoints, SSE/WebSocket, client SDK | `**/server/**`, `**/api/**`, `**/controller/**` |
| Sandbox | Runtime env, images, snapshots, warm pools | `**/sandbox/**`, `**/runtime/**`, `**/infra/**` |
| Clients | Slack/web/IDE/extension surfaces | `**/client/**`, `**/ui/**`, `**/extension/**` |
| Auth/Git | Auth flow, git/PR integrations | `**/auth/**`, `**/git/**`, `**/vcs/**` |
| Observability | Logs, traces, metrics, dashboards | `**/log/**`, `**/trace/**`, `**/metrics/**` |

### Output Format

Write the documentation to: `{project-name}-architecture.md`

Length: Comprehensive (1000-2000 lines typical). Prioritize completeness over brevity for onboarding docs.

---

## Variables

Replace before running:

- `{SUBMODULE_PATH}` - Path to the submodule directory (e.g., `./codex`, `./eigent`)
- `{project-name}` - Name for output file (e.g., `codex`, `eigent`)
- `{Project Name}` - Display name for headings (e.g., `Codex`, `Eigent`)

---

## Example Invocation

```
Analyze the ./codex directory and generate a comprehensive architecture
onboarding document following the template above. Focus especially on:
- How the Rust agent handles tool calls
- Session/conversation state management
- The model provider abstraction
- Context window and compaction strategies

Write the output to codex-architecture.md
```

---

## For Incremental Updates

When updating existing documentation:

1. Read the existing `{project-name}-architecture.md`
2. Check git log for recent changes in the submodule: `git -C {SUBMODULE_PATH} log --oneline -20`
3. Focus on sections affected by recent changes
4. Preserve existing accurate content
5. Update diagrams if data flow changed
6. Add new sections for new features
7. Mark deprecated patterns if removed

Update trigger: Run weekly or when submodule is updated.
