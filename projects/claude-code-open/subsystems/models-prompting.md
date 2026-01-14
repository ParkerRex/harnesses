# Model Selection and Prompting

This subsystem configures model routing and assembles the system prompt.

## Model selection

- CLI flags: `src/cli.ts`.
- Model registry and aliases: `src/models/config.ts`.
- Fallback behavior: `src/models/fallback.ts`.
- Quota tracking: `src/models/quota.ts`.
- Extended thinking options: `src/models/thinking.ts`.

## System prompt assembly

- Prompt builder: `src/prompt/builder.ts`.
- Prompt templates: `src/prompt/templates.ts`.
- OAuth identity requirements: `src/core/client.ts`.
