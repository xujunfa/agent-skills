---
name: openclaw-expert
description: "Use when working with OpenClaw — covers gateway setup, channels, skills/ClawHub, tools, troubleshooting, configuration, practical use cases, automation (cron/heartbeat/webhooks), multi-agent patterns, overnight coding, and Docker deployment hardening."
trigger_keywords:
  - openclaw
  - openclaw gateway
  - openclaw onboard
  - openclaw config
  - openclaw doctor
  - clawhub
  - openclaw skills
  - openclaw.json
  - openclaw channels
  - gateway.auth.token
  - openclaw troubleshooting
  - Pi agent
  - openclaw cron
  - openclaw heartbeat
  - HEARTBEAT.md
  - openclaw webhook
  - openclaw automation
  - openclaw morning briefing
  - openclaw email
  - openclaw overnight coding
  - openclaw multi-agent
  - openclaw docker
  - openclaw hardening
  - openclaw security
  - openclaw use cases
  - openclaw recipes
  - coding-agent
  - sessions_spawn
  - acpx
  - openclaw deployment
  - workspaceOnly
---

# OpenClaw 专家知识库

## Claude 使用说明
当用户询问 OpenClaw 相关问题时，按需加载下方对应的参考文件。**不要一次性加载所有文件**。

## 工具概览
- **性质：** 框架 / 自托管 AI 代理平台
- **当前版本锚点：** v2026.3.2
- **官方文档：** https://docs.openclaw.ai
- **代码仓库：** https://github.com/openclaw/openclaw
- **许可证：** MIT
- **运行时：** Node >= 22 (TypeScript)

## 按需加载指南

| 场景 / 问题 | 加载文件 |
|---|---|
| Gateway 设置、配置格式、身份认证、提供商、守护进程、Tailscale | `references/gateway-and-config.md` |
| 频道设置（WhatsApp/Telegram/Slack/Discord 等）、DM 策略、配对、群组、应用、节点、语音 | `references/channels-and-platforms.md` |
| Skills（内置/托管/工作区）、ClawHub、内置工具、浏览器、Canvas、Cron、会话 | `references/skills-and-tools.md` |
| 错误、诊断命令、常见修复、升级后问题 | `references/troubleshooting.md` |
| 实际用例、配方、晨间简报、邮件、通宵编码、内容自动化、多代理团队 | `references/use-cases-and-recipes.md` |
| Cron 任务、心跳、HEARTBEAT.md、Webhook、调度、重试、交付模式 | `references/automation-patterns.md` |
| Docker 部署、安全加固、网络隔离、CVE、生产最佳实践 | `references/deployment-and-hardening.md` |

## 关键陷阱
- Gateway **不支持热重载**频道设置——修改配置后必须执行 `openclaw gateway restart` 命令
- 配置字段从 `gateway.token` 更改为 `gateway.auth.token`（新版本）——旧字段会静默失败
- 非本地绑定需要 Token；Gateway 启动时没有 Token 会拒绝启动
- 将 ClawHub 第三方 Skills 视为不可信代码（ClawHavoc 事件：发现 800+ 个恶意 Skill）
- Windows 系统：使用 WSL2；不支持原生 Windows
- Auth 配置文件固定绑定到单个会话，不会按请求轮换——在执行 `/new` 或 `/reset` 时重置

## 维护信息
- **版本：** 1.1.0
- **创建时间：** 2026-03-06
- **最后更新：** 2026-03-06
- **工具版本锚点：** v2026.3.2
