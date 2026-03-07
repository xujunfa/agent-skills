# 任务系统与通信

> **来源:** https://code.claude.com/docs/en/agent-teams + https://claudefa.st/blog/guide/agents/agent-teams-controls
> **获取时间:** 2026-03-06
> **版本锚点:** Claude Code 2.1.x (experimental)

## 任务系统

### 任务状态

`pending` → `in_progress` → `completed`

### 依赖关系

- 任务可声明对其他任务的依赖（DAG）
- 被阻塞的任务（blockedBy 未完成）不可被认领
- 依赖完成后自动解除阻塞，无需手动干预

### 任务认领

- **Lead 分配：** 指定哪个 Teammate 做哪个任务
- **自认领：** Teammate 完成任务后自动拾取下一个可用、未阻塞的任务
- **文件锁：** 防止多个 Teammates 同时认领同一任务的竞态条件

### 任务粒度建议

| 粒度 | 问题 |
|---|---|
| 太小 | 协调开销超过收益 |
| 太大 | 长时间无检查点，浪费风险高 |
| 合适 | 自包含单元，产出明确交付物（函数、测试文件、评审报告） |

**经验法则：** 每 Teammate 5-6 个任务，保持高效且 Lead 可在卡住时重新分配。

### 任务存储

```
~/.claude/tasks/{team-name}/    # 每个任务一个 JSON 文件
```

检查任务状态：
```bash
cat ~/.claude/tasks/{team}/*.json | jq '{id, subject, status, owner, blockedBy}'
```

## Mailbox 通信系统

### 消息类型

| 类型 | 说明 |
|---|---|
| **message** | 发送给特定 Teammate |
| **broadcast** | 发送给所有 Teammates（谨慎使用，成本随团队规模线性增长） |

### 自动投递

- 消息自动投递到收件方——**不需要**手动检查 inbox
- 报告 Teammate 消息时**不需要**引用原文（用户已能看到）

### 空闲（Idle）行为

- Teammate 每轮结束后进入 idle——**完全正常**
- Idle **不代表**完成或不可用，只是等待输入
- Idle Teammates 可接收消息——发送消息会唤醒它们
- Idle 通知是自动的，无需对其做出反应（除非要分配新工作）
- **不要**将 idle 视为错误

### Inbox 手动检查

```bash
cat ~/.claude/teams/{team}/inboxes/team-lead.json | jq '.'
```

## Hooks——质量门禁

### TeammateIdle

**触发时机：** Teammate 即将进入空闲状态

**用途：** 自动分配后续任务，重定向提前完成的 Teammate

**退出码行为：**
| 退出码 | 效果 |
|---|---|
| 0 | 允许正常 idle |
| 2 | 将 stdout 作为反馈发送，保持 Teammate 继续工作 |

**支持停止 Teammate：** `{"continue": false, "stopReason": "..."}` 可停止 Teammate。

### TaskCompleted

**触发时机：** 任务被标记为完成时（完成前拦截）

**用途：** 强制质量门禁——要求测试通过、lint 检查、满足特定验收标准

**退出码行为：**
| 退出码 | 效果 |
|---|---|
| 0 | 允许任务完成 |
| 2 | 阻止完成，将 stdout 作为反馈发送给 Teammate |

**示例——自动测试门禁：** TaskCompleted hook 运行测试套件 → 无论哪个 Teammate 完成的工作，都不会在测试失败的情况下关闭任务。

### 在 settings.json 中配置

```json
{
  "hooks": {
    "TeammateIdle": [{
      "hooks": [{ "type": "command", "command": "./scripts/assign-next-task.sh" }]
    }],
    "TaskCompleted": [{
      "hooks": [{ "type": "command", "command": "./scripts/run-tests.sh" }]
    }]
  }
}
```

## Plan Approval 工作流

1. 要求 Teammate 在实现前先规划：
   ```text
   Spawn an architect teammate to refactor auth.
   Require plan approval before any changes.
   ```
2. Teammate 进入只读 plan mode → 产出计划 → 发送审批请求给 Lead
3. Lead 审核后 approve 或 reject（附反馈）
4. Reject 时 Teammate 留在 plan mode 修改后重新提交
5. Approve 后 Teammate 退出 plan mode 开始实现

**影响 Lead 判断：** 在 prompt 中给标准，如 "only approve plans that include test coverage"。

## Delegate Mode

按 `Shift+Tab` 切换——限制 Lead 只能协调（spawn、message、task），不能直接写代码。

**适用场景：** Lead 自行实现任务而非委派时，用 Delegate Mode 强制其只做协调。

也可口头指示：
```text
Wait for your teammates to complete their tasks before proceeding
```

## Subagent 生命周期 Hooks

| 事件 | Matcher | 说明 |
|---|---|---|
| `SubagentStart` | Agent type name | Subagent 开始执行时 |
| `SubagentStop` | Agent type name | Subagent 完成时 |

```json
{
  "hooks": {
    "SubagentStart": [{
      "matcher": "db-agent",
      "hooks": [{ "type": "command", "command": "./scripts/setup-db.sh" }]
    }]
  }
}
```
