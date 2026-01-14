# harness-howto

Submodules + architecture breakdowns for the major open-source coding agents. Auto-updated every 3 days.

**Why follow this repo:** One place to track how agentic systems handle sessions, tools, context windows, and orchestration across different implementations.

## How it works

- Submodules point to upstream repos (codex, eigent, gemini-cli, opencode, openhands, claude-code-open)
- GitHub Action runs every 3 days to pull latest changes
- If a submodule has 5+ new commits, Claude Code incrementally updates its architecture doc (patches based on changes) using [.prompts/doc-gen.md](.prompts/doc-gen.md)
- Docs commit directly to main

Manual trigger: Actions → "Update submodules & docs" → Run workflow

## Projects

Stars are from GitHub as of 2026-01-14.

| Project | Writeups | Stars |
| --- | --- | --- |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | [gemini-cli-architecture.md](gemini-cli-architecture.md)<br>[projects/gemini-cli/README.md](projects/gemini-cli/README.md) | 90.9k |
| [Eigent](https://github.com/eigent-ai/eigent) | [eigent-architecture.md](eigent-architecture.md)<br>[projects/eigent/README.md](projects/eigent/README.md) | 4.4k |
| [OpenCode](https://github.com/anomalyco/opencode) | [opencode-architecture.md](opencode-architecture.md)<br>[projects/opencode/README.md](projects/opencode/README.md) | 68.6k |
| [Codex](https://github.com/openai/codex) | [codex-dotfile-setup.md](codex-dotfile-setup.md)<br>[codex-architecture-breakdown.md](codex-architecture-breakdown.md)<br>[projects/codex/README.md](projects/codex/README.md) | 56.1k |
| [Cline](https://github.com/cline/cline) | [cline-architecture.md](cline-architecture.md)<br>[projects/cline/README.md](projects/cline/README.md) | 56.9k |
| [OpenHands](https://github.com/OpenHands/OpenHands) | [openhands-architecture.md](openhands-architecture.md)<br>[projects/openhands/README.md](projects/openhands/README.md) | 66.6k |
| [Claude Code Open](https://github.com/kill136/claude-code-open) | [claude-code-open-architecture.md](claude-code-open-architecture.md)<br>[projects/claude-code-open/README.md](projects/claude-code-open/README.md) | 66 |

## Docs in this repo

- `claude-dotfile-setup.md`
- `cline-architecture.md`
- `codex-dotfile-setup.md`
- `codex-architecture-breakdown.md`
- `claude-code-open-architecture.md`
- `eigent-architecture.md`
- `gemini-cli-architecture.md`
- `openhands-architecture.md`
- `opencode-architecture.md`
- `projects/codex/README.md`
- `projects/opencode/README.md`
- `projects/gemini-cli/README.md`
- `projects/cline/README.md`
- `projects/openhands/README.md`
- `projects/eigent/README.md`
- `projects/claude-code-open/README.md`
