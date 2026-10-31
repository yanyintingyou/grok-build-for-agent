---
name: grok-build-for-agent
description: Use when any compatible agent (Hermes, Claude Code, OpenAI Codex, OpenClaw, generic AgentSkills loaders, ACP-compatible agents, etc.) wants to call or orchestrate the official Grok Build CLI (xAI) for coding tasks, especially in headless mode or via ACP.
when-to-use: grok build, grok cli, xai coding agent, headless grok, grok agent stdio, grok 4.7, run grok from another agent
version: 1.5.0
author: yanyintingyou
license: MIT
platforms: [linux, macos, windows]
metadata:
  category: autonomous-ai-agents
  tags: [grok, xai, coding-agent, headless, acp, multi-agent, grok-4.7, mcp, plugin, skills]
  homepage: https://github.com/yanyintingyou/grok-build-for-agent
compatibility: [Hermes Agent, Claude Code, OpenAI Codex, OpenClaw, generic AgentSkills loaders, ACP-compatible agents]
---

# Grok Build for Agent

This skill enables **any compatible agent** to invoke the official **Grok Build CLI** (`grok`) from xAI for coding tasks.

**Important**: Refreshed against official xAI Grok Build docs (docs.x.ai/build/*), the CLI reference, the headless/ACP pages, and the public release changelog on **2026-10-31**. Target version: **v1.0.70** (released 2026-10-23), powered by **Grok 4.7** / model id `grok-4.7`. Grok Build ships almost daily — always run `grok version` and `grok --help` (and subcommand `--help`) locally before relying on exact behavior.

**Maintenance status**: this repository is frozen at the v1.0.70 snapshot and will not be updated further. Treat everything below as a dated reference, not a live contract with the CLI.

## When to Use

Use this skill when you want a compatible agent to:
- Run Grok Build in headless mode (`grok -p "..."` / `--single`, or via `--prompt-file` / `--prompt-json`)
- Execute complex one-off or long-running coding tasks via the Grok Build agent
- Integrate Grok Build into automated workflows, CI, or multi-agent orchestration
- Use ACP mode (`grok agent stdio`) for structured JSON-RPC integration with other systems or agents
- Manage sessions, worktrees, MCP servers, plugins, skills, memory, export/import, and configuration via agent-driven terminal calls
- Leverage Grok 4.7 / `grok-4.7-fast` coding and agentic capabilities

**Do not use** this skill if you want the agent to use Grok models directly via the `xai-oauth` provider or the raw xAI API (that is a separate, recommended path for simple chat/completion use cases).

## Prerequisites

- Grok Build CLI installed and available on `PATH`.
  - Official install: `curl -fsSL https://x.ai/cli/install.sh | bash` (Linux/macOS) or `irm https://x.ai/cli/install.ps1 | iex` (Windows PowerShell).
  - Verify with `grok version` (or `grok --version`).
- Authenticated: first launch opens browser login (`grok login`, `--device-auth` for headless/remote boxes); or set `XAI_API_KEY` for API-key auth.
- Running inside a Git repository is strongly recommended (worktrees, diffs, session resume all assume it).
- For advanced automation, understand permission modes, tool allow/deny rules, and sandbox profiles.

## Current Version Check (as of 2026-10-31)

- CLI version target: **grok v1.0.70** (stable channel, 2026-10-23; release cadence ~1 day, 179 releases tracked since 2026-05)
- Primary model: **Grok 4.7** (`grok-4.7`, 500k context, text+image input, text output, reasoning effort `low` / `medium` / `high` / `xhigh`)
- `grok-4.7-fast`: same model at 2x price, **only available inside Cursor and Grok Build**, not on the public API
- Behaviors relevant to agent orchestration:
  - Headless `-p` sessions wait for MCP servers before tool use
  - Headless `-p` waits for the follow-up turn a finished background task starts before exiting; `--background-wait-timeout` turns a stuck wait into an error instead of a false success (added in 1.0.62)
  - Output formats now include `streaming-messages-json` (Anthropic Messages wire format) alongside `plain` / `json` / `streaming-json`
  - Worktrees inherit saved tool approvals; `grok --resume` matches titles fuzzily and can restore sessions across folders
  - `grok update`, `grok login`, `grok models`, `grok sessions`, `grok export` no longer crash on closed/unwritable stdout (safe to pipe into `head`)

Always verify locally:
```bash
grok version
grok --help
grok agent --help
grok agent stdio --help
grok inspect --json
grok models
grok update --check
```

## One-Shot Headless Execution

```bash
grok -p "Your prompt here"
# aliases / alternatives:
# grok --single "Your prompt here"
# grok --prompt-file /path/to/prompt.md
# grok --prompt-json '[{"type":"text","text":"..."}]'
```

**Recommended flags for automation** (cross-checked against the current CLI reference; re-verify after upgrades):

| Flag | Purpose |
|------|---------|
| `-p, --single <PROMPT>` / `--prompt-file <PATH>` / `--prompt-json <JSON>` | Run single-turn headless prompt. Prefer `--prompt-file` for long/complex prompts to avoid shell escaping and ARG_MAX issues. |
| `-m, --model <MODEL>` | Select model (`grok-4.7`, `grok-4.7-fast`, or a custom id from `~/.grok/config.toml`). |
| `--effort <LEVEL>` | Reasoning effort: `low`, `medium`, `high` (default), `xhigh`. Gated by what the model supports. |
| `--output-format <plain\|json\|streaming-json\|streaming-messages-json>` | Control output for programmatic consumption. `streaming-json` = NDJSON of ACP session updates; `streaming-messages-json` = Anthropic Messages wire format (pair with `--include-partial-messages` for token deltas). |
| `--json-schema <SCHEMA>` | Constrain/validate the final answer as JSON Schema; implies `--output-format json`. |
| `--always-approve` / `--yolo` | Auto-approve all tool calls. Explicit `deny` rules and PreToolUse hooks still apply. Use only for well-scoped, low-risk tasks. |
| `--allow <RULE>` / `--deny <RULE>` | Fine-grained permission rules over `Bash` / `Edit` / `Read` / `Grep` / `MCPTool` / `WebFetch` / `WebSearch` patterns. `deny` always wins. |
| `--tools <LIST>` / `--disallowed-tools <LIST>` | Explicitly allow or remove built-in tools. Claude Code aliases (`--allowedTools` / `--disallowedTools`) are accepted. |
| `--max-turns <N>` | Hard limit on agent turns (prevents runaway loops). Always set it in agent-orchestrated runs. |
| `--no-plan` / `--no-subagents` / `--no-memory` / `--disable-web-search` | Disable those features for this run. |
| `--experimental-memory` | Enable cross-session memory for this run. |
| `--sandbox <PROFILE>` | `off` (default) / `workspace` / `devbox` / `read-only` / `strict`, or a custom profile from `sandbox.toml`. Env equivalent: `GROK_SANDBOX`. |
| `--background-wait-timeout <DURATION>` | Bound how long `-p` waits for background-task follow-up turns before failing instead of hanging. |
| `--rules <TEXT>` / `--system-prompt-override <TEXT>` | Append rules, or fully replace the system prompt. Claude Code aliases `--append-system-prompt` / `--system-prompt` work too. |
| `--plugin-dir <PATH>` | Load an extra plugin directory for this run (skills, agents, hooks, MCP servers). |
| `--cwd <PATH>` | Set working directory. |
| `-r, --resume [<ID>]` / `-c, --continue` | Resume by id (or most recent) / continue the latest session in this directory. |
| `--fork-session` | Fork into a new session ID when resuming. |
| `-s, --session-id <UUID>` | Use a specific session UUID for determinism. |
| `-w, --worktree [<NAME>]` / `--ref <REF>` | Start in a new git worktree (optionally from a specific ref). |
| `--no-auto-update` | Skip background update checks (recommended in scripts/CI/ACP). Persistent: `[cli] auto_update = false` in `~/.grok/config.toml`. |
| `--no-alt-screen` | Run inline (no alternate screen / fullscreen TUI takeover). |
| `--oauth` | Force the browser OAuth flow instead of a stored credential. |

**Example with an agent `terminal` tool (Hermes or similar):**
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

Structured result with a schema:
```bash
grok -p "Audit this repo for unsafe shell usage" --json-schema '{"type":"object","properties":{"findings":{"type":"array"}}}' --max-turns 8
```

## Background / Long-Running Tasks

For longer tasks use background mode + notification (agent-side):
```python
terminal(
    command='grok --no-auto-update -p "Implement the full feature described in the ticket" --always-approve --output-format streaming-json --max-turns 30 --background-wait-timeout 30m',
    workdir="~/projects/my-repo",
    background=true,
    pty=true,
    notify_on_complete=true
)
```

Headless runs also drive Grok's own background tasks: a finished background task may start a follow-up turn, and `-p` waits for it. Use `--background-wait-timeout` when the caller needs a bounded runtime rather than a silent hang.

## ACP Mode (Agent Client Protocol)

```bash
grok agent stdio
```

Runs Grok Build as a structured JSON-RPC agent over stdin/stdout — the right mode for IDE extensions, SDKs, and custom orchestration layers.

- `initialize` with `protocolVersion: 1` and client capabilities (`fs.readTextFile`, `fs.writeTextFile`, `terminal`).
- Authenticate with a method id from `init.authMethods`: `xai.api_key` (when `XAI_API_KEY` is set) or `cached_token` (after `grok login`), passing `_meta: { headless: true }`.
- `session/new` takes `cwd` and `mcpServers`; `session/prompt` returns **completion metadata only** (`stopReason` included).
- Assistant text arrives asynchronously via `session/update` events with `sessionUpdate: "agent_message_chunk"` — accumulate `content.text` yourself; do not read it from the `session/prompt` result.

## Using grok inspect, models, mcp, plugin, sessions, update

- `grok inspect [--json]` — Discover rules, skills, plugins, hooks, MCP servers, and config sources for the current directory.
- `grok models` — List available models.
- `grok mcp <list|add|remove|doctor>` — Manage MCP servers (stdio and HTTP; `--scope project` writes `.grok/config.toml`).
- `grok plugin <list|install|uninstall|update|enable|disable|details|validate>` + `grok plugin marketplace <list|add|remove|update>`
- `grok sessions <list|search|delete>` / `grok export <session-id> [output]` / `grok import [targets...]` (including from Claude Code)
- `grok worktree <list|show|rm|gc>` / `grok memory clear [--workspace|--global|--all]` / `grok du` / `grok dashboard`
- `grok update [--check] [--version <V>] [--alpha|--stable]` — check or install a specific channel/version
- `grok wrap <command...>` — run a command in a local PTY, forwarding OSC 52 clipboard writes
- `grok completions <shell>` — emit a completion script
- `grok setup` — fetch and install managed configuration

These are extremely useful for agents that need to inspect and extend the environment at runtime.

## Skills, Plugins, and Discovery Paths

Grok Build reads skills (this repository's format) from:
- `./.grok/skills/` (project, walking up to the repo root)
- `~/.grok/skills/` (user)
- `skills/` inside any enabled plugin
- `[skills] paths` in `~/.grok/config.toml`
- `~/.agents/skills/`

`SKILL.md` frontmatter fields Grok actually honors: `name`, `description`, `when-to-use` (alias `when_to_use`), `paths`, `allowed-tools`, `argument-hint`, `user-invocable`, `disable-model-invocation`, `metadata`. Fields `model`, `effort`, `license`, `compatibility` are accepted but ignored. Plugins load from `./.grok/plugins/`, `~/.grok/plugins/`, marketplaces under `~/.grok/plugins/marketplaces/`, `[plugins] paths`, or `--plugin-dir`. Grok also reads Claude Code's `CLAUDE.md`, `.claude/rules/`, plugins, skills, agents, hooks, and MCPs with zero configuration.

## Structured Output & Validation

Prefer `--output-format json` or `streaming-json`, or `--json-schema` for strict schema validation. `json` returns one object at the end (text, stopReason, sessionId, requestId); `streaming-json` emits NDJSON of ACP session updates including tool calls, results, and usage. `streaming-messages-json` is the drop-in option when the caller already speaks the Anthropic Messages streaming format.

## Permission Modes & Safety for Agent-Driven Automation

Default mode is **Ask**. The three modes are `Ask`, `Auto` (classifier auto-approves safe tools), and `Always-approve` (`--always-approve` / `--yolo`); `Shift+Tab` cycles Normal → Plan → Auto → Always-approve. Set a persistent default in `~/.grok/config.toml` (user-level only, not project):

```toml
[ui]
permission_mode = "auto"  # "ask" | "auto" | "always-approve"
```

For automation:
- Use `--always-approve` only for narrow, well-understood tasks.
- Prefer explicit `--allow` / `--deny` rules plus `--tools` / `--disallowed-tools`; `deny` overrides `allow`.
- Add `--sandbox` when running against untrusted codebases. Sandbox restricts what an approved call can touch; permissions decide whether it runs at all.
- Limit `--max-turns`, and bound waits with `--background-wait-timeout`.
- Plan mode gates edit tools in the parent session only; subagents inherit the permission mode instead. It is not a containment boundary, and shell redirection can still write.

## Sandbox Profiles

| Profile | Read | Write | Child-process network |
|---------|------|-------|-----------------------|
| `off` (default) | unrestricted | unrestricted | allowed |
| `workspace` | anywhere | CWD, `~/.grok/`, temp | allowed |
| `devbox` | anywhere | top-level dirs except `/data` | allowed |
| `read-only` | anywhere | only `~/.grok/` and temp | blocked |
| `strict` | CWD + system paths | CWD, `~/.grok/`, temp | blocked |

Network blocking is enforced on Linux only; on macOS `read-only` / `strict` are no-ops for child-process networking. Model API and web tools are never sandboxed. `~/.grok/` stays writable in every profile. Custom profiles live in `~/.grok/sandbox.toml` or `.grok/sandbox.toml` and can `extends` a built-in.

## Configuration Notes

- Global config: `~/.grok/config.toml` (Windows: `%USERPROFILE%\.grok\config.toml`); `$GROK_HOME` overrides the root.
- Project config: `.grok/config.toml` (no permission-mode override there).
- `grok inspect [--json]` shows everything Grok discovered — the fastest way to debug a misconfigured environment.
- Persistent auto-update disable: `[cli] auto_update = false`.
- Custom models, default model, reasoning effort, subagent model routing (`[subagents.models]`), and skill paths (`[skills] paths`) are all config-file level.
- Sessions are stored under `~/.grok/sessions`.

## Rules

1. **Always** include `--no-auto-update` in scripts, CI, ACP, and automated environments.
2. Prefer structured output (`json`, `streaming-json`, `streaming-messages-json`, or `--json-schema`) for reliable parsing by the calling agent.
3. For long or complex prompts, use `--prompt-file <PATH>` to avoid quoting/escaping and command-line length limits.
4. Use `--always-approve` and permissive modes **only** for well-scoped, low-risk tasks. Prefer explicit allow/deny lists.
5. Authentication: headless/server use `XAI_API_KEY`; interactive opens browser (`grok login --device-auth` for device codes).
6. Re-check `grok version`, `grok --help`, and subcommand helps after every upgrade — this snapshot describes v1.0.70 behavior observed on 2026-10-31 and is not maintained further.
7. When using MCPs or plugins, verify with `grok mcp` / `grok plugin` and `grok inspect`.
8. Limit `--max-turns` and use tool allowlists to control cost and blast radius in agent-orchestrated runs.
9. Do not treat plan mode as a sandbox; combine `--deny` rules and `--sandbox` when the codebase is untrusted.

## Limitations (Strictly Based on Latest Docs)

- Many rich TUI features (mouse, fullscreen visuals, some Plan Mode UI, subagent views, dashboard) are unavailable or degraded in pure headless / ACP mode.
- Flag set and behavior change between minor versions, sometimes daily; always verify locally.
- Enterprise features (`dontAsk` headless mode, locked always-approve, managed `requirements.toml` pins) depend on deployment context.
- ACP method coverage keeps evolving; test `grok agent stdio` end-to-end for production integrations.
- This skill provides orchestration guidance and curated flag recommendations, not an exhaustive CLI reference.

## Recommended Pattern in Compatible Agents

When delegating work to Grok Build:

1. The calling agent generates a clear, self-contained prompt (or prompt file).
2. The agent calls its `terminal` tool (or equivalent) with carefully chosen `grok -p ...` / `--prompt-file` flags, preferring structured output and safety limits.
3. For very long tasks, use background execution + completion notification, with `--background-wait-timeout` as a bound.
4. For deep integration, spawn `grok agent stdio` and communicate via ACP/JSON-RPC, consuming text from `session/update`.
5. Use `grok inspect`, `grok mcp`, `grok plugin`, and the session/worktree commands to manage environment and state.
6. After completion, export transcripts or import context from other tools (e.g. Claude Code).

This keeps the calling agent as the orchestrator while fully leveraging Grok Build's coding agent capabilities (Grok 4.7 + tooling).

The dual-layout design (root SKILL.md + packaged copy) ensures this skill works seamlessly across Hermes, Claude Code, OpenAI Codex, OpenClaw, and generic AgentSkills loaders.
