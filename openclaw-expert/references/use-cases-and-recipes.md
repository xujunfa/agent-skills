<!-- Updated: 2026-03-06 from web search (multiple community sources) -->
# 用例和方案

> 实用、可操作的 OpenClaw 模式 — 人们每天实际运行的功能。

## 高价值用例（按采用度排序）

### 1. 早间简报（最容易获得高价值）

**模式：** Cron（隔离会话）在 8AM -> 聚合 Calendar + Email + News -> 通过 WhatsApp/Telegram 投递。

```bash
openclaw cron add \
  --name "Morning Briefing" \
  --cron "0 8 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "Compile morning briefing: calendar events, top 5 unread emails, weather, top HN stories. Format as concise bullet list." \
  --announce \
  --channel telegram
```

隔离会话将简报上下文保持在主聊天之外。在 Haiku 上成本约 $0.05-0.30/天；在 Opus 上 $5-10/天。

### 2. 邮件管理和分类

**模式：** Heartbeat（每 30 分钟）或 Cron -> Gmail 检查 -> 优先级排序 + 摘要 -> 通过消息通道通知。

- 使用 `HEARTBEAT.md` 进行持续监控和上下文跟踪
- 使用 Cron 隔离会话生成干净的每日摘要
- 发送时：使用 Resend API 而不是完整的 Gmail OAuth，以避免暴露整个收件箱
- VIP 邮件标记 + 其余邮件的批量摘要

### 3. 夜间编码（开发者最爱）

**模式：** 晚间通过聊天分配任务 -> agent 生成编码子 agent -> 编写代码、运行测试、修复错误 -> 早上创建 PR。

主要方法：
- **coding-agent skill**（内置）：通过后台进程委托给 Codex/Claude Code
- **ACP sessions**：`runtime: "acp"` 从 agent 回合生成 Claude Code 会话
- **容器化 Claude Code**：`openclaw-plugin-claude-code` 在 rootless Podman 容器中运行
- **sessions_spawn**：使用文件附件生成子 agent（v2026.3.2+）

获胜模式：启动 CLI -> 让它在工作目录中编辑 -> 流式传输日志返回 -> 如果询问则回答提示。

### 4. 内容自动化（采用度最广 ~35%）

**模式：** 创建一个内容 -> agent 在各平台重新利用、创建图像、发布、跟踪性能。

- 使用记忆进行新闻通讯写作（避免重复主题）
- 社交媒体交叉发布，采用平台特定的格式
- 通过 cron 进行内容日历管理
- 性能跟踪 + "下一步写什么"的建议

### 5. 日历和日程安排

**模式：** Agent 通过邮件/聊天处理日程安排的来回协商。

- 查找跨参与者的最佳会议时间
- 与外部方（经销商、客户、服务）协调
- 检测到冲突时自动提议替代方案

### 6. DevOps 和依赖管理

**模式：** 每周 cron -> 扫描依赖文件 -> 检查注册表 + 漏洞数据库 -> 用优先级建议通知。

```bash
openclaw cron add \
  --name "Dependency Audit" \
  --cron "0 9 * * 1" \
  --session isolated \
  --message "Scan package.json/requirements.txt for outdated deps and known CVEs. Prioritize by severity."
```

### 7. 研究和数据聚合

**模式：** 监控来源 -> 去重 -> 策划每日摘要。

- 来自 100+ 来源的 AI 新闻聚合
- 竞争对手监控
- 市场/股票分析
- 学术论文跟踪

### 8. 自写 Skills（OpenClaw 独特功能）

当 agent 无法提供请求时，它自主地编写新的 SKILL.md，热重载配置。社区报告的示例：图像生成、Todoist 集成、HomePod 控制。

相关 skills：`advanced-skill-creator`、`agent-evolver`、`agent-reflect`、`adaptive-learning-agents`。

### 9. CRM 和销售自动化

**模式：** 转录销售通话 -> 自动记录笔记/后续步骤/跟进到 Salesforce/HubSpot。

节省每次通话 15-20 分钟。与电子邮件自动化结合进行后续序列。

### 10. 多 Agent 团队

**模式：** 主 agent + 专家 agents，每个都有隔离的工作区/会话。

设置：`agents.list[]` 具有每个 agent 的 `agentDir`、`AGENTS.md`、`SOUL.md`。

最佳实践：
- 从 2 个 agents 开始（1 个主 agent + 1 个专家），而不是 4 个
- 绝不在 agents 间重复使用 `agentDir`（会导致会话冲突）
- 对 agent 间通信使用 webhook/API 触发模式或共享通道
- 对于复杂管道使用协调器/编排器模式

## 通用方案模式

每个自动化都遵循：**Trigger** -> **Action** -> **Output**

| Trigger | Action | Output |
|---|---|---|
| Cron（基于时间） | Fetch/check/compute | Send to channel |
| Webhook（基于事件） | Extract/analyze | Save to file |
| Heartbeat（上下文感知） | Reason about relevance | Create PR / log |

## 成本管理

| Model | Heartbeat cost（30 分钟间隔） | 每日估计 |
|---|---|---|
| Haiku | 几分之一的分币 | $0.50-2 |
| Sonnet | ~$0.02-0.05 | $2-8 |
| Opus | ~$0.10-0.50 | $5-30 |

使用安静时间避免不必要的唤醒。在 heartbeat 配置中设置 `activeHours`。
