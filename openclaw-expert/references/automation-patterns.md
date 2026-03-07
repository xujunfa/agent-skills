<!-- Updated: 2026-03-06 from web search (docs.openclaw.ai + community sources) -->
# 自动化模式

> Cron、Heartbeat 和 Webhooks —— 三大调度原语。

## Cron vs Heartbeat：决策指南

| 功能 | Cron | Heartbeat |
|---|---|---|
| 执行方式 | 隔离会话（独立上下文） | 主会话（共享上下文） |
| 触发方式 | 精确时间表（crontab 语法） | 定期间隔（默认 30 分钟） |
| 智能性 | 按时间盲目执行 | 判断是否需要执行 |
| 上下文 | 每次运行清空状态 | 完整对话历史 |
| 最适合 | 周期性报告、摘要、审计 | 监控、主动告警 |
| 成本 | 可预测（按次计费） | 可变（取决于上下文大小） |

**经验法则：** 对于"在 Y 时刻执行 X"使用 Cron。对于"监控 X"使用 Heartbeat。

## Cron 任务

### 执行方式

| 方式 | 会话 | 行为 |
|---|---|---|
| `main` | 主会话 | 排队系统事件，在下一个 Heartbeat 时运行 |
| `isolated`（默认） | `cron:<jobId>` | 专用代理轮次，拥有独立的上下文/模型/思考 |

### 关键命令

```bash
openclaw cron add --name "Job Name" --cron "0 8 * * *" --tz "TZ" \
  --session isolated --message "prompt" --announce --channel telegram

openclaw cron list              # 列出所有任务
openclaw cron run <jobId>       # 手动触发（测试）
openclaw cron remove <jobId>    # 删除任务
```

### 交付模式

| 模式 | 行为 |
|---|---|
| `announce`（默认） | 将结果发送到指定频道 |
| `webhook` | POST 结果到 URL：`--delivery webhook --delivery-to "https://..."` |
| `none` | 静默执行（结果仅在日志中） |

### 重试行为

- **周期性任务：** 连续出错时的指数退避：30s -> 1m -> 5m -> 15m -> 60m。下一次成功后重置。
- **一次性（`at`）任务：** 对于瞬时错误（速率限制、网络、服务器错误）最多重试 3 次。永久错误立即禁用。

### 持久化

任务保存在 `~/.openclaw/cron/` 下 —— 网关重启后继续存在。

## Heartbeat

### HEARTBEAT.md

放置在 `~/.openclaw/HEARTBEAT.md`（或每个代理工作区）。代理在每次唤醒时读取此文件，推理需要关注的内容。

```markdown
## Email Triage
- Check Gmail for new unread emails
- Flag VIP emails (from: boss@co.com, client@co.com)
- Batch non-urgent into digest

## Project Monitor
- Check if CI pipeline has failures
- Notify me of any PRs waiting for my review
```

### 配置

```bash
openclaw config set agents.defaults.heartbeat.directPolicy allow
# Per-agent: agents.list[].heartbeat.directPolicy
```

Heartbeat 间隔和活跃时段可在配置中调整。使用静默时段在睡眠期间抑制非关键告警。

### 调试 Heartbeat

```bash
openclaw heartbeat --now          # 手动触发
openclaw heartbeat --dry-run      # 干运行（无操作）
openclaw logs --filter heartbeat --follow  # 监控
```

### 成本控制

- 对 Heartbeat 检查使用 Haiku（每次检查仅需几分的费用）
- 设置 `activeHours`（例如 7AM-11PM）以避免夜间 token 消耗
- 保持 HEARTBEAT.md 简洁 —— 越短每次检查成本越低

## Webhooks

### 入站（External -> OpenClaw）

外部服务（GitHub、Stripe、智能家居中心）调用 OpenClaw HTTP 端点来触发代理操作。

```bash
# Per cron job
openclaw cron add --name "GitHub PR Review" \
  --delivery webhook \
  --delivery-to "https://hooks.openclaw.local/pr-review"
```

### 常见 Webhook 来源

| 来源 | 用例 |
|---|---|
| GitHub | PR 打开/合并、问题创建、CI 失败 |
| Stripe | 收到付款、订阅变更 |
| Sentry | 错误告警 -> 自动分类 |
| Smart home | 传感器触发 -> 代理操作 |

### 出站（OpenClaw -> External）

每个任务的 Webhook 发送：`delivery.mode = "webhook"` + `delivery.to = "<url>"`。

## 组合模式

**晨间简报：** Cron（隔离，上午 8 点）+ 发送到 WhatsApp
**持续监控：** Heartbeat + HEARTBEAT.md 检查清单
**事件响应：** 入站 Webhook（GitHub CI 失败）-> 生成编码代理进行调查
**定期审计：** 每周 Cron + 隔离会话 + Webhook 交付到 Slack
