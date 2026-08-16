---
name: grok-build-for-agent
description: Use when any compatible agent (Hermes, Claude Code, OpenAI Codex, OpenClaw, generic AgentSkills loaders, ACP-compatible agents, etc.) wants to call or orchestrate the official Grok Build CLI (xAI) for coding tasks, especially in headless mode or via ACP.
version: 1.4.0
author: yanyintingyou
license: MIT
platforms: [linux, macos, windows]
metadata: {"category":"autonomous-ai-agents","tags":["grok","xai","coding-agent","headless","acp","multi-agent","grok-4.6","mcp","plugin","skills"],"compatibility":{"agents":["Hermes Agent","Claude Code","OpenAI Codex","OpenClaw","generic AgentSkills loaders","ACP-compatible agents"]},"openclaw":{"homepage":"https://github.com/yanyintingyou/grok-build-for-agent"}}
---

# Grok Build for Agent

This skill enables **any compatible agent** to invoke the official **Grok Build CLI** (`grok`) from xAI for coding tasks.

**Important**: This skill was comprehensively refreshed against the latest official xAI Grok Build documentation (docs.x.ai/build/*), changelog, CLI reference, and headless/ACP pages on **2026-08-15**. Target version: **v1.0.4** (powered by **Grok 4.6** / model id `grok-4.6`). Grok Build evolves rapidly — always run `grok version` and `grok --help` (and subcommand `--help`) locally before relying on exact behavior.

## When to Use

Use this skill when you want a compatible agent to:
- Run Grok Build in headless mode (`grok -p "..."` / `--single`, or via `--prompt-file` / `--prompt-json`)
- Execute complex one-off or long-running coding tasks via the Grok Build agent
- Integrate Grok Build into automated workflows, CI, or multi-agent orchestration
- Use ACP mode (`grok agent stdio`) for structured JSON-RPC integration with other systems or agents
- Manage sessions, worktrees, MCP servers, plugins, memory, export/import, and configuration via agent-driven terminal calls
- Leverage Grok 4.6's strong coding and agentic capabilities

**Do not use** this skill if you want the agent to use Grok models directly via the `xai-oauth` provider or raw API (that is a separate, recommended path for simple chat/completion use cases).

## Prerequisites

- Grok Build CLI installed and available on `PATH`.
  - Official install: `curl -fsSL https://x.ai/cli/install.sh | bash` (Linux/macOS) or `irm https://x.ai/cli/install.ps1 | iex` (Windows).
  - Verify with `grok version` (or `grok --version`).
- Authenticated (first launch opens browser login; headless/server use `export XAI_API_KEY=...` or `grok login` / device-auth).
- Running inside a Git repository is strongly recommended.
- For advanced automation, understand permission modes, tool allow/deny lists, and sandbox profiles.

## Current Version Check (as of 2026-08-15)

- CLI version target: **grok ~1.0.4** (stable channel; check `grok version` locally)
- Primary model powering Grok Build: **Grok 4.6** (`grok-4.6`)
- Key improvements since 0.2.9x / 1.0.x relevant to agents:
  - Headless (`grok -p`) sessions wait for MCP servers before tool use
  - Non-interactive sessions no longer fail when the agent asks for user input or plan approval
  - Streaming-json includes tool calls, results, and usage; JSON output includes token usage/cost
  - Faster subagent spawning, better permission/auto mode, improved worktree & session handling
  - Richer `grok inspect`, plugin/marketplace, dashboard, and memory controls

Always verify locally:
```bash
grok version
grok --help
grok agent --help
grok agent stdio --help
grok inspect --json
grok models
```

## One-Shot Headless Execution

```bash
grok -p "Your prompt here"
# aliases / alternatives:
# grok --single "Your prompt here"
# grok --prompt-file /path/to/prompt.md
# grok --prompt-json '[{"type":"text","text":"..."}]'
```

**Recommended flags for automation** (verified against current official docs + CLI reference):

| Flag | Purpose |
|------|---------|
| `-p, --single <PROMPT>` / `--prompt-file <PATH>` / `--prompt-json <JSON>` | Run single-turn headless prompt (prefer `--prompt-file` for long/complex prompts to avoid shell escaping and ARG_MAX issues). |
| `-m, --model <MODEL>` | Select model (e.g. `grok-4.6` or custom via `~/.grok/config.toml`). |
| `--effort <LEVEL>` | Control reasoning effort (low / medium / high / xhigh where supported). |
| `--json-schema <SCHEMA>` | Constrain/validate final output as JSON Schema (excellent for structured parsing by agents). |
| `--output-format <plain\|json\|streaming-json>` | Control output format for programmatic consumption. Streaming-json now includes tool calls/results/usage. |
| `--always-approve` / `--yolo` | Auto-approve all tool calls (deny rules & PreToolUse hooks still apply; use only for well-scoped, low-risk tasks). |
| `--allow <RULE>` / `--deny <RULE>` | Fine-grained permission rules (Bash/Edit/Read/Grep/MCPTool/WebFetch/WebSearch patterns). |
| `--tools <LIST>` / `--disallowed-tools <LIST>` | Explicitly allow or remove built-in tools. Claude Code aliases supported. |
| `--max-turns <N>` | Hard limit on agent turns (prevents runaway loops). |
| `--no-plan` | Disable Plan Mode. |
| `--no-subagents` | Disable subagent spawning. |
| `--no-memory` / memory config | Control cross-session memory (prefer config or env in recent versions). |
| `--disable-web-search` | Disable web search and web_fetch tools. |
| `--sandbox <PROFILE>` | Apply sandbox profile for FS/network access. |
| `--rules <TEXT>` / `--system-prompt-override <TEXT>` | Append rules or fully override system prompt. |
| `--cwd <PATH>` | Set working directory. |
| `-r, --resume [<ID>]` / `-c, --continue` | Resume or continue sessions. |
| `--fork-session` | Fork into new session ID when resuming. |
| `-s, --session-id <UUID>` | Use specific session UUID for determinism. |
| `-w, --worktree [<NAME>]` / `--ref <REF>` | Start in new git worktree (optionally from specific ref). |
| `--no-auto-update` | Skip background updates (recommended in scripts/CI/ACP). Persistent: `[cli] auto_update = false` in `~/.grok/config.toml`. |
| `--no-alt-screen` | Run inline (no alternate screen / fullscreen TUI takeover). |

**Example with agent `terminal` tool (Hermes or similar):**
```python
terminal(
    command='grok --no-auto-update -p "Refactor the authentication module to use JWT with proper error handling" --output-format streaming-json --always-approve --max-turns 15',
    workdir="~/projects/my-repo",
    pty=true
)
```

Prefer `--prompt-file` for complex prompts:
```bash
grok --no-auto-update --prompt-file ./task.md --output-format streaming-json --always-approve --max-turns 20
```

## Background / Long-Running Tasks

For longer tasks use background mode + notification (agent-side):
```python
terminal(
    command='grok --no-auto-update -p "Implement the full feature described in the ticket" --always-approve --output-format streaming-json --max-turns 30',
    workdir="~/projects/my-repo",
    background=true,
    pty=true,
    notify_on_complete=true
)
```

## ACP Mode (Agent Client Protocol)

```bash
grok agent stdio
```

This runs Grok Build as a structured JSON-RPC agent over stdin/stdout. Ideal for deep integration with other agent frameworks, IDEs, or custom orchestration layers. Use when you need reliable machine-to-machine communication rather than one-shot prompts. Authenticate via cached token or `XAI_API_KEY` (headless meta supported).

## Using grok inspect, models, mcp, plugin, sessions, etc.

- `grok inspect [--json]` — Discover rules, skills, plugins, hooks, MCP servers, and config sources in the current directory.
- `grok models` — List available models and their configurations.
- `grok mcp <list|add|remove|doctor>` — Manage MCP servers.
- `grok plugin <list|install|uninstall|update|enable|disable|details|validate>` + `grok plugin marketplace ...` — Manage plugins and marketplaces.
- `grok sessions <list|search|delete>` / `grok export` / `grok import` (incl. from Claude Code)
- `grok worktree <list|show|rm|gc>` / `grok memory clear` / `grok du` / `grok dashboard` / `grok update`

These are extremely useful for agents to dynamically inspect and extend the environment.

## Structured Output & Validation

Prefer `--output-format json` or `streaming-json`, or `--json-schema` for strict schema validation. Recent versions enrich JSON/streaming-json with tool calls, results, usage, and cost — making it much easier for the calling agent to reliably parse results.

## Permission Modes & Safety for Agent-Driven Automation

Grok Build supports rich permission controls (ask / auto / always-approve). For automation:
- Use `--always-approve` (or `--yolo`) only for narrow, well-understood tasks.
- Prefer explicit `--allow` / `--deny` rules and `--tools` / `--disallowed-tools`.
- Consider `--sandbox` profiles for untrusted codebases.
- Monitor with `grok inspect` and limit `--max-turns`.
- Auto mode (classifier) auto-approves many safe/read-only operations while still respecting deny rules & hooks.
- In headless/1.0.x: sessions wait for MCP readiness; non-interactive runs no longer hard-fail on ask/plan prompts (behavior improved for agent orchestration).

## Configuration Notes

- Global config: `~/.grok/config.toml` (or `$GROK_HOME/config.toml`)
- Use `grok inspect` (or `--json`) to verify discovered configuration, skills, MCPs, etc.
- Persistent auto-update disable: `[cli] auto_update = false`
- Custom models / default model / reasoning effort can be defined in config.toml and selected with `-m` / `--effort`.
- Many features (MCP, permissions, memory, tools) can be toggled via env vars, config, or flags.

## Rules

1. **Always** include `--no-auto-update` in scripts, CI, ACP, and automated environments.
2. Prefer structured output (`json`, `streaming-json`, or `--json-schema`) for reliable parsing by the calling agent.
3. For long or complex prompts, use `--prompt-file <PATH>` to avoid quoting/escaping and command-line length limits.
4. Use `--always-approve` and permissive modes **only** for well-scoped, low-risk tasks. Prefer explicit allow/deny lists.
5. Authentication: headless/server use `XAI_API_KEY`; interactive opens browser (`grok login --device-auth` available).
6. Re-check `grok version`, `grok --help`, and subcommand helps after every upgrade — this skill targets ~v1.0.4 behavior observed on 2026-08-15.
7. When using MCPs or plugins, verify with `grok mcp` / `grok plugin` and `grok inspect`.
8. Limit `--max-turns` and use tool allowlists to control cost and blast radius in agent-orchestrated runs.

## Limitations (Strictly Based on Latest Docs)

- Many rich TUI features (mouse, fullscreen visuals, some Plan Mode UI, subagent views) are not available or fully usable in pure headless / ACP mode.
- Flag set and behavior can change between minor versions; always verify locally.
- Enterprise/sandbox/advanced permission features (e.g. locked modes, dontAsk) may require specific deployment context.
- Full ACP JSON-RPC method documentation continues to evolve; test `grok agent stdio` thoroughly for production integrations.
- This skill provides orchestration guidance and curated flag recommendations, not an exhaustive CLI reference.

## Recommended Pattern in Compatible Agents

When delegating work to Grok Build:

1. The calling agent generates a clear, self-contained prompt (or prompt file).
2. The agent calls its `terminal` tool (or equivalent) with carefully chosen `grok -p ...` / `--prompt-file` flags, preferring structured output and safety limits.
3. For very long tasks, use background execution + completion notification.
4. For deep integration, spawn `grok agent stdio` and communicate via ACP/JSON-RPC.
5. Use `grok inspect`, `grok mcp`, `grok plugin`, session/worktree commands to manage environment and state.
6. After completion, optionally export transcripts or import context from other tools (e.g. Claude Code).

This keeps the calling agent as the orchestrator while fully leveraging Grok Build's powerful coding agent capabilities (Grok 4.6 + tooling).

The dual-layout design (root SKILL.md + packaged copy) ensures this skill works seamlessly across Hermes, Claude Code, OpenAI Codex, OpenClaw, and generic AgentSkills loaders.
