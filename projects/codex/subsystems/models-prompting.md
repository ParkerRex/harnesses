# Model Selection and Prompting

This subsystem selects the model, builds the prompt, and controls how tools and
instructions are injected into each request.

## Model selection

- `codex-rs/core/src/models_manager/manager.rs` selects a model.
  - Defaults depend on auth mode (ChatGPT vs API key).
  - Caches remote model metadata.
- `codex-rs/core/src/models_manager/model_info.rs` defines per-model behavior:
  - Base instructions for each model (`prompt.md`, `gpt_5_2_prompt.md`, etc.).
  - Tool support (apply_patch tool type, shell type, parallel tools).
  - Truncation policy and auto-compaction thresholds.

## Prompt construction (inputs + instructions)

Prompt inputs are built from:

1) History (`ContextManager`) - user/assistant messages, tool calls/outputs
2) Base instructions - model-specific prompt text
3) Developer instructions - config overrides
4) User instructions - AGENTS.md + config + skills
5) Tools - serialized tool specs for the current turn

Important files:

- `codex-rs/core/src/client_common.rs` - `Prompt` structure; tool serialization
- `codex-rs/core/src/models_manager/model_info.rs` - base prompt text + model config
- `codex-rs/core/prompt.md` and `codex-rs/core/gpt_*.md` - instruction templates
- `codex-rs/core/src/project_doc.rs` - loads AGENTS.md from repo root -> cwd
- `codex-rs/core/src/user_instructions.rs` - wraps AGENTS/skills into message items
- `codex-rs/core/src/custom_prompts.rs` - discovers prompts in `$CODEX_HOME/prompts`
- `codex-rs/core/src/skills/` - skill discovery and injection

## Prompt assembly diagram

```mermaid
flowchart TB
  A[ModelInfo: base_instructions] --> P[Prompt]
  B[Developer instructions (config)] --> P
  C[User instructions: AGENTS.md + config] --> P
  D[Skills injections] --> P
  E[ContextManager history] --> P
  F[Tool specs for this turn] --> P
  P --> M[Model request]
```

Key idea: base + developer + user + skills are additive; history is normalized
and truncated as needed before being sent.
