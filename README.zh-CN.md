# grok-build-for-agent

简体中文 | [English](README.md)

这是一个**通用多 Agent 兼容**的 skill，允许 Hermes、Claude Code、OpenAI Codex、OpenClaw 以及其他兼容 Agent 调用和编排 xAI 官方的 **Grok Build CLI**。

**适配的 Grok Build CLI 版本**：`~v1.0.4`（截至 2026-08-15，**Grok 4.6** / `grok-4.6` 驱动）。

## 跨 Agent 兼容结构与安装

本仓库使用双布局 skill 结构以实现最大兼容性：

```text
repo/
├── SKILL.md                                      # 跨 Agent 通用的权威说明 (v1.4.0)
├── AGENTS.md                                    # Codex / OpenAI 项目指令
├── CLAUDE.md                                    # Claude Code 项目指令
├── agents/openai.yaml                           # 可选 OpenAI agent 元数据
└── skills/autonomous-ai-agents/grok-build-for-agent/SKILL.md  # 打包版 skill（字节级一致）
```

根目录 `SKILL.md` 与 `skills/autonomous-ai-agents/grok-build-for-agent/SKILL.md` 保持字节级一致。相关支持文件会出现在两个可安装位置旁边。

### 作为可复用 skill 安装

- **Claude Code**：复制 `skills/autonomous-ai-agents/grok-build-for-agent/` 到 `~/.claude/skills/autonomous-ai-agents/grok-build-for-agent/`。
- **OpenAI Codex / OpenAI agents**：在本仓库工作时保留 `AGENTS.md`；如需作为可复用 skill，复制 `skills/autonomous-ai-agents/grok-build-for-agent/` 到 `~/.agents/skills/autonomous-ai-agents/grok-build-for-agent/`。
- **OpenClaw**：复制 `skills/autonomous-ai-agents/grok-build-for-agent/` 到相应 skills 目录。
- **Hermes Agent**：复制 `skills/autonomous-ai-agents/grok-build-for-agent/` 到 `~/.hermes/skills/autonomous-ai-agents/grok-build-for-agent/`，然后开启新会话。
- **通用 AgentSkills 加载器**：使用包含 `SKILL.md` 的目录；skill 名称为 `grok-build-for-agent`。

本 skill 基于 xAI Grok Build 最新官方文档（docs.x.ai/build/*）及 ~v1.0.4 CLI 全面更新。Grok Build 仍在快速迭代，升级后请重新运行 `grok --help` 核对参数。

## 动因

虽然 Agent 可以通过 `xai-oauth` 或 xAI API 直接使用 Grok 模型，但很多用户希望利用 **Grok Build** 完整的 coding agent 能力（headless 执行、Plan Mode、丰富 tooling、MCP/plugin 支持、ACP、会话/worktree 管理、Grok 4.6 能力等）。

本 skill 为任何兼容 Agent 提供了一种干净、标准化的方式来调用 `grok` 二进制，支持 headless 模式或 ACP 模式，类似于 `codex` skill 调用 OpenAI Codex CLI 的方式，但现在具有更广泛的跨 Agent 生态兼容性。

## 官方文档支持的关键能力

- Headless 执行：`grok -p "prompt"` / `--single` / `--prompt-file` / `--prompt-json`
- 输出格式：`plain`、`json`、`streaming-json`（现已包含 tool/usage 等更丰富数据）；`--json-schema` 用于结构化验证
- 丰富 flag：`--always-approve`/`--yolo`、`--effort`、`--allow`/`--deny`、`--tools`/`--disallowed-tools`、`--max-turns`、`--no-plan`、`--no-subagents`、worktree、会话管理、`grok inspect`、MCP/plugin 命令等
- ACP 支持：`grok agent stdio`
- 模型：**Grok 4.6**（主要）
- 会话/worktree/export/import（支持从 Claude Code 导入）
- Headless 可靠性提升（等待 MCP、ask/plan 不再硬失败）

## 安装

添加本仓库为 skill tap 后（或手动复制 skill 目录）：

```bash
# Hermes 风格示例
hermes skills tap add yanyintingyou/grok-build-for-agent
# 然后按你的 Agent 方式安装 / 启用 skill
```

## 使用示例

对 Agent 说：

```text
使用 grok-build-for-agent 以 headless 模式运行以下任务：
任务：将用户认证模块重构为使用 JWT，并添加完善错误处理和测试。
工作目录：~/projects/my-app
使用 --always-approve --output-format streaming-json --max-turns 20
```

Agent 会生成符合当前 Grok Build 官方文档参数的 `terminal`（或等效）调用。

## 版本说明

本 skill 适配 **Grok Build ~v1.0.4**（2026-08-15 观测，Grok 4.6 驱动）。未来版本可能有破坏性变更；自动化前请先运行 `grok version` 和 `grok --help`。双布局与多 Agent metadata 确保广泛兼容性。

## License

MIT License.
