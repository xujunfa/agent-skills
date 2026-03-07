# 故障排除

> **来源:** https://docs.openclaw.ai/gateway/troubleshooting + 网络搜索结果
> **获取时间:** 2026-03-06
> **版本锚点:** v2026.3.2

## 第一线诊断

| 命令 | 用途 |
|---|---|
| `openclaw doctor --fix` | 自动诊断 + 修复大多数配置问题 |
| `openclaw logs --follow` | 跟踪 Gateway 日志（可揭示 90% 的错误） |
| `openclaw config get <key>` | 验证特定配置值 |
| `openclaw gateway status` | 检查 Gateway 是否正在运行 |

## 常见错误及修复

### "Gateway Start Blocked"（Gateway 启动被阻止）
- **原因:** `gateway.mode` 未设置
- **修复:** `openclaw setup` 或 `openclaw config set gateway.mode local`
- 快速绕过: `--allow-unconfigured` 标志

### "disconnected (1008)" / "unauthorized: gateway token missing"
- **原因:** Token 不匹配或缺少 `gateway.auth.token`
- **修复:** 检查 `openclaw config get gateway.auth.token` 是否与客户端匹配
- **注意:** 配置键从 `gateway.token` 更改为 `gateway.auth.token` — 旧键会无声地失败
- v2026.2.19+：如果未设置，会自动生成 Token

### 端口冲突
- 默认端口: `18789`（Gateway WS）、`3001`（有时）
- **修复:** 杀死冲突进程，或 `openclaw config set gateway.port <other>`
- Docker: `gateway.bind` 必须是 `"0.0.0.0"`（不是 `127.0.0.1`）

### Docker 特定问题
- 容器绑定到 `127.0.0.1` = 从主机无法访问
- 每次容器重启 = 新 Token = 使所有配对设备失效
- 修复：在配置或环境变量中持久化 Token，将 bind 设置为 `0.0.0.0`
- 健康检查探针: `/health`、`/healthz`、`/ready`、`/readyz`（v2026.3.1+）

### 设备配对必需
- 每个新的浏览器/设备需要批准（安全特性）
- `openclaw pairing approve <channel> <code>`
- v2026.2.21+：省略时默认为唯一可用的 Channel

### 升级后故障
- 最常见原因：配置漂移或新版本中更严格的默认值
- 旧的 `clawdbot-gateway.service` 可能与新 Gateway 冲突
- **修复:** `openclaw doctor --fix`，然后检查是否存在双重 Gateway 进程
- 品牌化：最近版本中 `bot.molt` launchd 标签已替换为 `ai.openclaw`

### 速率限制 (429)
- 不是认证失败 — 超出提供商配额
- 维度: 请求/分钟、输入 Token/分钟、输出 Token/分钟
- v2026.2.19+：错误消息包含活跃模型名称
- 自动故障转移：轮换认证配置文件，然后模型回退

### 配置后 Channel 不工作
- **第 1 原因:** Gateway 不支持热重载 — 必须 `openclaw gateway restart`
- 验证: `openclaw config get channels.<name>.<key>`

### Node.js 版本问题
- 需要 Node >= 22
- 较老版本：模糊的语法错误，不会提及 Node 版本
- 检查: `node --version`

## 平台特定说明

### Windows
- **使用 WSL2** — 不支持原生 Windows
- 在 Ubuntu WSL 中运行 Gateway
- 对于 WhatsApp/Telegram Channel 使用 Node（不是 Bun）
- 在 WSL 内部优先使用 POSIX 路径

### macOS
- 需要签名的构建版本，以便权限能在重建后持久化
- TCC 权限：通过 `node.invoke` 进行屏幕录制、通知、摄像头
- 屏幕捕获工具的 `needsScreenRecording: true` 标志

## 安全性

### CVE-2026-25253
- Gateway 泄露导致远程命令执行
- **修复:** 更新到 2026 年 1 月 29 日之后的任何版本
- 更新后轮换所有 Token

### 最佳实践
- 切勿将 Gateway 暴露到开放互联网；绑定到本地主机
- 使用 SSH 隧道或 Tailscale 进行远程访问
- 定期轮换 `gateway.auth.token`；使用强随机字符串
- 在 `~/.openclaw/.env` 中存储机密，使用 `chmod 600`
- 运行 `openclaw doctor` 以发现有风险的 DM 策略
- 将第三方 ClawHub 技能视为不可信代码

### DM 安全默认值
- 所有 Channel 上的默认 `dmPolicy="pairing"`
- 未知发件人获得配对代码，消息不被处理
- 开放 DM 需要显式设置: `dmPolicy="open"` + `allowFrom: ["*"]`

## 有用的聊天命令

| 命令 | 用途 |
|---|---|
| `/status` | 会话状态（模型、Token、成本） |
| `/new` 或 `/reset` | 重置会话 |
| `/compact` | 压缩会话上下文 |
| `/think <level>` | off\|minimal\|low\|medium\|high\|xhigh |
| `/verbose on\|off` | 切换详细输出 |
| `/usage off\|tokens\|full` | 每个响应的使用情况页脚 |
| `/restart` | 重启 Gateway（群组中仅限所有者） |
| `/elevated on\|off` | 切换提升的 bash 访问权限 |
