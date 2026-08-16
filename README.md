# grok-build-for-agent

[简体中文](README.zh-CN.md) | English

A universal **multi-agent compatible skill** that allows Hermes, Claude Code, OpenAI Codex, OpenClaw, and other compatible agents to invoke and orchestrate the official **Grok Build CLI** from xAI.

**Target Grok Build CLI version**: `~v1.0.4` (as of 2026-08-15) powered by **Grok 4.6** / model `grok-4.6`.

## Cross-agent compatibility and installation

This repository uses a dual-layout skill structure for maximum compatibility:

```text
repo/
├── SKILL.md                                      # canonical cross-agent instructions (v1.4.0)
├── AGENTS.md                                    # Codex/OpenAI project instructions
├── CLAUDE.md                                    # Claude Code project instructions
├── agents/openai.yaml                           # optional OpenAI agent metadata
└── skills/autonomous-ai-agents/grok-build-for-agent/SKILL.md  # packaged skill copy (byte-identical)
```

The root `SKILL.md` and the packaged `skills/autonomous-ai-agents/grok-build-for-agent/SKILL.md` are intentionally byte-identical. Supporting files are available next to both copies where relevant.

### Install as a reusable skill

- **Claude Code**: copy `skills/autonomous-ai-agents/grok-build-for-agent/` to `~/.claude/skills/autonomous-ai-agents/grok-build-for-agent/`.
- **OpenAI Codex / OpenAI agents**: keep `AGENTS.md` when working in this repository, or copy `skills/autonomous-ai-agents/grok-build-for-agent/` to `~/.agents/skills/autonomous-ai-agents/grok-build-for-agent/`.
- **OpenClaw**: copy `skills/autonomous-ai-agents/grok-build-for-agent/` to `<workspace>/skills/autonomous-ai-agents/grok-build-for-agent/`, `<workspace>/.agents/skills/...`, or `~/.openclaw/skills/...`.
- **Hermes Agent**: copy `skills/autonomous-ai-agents/grok-build-for-agent/` to `~/.hermes/skills/autonomous-ai-agents/grok-build-for-agent/`, then start a new Hermes session.
- **Generic AgentSkills loaders**: use the directory containing `SKILL.md`; skill name is `grok-build-for-agent`.

This skill is based on the latest xAI Grok Build docs (docs.x.ai/build/*) and CLI help from ~v1.0.4. Re-check `grok --help` after upgrades because Grok Build continues to evolve rapidly.

## Motivation

While agents can use Grok models directly via `xai-oauth` or the xAI API, many users want the full **Grok Build** coding agent experience (headless execution, Plan Mode, rich tooling, MCP/plugin support, ACP, session/worktree management, Grok 4.6 capabilities, etc.).

This skill provides a clean, standardized way for any compatible agent to call the `grok` binary in headless mode or ACP mode — similar to how `codex` skills work with OpenAI Codex CLI, but now universally applicable across agent ecosystems.

## Key Capabilities (from Official ~v1.0.4 Docs)

- Headless execution: `grok -p "prompt"` / `--single` / `--prompt-file` / `--prompt-json`
- Output formats: `plain`, `json`, `streaming-json` (now richer with tool/usage data); plus `--json-schema` for validation
- Rich flag set: `--always-approve`/`--yolo`, `--effort`, `--allow`/`--deny`, `--tools`/`--disallowed-tools`, `--max-turns`, `--no-plan`, `--no-subagents`, worktree, session management, MCP/plugin commands, `grok inspect`, etc.
- ACP support: `grok agent stdio`
- Model: **Grok 4.6** (primary)
- Session/worktree/export/import (including from Claude Code)
- Headless reliability improvements (MCP wait, no hard-fail on ask/plan)

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

This skill targets **Grok Build ~v1.0.4** (observed 2026-08-15, powered by Grok 4.6). Future versions may introduce breaking changes; always run `grok version` and `grok --help` locally before automation. The dual-layout and multi-agent metadata ensure broad compatibility.

## License

MIT License.
