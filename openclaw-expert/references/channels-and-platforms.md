# 渠道和平台

> **来源：** https://github.com/openclaw/openclaw (README) + 网络搜索结果
> **获取时间：** 2026-03-06
> **版本标记：** v2026.3.2

## 支持的渠道

WhatsApp (Baileys)、Telegram (grammY)、Slack (Bolt)、Discord (discord.js)、
Google Chat (Chat API)、Signal (signal-cli)、BlueBubbles (iMessage，推荐)、
iMessage (legacy imsg)、IRC、Microsoft Teams、Matrix、Feishu、LINE、Mattermost、
Nextcloud Talk、Nostr、Synology Chat、Tlon、Twitch、Zalo、Zalo Personal、WebChat。

## 渠道配置

- 通过 `openclaw config set channels.<name>.<key> <value>` 设置
- 网关**不会**热重载渠道设置 — 必须执行 `openclaw gateway restart`
- 交互式添加渠道：`openclaw channels add`（插件板载钩子）
- 每个渠道都有自己的配置命名空间：`channels.telegram.*`、`channels.discord.*` 等

## DM 策略和配对

| 策略 | 行为 |
|---|---|
| `pairing`（默认） | 未知发送者获得配对码；消息不处理 |
| `open` | 任何人都可以 DM（需要 `allowFrom: ["*"]`） |
| `block` | 完全阻止 DM |

- 配置键：`dmPolicy` 或 `channels.<name>.dmPolicy`
- 批准配对：`openclaw pairing approve <channel> <code>`
- 列出待处理：`openclaw pairing list`
- 自 v2026.2.21 起：命令在省略时默认使用唯一可用渠道

## 群组路由

- 激活模式：`mention`（默认）或 `always`
- 切换：在群组聊天中输入 `/activation mention|always`
- 每渠道的分块和路由规则
- 用于线程上下文的回复标签
- 群组命令仅限所有者使用

## 会话模型

- `main` 会话用于直接聊天；群组会话按群组隔离
- 消息排序的队列模式
- 跨渠道的回复支持
- 聊天命令：`/status`、`/new`、`/reset`、`/compact`、`/think <level>`、`/verbose`、`/usage`、`/restart`

## 伴侣应用

### macOS 应用（可选）
- 菜单栏控制平面和健康监控
- Voice Wake + 按键通话覆盖层
- WebChat + 调试工具
- 通过 SSH 远程网关控制
- 需要签名的 macOS 版本以获得权限

### iOS Node（可选）
- Canvas、Voice Wake、Talk Mode
- 摄像头、屏幕录制
- Bonjour + 通过网关 WS 的设备配对

### Android Node（可选）
- 聊天会话、语音标签页、Canvas
- 摄像头/屏幕录制
- 设备命令：通知、位置、SMS、照片、联系人、日历、运动
- 操作：`camera.list`、`device.permissions`、`device.health`、`notifications.actions`

### macOS Node 模式
- `system.run` — 执行本地命令 (stdout/stderr/exit)
- `system.notify` — 发送用户通知
- 通过 `node.invoke` 暴露 Canvas/摄像头
- `needsScreenRecording: true` 用于屏幕录制权限

## 语音

- **Voice Wake**：macOS/iOS 上的唤醒词（常开监听）
- **Talk Mode**：Android 上的连续语音
- TTS：ElevenLabs 主要 + 系统 TTS 后备
- 语音思考级别：`/think <level>`（off|minimal|low|medium|high|xhigh）

## 媒体管道

- 图像/音频/视频与转录钩子
- 大小限制和临时文件生命周期管理
- Telegram：保留原始 `file_name` 用于下载 (v2026.3.2+)

## 提升的 Bash 访问

- `/elevated on|off` — 切换每个会话的提升访问
- 需要启用和允许列表
- 通过 `sessions.patch` 保存，与 `thinkingLevel`、`model` 等并行

## Agent 间通信（会话工具）

| 工具 | 用途 |
|---|---|
| `sessions_list` | 发现活跃会话 + 元数据 |
| `sessions_history` | 获取会话的转录日志 |
| `sessions_send` | 向另一个会话发送消息（可选回复） |
| `sessions_spawn` | 生成带文件附件的子代理 (v2026.3.2+) |

## 心跳

- `agents.defaults.heartbeat.directPolicy`：`allow` 或 `block`
- 每个代理：`agents.list[].heartbeat.directPolicy`
- 默认值在 v2026.3.2 中更改为 `allow`（破坏性变更）
