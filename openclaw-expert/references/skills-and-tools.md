# Skills 和工具

> **来源:** https://docs.openclaw.ai/tools/skills + https://github.com/openclaw/clawhub + 网页搜索
> **获取日期:** 2026-03-06
> **版本锚点:** v2026.3.2

## Skills 架构

Skills = Markdown 指令文件，在激活时加载到 Agent 上下文中。
非编译代码——纯文本驱动架构（遵循 AgentSkills 规范）。

### SKILL.md 格式
```yaml
---
name: skill-name
version: 1.0.0
# ... 需求项（环境变量、二进制文件、安装规范）
---
```
后跟 Markdown 指令。解析器仅支持单行 Frontmatter 键。

### Skill 优先级（从高到低）

| 位置 | 优先级 | 范围 |
|---|---|---|
| `<workspace>/skills/` | 最高 | 项目特定 |
| `~/.openclaw/skills/` | 中等 | 用户级（托管/已安装） |
| 捆绑（在 openclaw 包中） | 最低 | 始终可用 |

- 同名 Skill：最高优先级获胜
- 通过放置在工作区或托管目录中来覆盖捆绑 Skill
- v2026.3.2 附带约 53 个捆绑 Skill（邮件、日历、GitHub、浏览器等）

## ClawHub——公共注册表

- **URL:** https://clawhub.com（也支持 https://clawhub.ai）
- **规模:** 13,700+ 社区 Skill（增长迅速）
- 搜索：基于向量/嵌入（不仅仅是关键字）
- 版本控制：Semver 版本、更改日志 + 标签
- 发布：Fork `openclaw/clawhub`、添加 Skill 文件夹、打开 PR
- 要求：GitHub 账户需满 1 周以上

### ClawHub CLI 命令

| 命令 | 用途 |
|---|---|
| `clawhub install <slug>` | 安装 Skill |
| `clawhub uninstall <slug>` | 移除 Skill |
| `clawhub list` | 列出已安装的 Skill |
| `clawhub update --all` | 更新所有 Skill |
| `clawhub inspect <slug>` | 预览而不安装 |

默认安装路径：`./skills`（或已配置的工作区）。覆盖：`--workdir` 或 `CLAWHUB_WORKDIR`。

## 安全警告：ClawHavoc 事件

- 2026 年 1 月至 2 月：ClawHub 上发现 800+ 恶意 Skill
- 攻击向量：仿冒名称、虚假"先决条件"安装步骤
- 载荷：Atomic macOS Stealer (AMOS)、反向 Shell、凭证泄露
- **启用前务必阅读 Skill 代码。将第三方 Skill 视为不受信任。**

## 内置工具

### 浏览器控制
- 专用 openclaw Chrome/Chromium 实例
- CDP (Chrome DevTools Protocol) 控制
- 快照、操作、上传、配置文件管理

### Canvas + A2UI
- Agent 驱动的可视化工作区
- `canvas.push`、`canvas.reset`、`canvas.eval`、`canvas.snapshot`
- A2UI：Agent 到 UI 的实时交互内容框架

### Cron 和自动化
- Cron 任务：计划的 Agent 任务
- Webhooks：HTTP 触发的 Agent 操作
- Gmail Pub/Sub：邮件触发的自动化

### PDF 工具 (v2026.3.2+)
- 原生 Anthropic 和 Google PDF 提供商支持
- 非原生模型的提取回退
- 配置：`agents.defaults.pdfModel`、`pdfMaxBytesMb`、`pdfMaxPages`

### Node 工具（设备本地，通过 `node.invoke`）

| 工具 | 平台 | 用途 |
|---|---|---|
| `system.run` | macOS | 执行本地命令 |
| `system.notify` | macOS | 发送通知 |
| `camera.snap` | iOS/Android/macOS | 拍摄照片 |
| `camera.clip` | iOS/Android | 录制视频片段 |
| `screen.record` | iOS/Android/macOS | 屏幕录制 |
| `location.get` | iOS/Android | 获取设备位置 |
| `device.health` | Android | 电池、存储空间等 |
| `notifications.actions` | Android | 打开/关闭/回复 |

## 出站适配器 (v2026.3.2+)

- 共享 `sendPayload` 支持：Discord、Slack、WhatsApp、Zalo、Zalouser
- 多媒体迭代及块感知文本回退

## ACP (Agent Communication Protocol)

- 线程绑定的 Agent 运行时，用于多 Agent 协调
- `acp` 生成/发送调度、生命周期控制、启动协调
- 在 v2026.3.2+ 中默认启用
