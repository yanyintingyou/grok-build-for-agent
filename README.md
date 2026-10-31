# grok-build-for-agent

[简体中文](README.zh-CN.md) | English

> **Project status: frozen — no further updates.**
> Final refresh: **2026-10-31**, targeting **Grok Build CLI v1.0.70** (2026-10-23) powered by **Grok 4.7** / `grok-4.7`.
> Grok Build ships roughly every day; this snapshot will go stale quickly. For current flags and behavior, use `grok --help` and the official docs at [docs.x.ai/build](https://docs.x.ai/build) instead of this repository.

A universal **multi-agent compatible skill** that allows Hermes, Claude Code, OpenAI Codex, OpenClaw, and other compatible agents to invoke and orchestrate the official **Grok Build CLI** from xAI.

**Target Grok Build CLI version**: `~v1.0.70` (as of 2026-10-31) powered by **Grok 4.7** / model `grok-4.7` (plus `grok-4.7-fast`, Grok Build–only).

## Cross-agent compatibility and installation

This repository uses a dual-layout skill structure for maximum compatibility:

```text
repo/
├── SKILL.md                                      # canonical cross-agent instructions (v1.5.0)
├── AGENTS.md                                     # Codex/OpenAI project instructions
├── CLAUDE.md                                     # Claude Code project instructions
├── agents/openai.yaml                            # optional OpenAI agent metadata
└── skills/autonomous-ai-agents/grok-build-for-agent/SKILL.md  # packaged skill copy (byte-identical)
```

The root `SKILL.md` and the packaged `skills/autonomous-ai-agents/grok-build-for-agent/SKILL.md` are intentionally byte-identical. Supporting files are available next to both copies where relevant.

### Install as a reusable skill

- **Claude Code**: copy `skills/autonomous-ai-agents/grok-build-for-agent/` to `~/.claude/skills/autonomous-ai-agents/grok-build-for-agent/`.
- **OpenAI Codex / OpenAI agents**: keep `AGENTS.md` when working in this repository, or copy `skills/autonomous-ai-agents/grok-build-for-agent/` to `~/.agents/skills/autonomous-ai-agents/grok-build-for-agent/`.
- **OpenClaw**: copy `skills/autonomous-ai-agents/grok-build-for-agent/` to `<workspace>/skills/autonomous-ai-agents/grok-build-for-agent/`, `<workspace>/.agents/skills/...`, or `~/.openclaw/skills/...`.
- **Hermes Agent**: copy `skills/autonomous-ai-agents/grok-build-for-agent/` to `~/.hermes/skills/autonomous-ai-agents/grok-build-for-agent/`, then start a new Hermes session.
- **Grok Build itself**: copy to `~/.grok/skills/grok-build-for-agent/` or `<repo>/.grok/skills/grok-build-for-agent/`.
- **Generic AgentSkills loaders**: use the directory containing `SKILL.md`; skill name is `grok-build-for-agent`.

This skill was based on the xAI Grok Build docs (docs.x.ai/build/*), the CLI reference, and CLI behavior observed at ~v1.0.70. Re-check `grok --help` after upgrades — Grok Build evolves faster than this snapshot.

## Motivation

While agents can use Grok models directly via `xai-oauth` or the xAI API, many users want the full **Grok Build** coding agent experience (headless execution, Plan Mode, rich tooling, MCP/plugin support, ACP, session/worktree management, Grok 4.7 capabilities, etc.).

This skill provides a clean, standardized way for any compatible agent to call the `grok` binary in headless mode or ACP mode — similar to how `codex` skills work with OpenAI Codex CLI, but now universally applicable across agent ecosystems.

## Key Capabilities (from Official ~v1.0.70 Docs)

- Headless execution: `grok -p "prompt"` / `--single` / `--prompt-file` / `--prompt-json`
- Output formats: `plain`, `json`, `streaming-json`, `streaming-messages-json` (Anthropic Messages wire format); `--json-schema` for validation
- Rich flag set: `--always-approve`/`--yolo`, `--effort` (low/medium/high/xhigh), `--allow`/`--deny`, `--tools`/`--disallowed-tools`, `--max-turns`, `--no-plan`, `--no-subagents`, `--background-wait-timeout`, `--plugin-dir`, worktree, session management, MCP/plugin commands, `grok inspect`, etc.
- Sandbox profiles: `off` / `workspace` / `devbox` / `read-only` / `strict` (network enforcement is Linux-only)
- ACP support: `grok agent stdio` (text streams via `session/update`, not the `session/prompt` result)
- Models: **Grok 4.7** (`grok-4.7`, 500k context) and `grok-4.7-fast` (Cursor + Grok Build only)
- Session/worktree/export/import (including from Claude Code); worktrees inherit saved tool approvals
- Newer CLI surface: `grok wrap`, `grok completions`, `grok setup`, `grok update --check|--version|--alpha|--stable`

## Installation

After adding this repository as a skill tap (or manually copying the skill directory):

```bash
# example for Hermes-style managers
hermes skills tap add yanyintingyou/grok-build-for-agent
# then install / enable the skill as appropriate for your agent
```

## Usage Example

Ask your agent:

```text
Use grok-build-for-agent to run the following task with Grok Build in headless mode:
Task: Refactor the user authentication module to use JWT with proper error handling and tests.
Working directory: ~/projects/my-app
Use --always-approve --output-format streaming-json --max-turns 20
```

The agent will generate an appropriate `terminal` (or equivalent) call using the documented flags from the current Grok Build release.

## Version Notice

This skill targets **Grok Build ~v1.0.70** (observed 2026-10-31, powered by Grok 4.7). Future versions may introduce breaking changes; always run `grok version` and `grok --help` locally before automation.

This repository is **no longer maintained**. No further commits, version bumps, or compatibility fixes will be made. If you need up-to-date guidance, derive it from `grok --help` and the official docs; the dual-layout structure and multi-agent metadata here remain valid as a packaging example.

## License

MIT License.
