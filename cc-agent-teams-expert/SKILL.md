---
name: cc-agent-teams-expert
description: "Use when working with Claude Code Agent Teams or multi-agent orchestration — covers architecture, subagents, TeammateTool, task system, hooks, use cases, worktree isolation, cost optimization, and troubleshooting."
trigger_keywords:
  - agent teams
  - agent team
  - TeammateTool
  - subagents
  - multi-agent
  - claude code swarm
  - teammates
  - CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS
  - TeammateIdle
  - TaskCompleted
  - competing hypotheses
  - parallel agents
  - orchestration
  - 编排设计
  - agent design
  - role design
  - 角色设计
  - 适用场景
---

# Claude Code Agent Teams 专家知识库

## Claude 使用说明
当用户询问 Claude Code Agent Teams 相关问题时，按需加载下方对应的参考文件。**不要一次性加载所有文件**。

## 工具概览
- **性质:** Claude Code CLI 实验性特性（多 Agent 协同）
- **状态:** Experimental，需 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 启用
- **官方文档:** https://code.claude.com/docs/en/agent-teams
- **核心架构:** Lead + Teammates + 共享任务列表 + Mailbox 通信

## 按需加载指南

| 场景 / 问题 | 加载文件 |
|---|---|
| 架构、启用、配置、显示模式、创建团队、权限、上下文加载 | `references/architecture-and-setup.md` |
| Subagents 对比、内置 Agent、自定义 Subagent、Frontmatter、决策指南 | `references/subagents-and-comparison.md` |
| 任务系统、Mailbox 通信、Hooks（TeammateIdle/TaskCompleted）、Plan Approval、Delegate Mode | `references/task-system-and-communication.md` |
| 具体用例（8 种）、C 编译器案例、企业采用 | `references/use-cases-and-patterns.md` |
| 团队规模、Token 成本、文件冲突、Git Worktree、CLAUDE.md 优化、故障排除、限制 | `references/best-practices-and-troubleshooting.md` |
| **适用性判断、编排设计、角色模板、通信拓扑、Prompt 模板、反模式** | `references/orchestration-design-guide.md` |

## 关键陷阱
- Agent Teams 是**实验性**特性，默认禁用——必须显式设置环境变量
- 两个 Teammates 编辑同一文件 = 覆盖（用 worktree 或文件 ownership 规避）
- Teammates 不继承 Lead 对话历史——必须在 spawn prompt 中给足上下文
- `/resume` 和 `/rewind` 不恢复 in-process Teammates——需重新 spawn
- Lead 可能自行实现而非委派——用 Delegate Mode 或口头指示等待
- broadcast 消息成本随团队规模线性增长——优先用定向 message
- `model: inherit` 的 Teammate 在父会话异常结束后可能成为僵尸（需 `rm -rf ~/.claude/teams/<name>` 恢复）
- 会话意外中断时 worktree Agent 的未推送提交可能丢失——关键工作应频繁 commit+push
- v2.1.63 起 `Task` tool 改名为 `Agent`——旧 `Task(...)` 语法仍可用作别名

## 维护信息
- **版本:** 1.1.0
- **创建时间:** 2026-03-06
- **最后更新:** 2026-03-06
- **工具版本锚点:** Claude Code 2.1.x (experimental)
