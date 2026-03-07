# Subagents 与对比

> **来源:** https://code.claude.com/docs/en/sub-agents + https://code.claude.com/docs/en/agent-teams
> **获取时间:** 2026-03-06
> **版本锚点:** Claude Code 2.1.x

## Subagents 概述

Subagents = 单会话内的专用 AI 助手，隔离上下文，完成后返回结果给主对话。

### 内置 Subagents

| Agent | 模型 | 工具权限 | 用途 |
|---|---|---|---|
| **Explore** | Haiku（快速） | 只读 | 代码搜索、文件发现、代码库探索 |
| **Plan** | 继承主会话 | 只读 | plan mode 下的代码库研究 |
| **General-purpose** | 继承主会话 | 全部 | 复杂多步任务 |
| **Bash** | 继承 | 终端命令 | 独立上下文中运行命令 |
| **Claude Code Guide** | Haiku | — | 回答 Claude Code 相关问题 |

### 自定义 Subagent 创建

**方式 1 — `/agents` 命令：** 交互式引导创建
**方式 2 — Markdown 文件：** 手动创建

```markdown
---
name: code-reviewer
description: Expert code review. Use proactively after code changes.
tools: Read, Glob, Grep, Bash
model: sonnet
---
You are a senior code reviewer...
```

### 存储位置（优先级从高到低）

| 位置 | 范围 | 优先级 |
|---|---|---|
| `--agents` CLI flag | 当前会话 | 1（最高） |
| `.claude/agents/` | 当前项目 | 2 |
| `~/.claude/agents/` | 所有项目 | 3 |
| Plugin `agents/` | 插件启用处 | 4（最低） |

### 关键 Frontmatter 字段

| 字段 | 必需 | 说明 |
|---|---|---|
| `name` | 是 | 小写+连字符标识符 |
| `description` | 是 | Claude 据此决定何时委派 |
| `tools` | 否 | 允许的工具（默认继承全部） |
| `disallowedTools` | 否 | 禁止的工具 |
| `model` | 否 | `sonnet`/`opus`/`haiku`/`inherit` |
| `permissionMode` | 否 | `default`/`acceptEdits`/`dontAsk`/`bypassPermissions`/`plan` |
| `maxTurns` | 否 | 最大 agentic 回合数 |
| `skills` | 否 | 注入到 Subagent 上下文的 Skills |
| `memory` | 否 | 持久记忆范围：`user`/`project`/`local` |
| `background` | 否 | `true` = 后台运行 |
| `isolation` | 否 | `worktree` = 在 git worktree 中隔离运行 |
| `hooks` | 否 | 生命周期钩子 |
| `mcpServers` | 否 | 可用的 MCP 服务器 |

### 前台 vs 后台

- **前台**：阻塞主对话，权限弹窗传递给用户
- **后台**：并发运行，启动前预批准工具权限；`Ctrl+B` 可将运行中的任务后台化
- 禁用后台：`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`

### 持久记忆

```yaml
memory: user  # 跨项目记忆 ~/.claude/agent-memory/<name>/
```

启用后 Subagent 可读写记忆目录，`MEMORY.md` 前 200 行自动注入上下文。

## Subagents vs Agent Teams 对比

| 维度 | Subagents | Agent Teams |
|---|---|---|
| **上下文** | 独立窗口，结果返回调用方 | 完全独立窗口 |
| **通信** | 仅向主 Agent 汇报 | Teammates 直接互发消息 |
| **协调** | 主 Agent 管理所有工作 | 共享任务列表，自协调 |
| **最适合** | 聚焦任务，只需结果 | 复杂工作，需讨论和协作 |
| **Token 成本** | 较低（结果摘要返回） | 较高（每个 Teammate 独立实例） |
| **嵌套** | 不可（Subagent 不能再生成 Subagent） | 不可（Teammate 不能生成自己的团队） |

### 决策指南

**用 Subagents 当：**
- 任务产生大量输出但主对话不需要全部
- 需要限制特定工具/权限
- 工作自包含，可返回摘要
- 延迟敏感（Subagent 启动快）

**用 Agent Teams 当：**
- Teammates 需共享发现、相互挑战
- 工作需并行探索 + 自主协调
- 多个独立组件需同时开发
- 需要 Competing Hypotheses 模式

**分层方法（复杂项目）：** 规划阶段定义角色和边界 → 并行 Agent Teams 执行。

### 混合使用

Agent Teams 内的 Teammates 本身可使用 Subagents（如 Explore）进行代码搜索。两者互补而非互斥。

## 限制 Subagent 生成

```yaml
tools: Agent(worker, researcher), Read, Bash  # 只允许生成 worker 和 researcher
```

阻止特定 Subagent：
```json
{ "permissions": { "deny": ["Agent(Explore)", "Agent(my-agent)"] } }
```

> **注意：** v2.1.63 起 `Task` tool 已重命名为 `Agent`。旧的 `Task(...)` 语法仍可用作别名，但新代码和文档应使用 `Agent(...)` 语法。
