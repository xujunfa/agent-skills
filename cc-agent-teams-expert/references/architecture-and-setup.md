# 架构与设置

> **来源:** https://code.claude.com/docs/en/agent-teams + https://addyosmani.com/blog/claude-code-agent-teams/
> **获取时间:** 2026-03-06
> **版本锚点:** Claude Code 2.1.x (experimental)

## 核心架构

Agent Teams = 多个独立 Claude Code 实例协同工作，共享任务列表和消息系统。

| 组件 | 职责 |
|---|---|
| **Team Lead** | 主会话，创建团队、分配任务、综合结果 |
| **Teammates** | 独立 Claude Code 实例，各有独立上下文窗口 |
| **Task List** | 共享任务队列，支持状态追踪和依赖关系 |
| **Mailbox** | Agent 间直接通信系统（非仅向 Lead 汇报） |

**核心洞察:** LLM 随上下文膨胀性能下降。每个 Agent 窄范围 + 干净上下文 = 更好推理。

### 本地存储

```
~/.claude/teams/{team-name}/config.json    # 团队元数据 + 成员列表
~/.claude/tasks/{team-name}/               # 任务列表
```

`config.json` 包含 `members` 数组（name、agent ID、agent type），Teammates 可读取此文件发现彼此。

## 启用方式

**环境变量（推荐）：**
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

**settings.json 持久化：**
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## 创建团队

用自然语言描述任务和团队结构：

```text
Create an agent team to review our auth system. Spawn three teammates:
- Security reviewer: audit for vulnerabilities
- Performance analyst: profile response times
- Test coverage checker: verify edge cases
```

Claude 自动创建团队、生成任务、分配 Teammates。

### 指定模型

```text
Create a team with 4 teammates. Use Sonnet for each teammate.
```

可混合模型：Opus 做 Lead 策略，Sonnet 做 Teammate 执行。

## 显示模式

| 模式 | 行为 | 要求 |
|---|---|---|
| `in-process`（默认） | 所有 Teammates 在主终端内 | 任何终端 |
| `tmux` / `iTerm2` | 每个 Teammate 独立 pane | tmux 或 iTerm2 + `it2` CLI |
| `auto` | 若已在 tmux 中则 split，否则 in-process | — |

**配置：**
```json
{ "teammateMode": "in-process" }
```

**单次覆盖：** `claude --teammate-mode in-process`

### 键盘操作（in-process 模式）

| 快捷键 | 功能 |
|---|---|
| `Shift+Down` | 切换到下一个 Teammate |
| `Enter` | 查看 Teammate 会话 |
| `Escape` | 中断当前操作 |
| `Ctrl+T` | 切换任务列表 |
| `Shift+Tab` | 切换 Delegate Mode（Lead 仅协调不写代码） |
| `Ctrl+C` ×2（3秒内） | 终止所有后台 Agent |
| `Ctrl+F` ×2 | 终止后台 Agent（确认式） |

**Split pane 不支持的终端：** VS Code 集成终端、Windows Terminal、Ghostty。

## 权限

- Teammates 继承 Lead 的权限设置
- `--dangerously-skip-permissions` 会传递给所有 Teammates
- Spawn 后可单独更改，但 spawn 时无法为个别 Teammate 设定
- 建议：团队启动前在权限设置中预批准常见操作，减少权限弹窗

## 上下文加载

Teammates 自动加载：
- `CLAUDE.md`（项目上下文）
- MCP servers
- Skills

**不继承** Lead 的对话历史。需在 spawn prompt 中包含任务细节：
```text
Spawn a security reviewer with the prompt: "Review src/auth/ for
vulnerabilities. Focus on token handling and session management.
The app uses JWT in httpOnly cookies."
```

## 团队关闭

1. 先关闭所有 Teammates：`Ask the researcher teammate to shut down`
2. 再让 Lead 清理：`Clean up the team`

**关键：** 必须由 Lead 执行清理。Teammate 清理可能导致资源不一致。
