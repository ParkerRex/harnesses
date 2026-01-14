# harness-howto

Submodules + architecture breakdowns for the major open-source coding agents. Auto-updated every 3 days.

**Why follow this repo:** One place to track how agentic systems handle sessions, tools, context windows, and orchestration across different implementations.

## How it works

- Submodules point to upstream repos (codex, eigent, gemini-cli, opencode, openhands)
- GitHub Action runs every 3 days to pull latest changes
- If a submodule has 5+ new commits, Claude Code regenerates its architecture doc using [.prompts/doc-gen.md](.prompts/doc-gen.md)
- Docs commit directly to main

Manual trigger: Actions → "Update submodules & docs" → Run workflow

## Projects

Stars are from GitHub as of 2026-01-14.

| Project | Writeups | Stars |
| --- | --- | --- |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | [gemini-cli-architecture.md](gemini-cli-architecture.md) | 90.9k |
| [Eigent](https://github.com/eigent-ai/eigent) | [eigent-architecture.md](eigent-architecture.md) | 4.4k |
| [OpenCode](https://github.com/anomalyco/opencode) | [opencode-architecture.md](opencode-architecture.md) | 68.6k |
| [Codex](https://github.com/openai/codex) | [codex-dotfile-setup.md](codex-dotfile-setup.md)<br>[codex-architecture-breakdown.md](codex-architecture-breakdown.md) | 56.1k |
| [OpenHands](https://github.com/OpenHands/OpenHands) | [openhands-architecture.md](openhands-architecture.md) | 66.6k |

## Docs in this repo

- `claude-dotfile-setup.md`
- `codex-dotfile-setup.md`
- `codex-architecture-breakdown.md`
- `eigent-architecture.md`
- `gemini-cli-architecture.md`
- `openhands-architecture.md`
- `opencode-architecture.md`
