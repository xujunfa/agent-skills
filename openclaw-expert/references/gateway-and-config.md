# Gateway 与配置

> **来源：** https://docs.openclaw.ai + https://github.com/openclaw/openclaw (README)
> **获取时间：** 2026-03-06
> **版本锚点：** v2026.3.2

## 架构

- 单一 **Gateway** 进程 = WebSocket 控制平面，管理会话、通道、工具、事件
- 默认端口：`18789`，绑定：`127.0.0.1`（本地回路）
- 客户端通过 WS 连接：CLI、WebChat UI、macOS 应用、iOS/Android 节点、Pi agent（RPC）
- Gateway 直接提供 Control UI + WebChat — 无需单独的 Web 服务器

```
通道 (WhatsApp/Telegram/...) -> Gateway (ws://127.0.0.1:18789)
                                ├─ Pi agent (RPC)
                                ├─ CLI (openclaw ...)
                                ├─ WebChat UI
                                ├─ macOS 应用
                                └─ iOS / Android 节点
```

## 安装

```bash
npm install -g openclaw@latest   # 或 pnpm add -g openclaw@latest
openclaw onboard --install-daemon  # 向导 + launchd/systemd 守护进程
```

- 运行时：**Node >= 22**（较旧版本会导致隐晦的语法错误）
- 从源代码开发：`pnpm install && pnpm ui:build && pnpm build`
- 开发循环：`pnpm gateway:watch`（TypeScript 更改时自动重新加载）

## 配置文件

- 主要配置：`~/.openclaw/openclaw.json`
- 旧版配置：`~/.clawdbot/moltbot.json`（仍支持）
- 工作区：`~/.openclaw/workspace`（默认）或通过配置自定义
- CLI：`openclaw config set <key> <value>` / `openclaw config get <key>`
- 架构：`openclaw config schema` — 始终在猜测配置键之前检查

## 认证与令牌

### Gateway 认证
- `gateway.auth.token` — 保护 Gateway WS；**非本地回路绑定时必需**
- `gateway.auth.mode` — `"token"`（默认）或 `"password"`
- 自动生成：从 v2026.2.19+ 起，未设置时 Gateway 会自动生成并持久化令牌

### 提供商认证（LLM 密钥）
- `openclaw models auth paste-token --provider anthropic|openai|...`
- 认证配置：`"anthropic:subscription"`（OAuth）+ `"anthropic:api"`（API 密钥）— OAuth 优先级更高
- 故障转移：在速率限制时轮转认证配置，回退到 `agents.defaults.model.fallbacks`
- 配置按会话锁定（不是按请求）；在 `/new`、`/reset` 或压缩时重置
- 单行命令：`openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$KEY"`

### 密钥管理（v2026.2.26+）
- 存储在 `~/.openclaw/.env`（权限为 `chmod 600`）
- 引用：在配置中使用 `${ANTHROPIC_API_KEY}` 语法
- CLI：`openclaw secrets audit|configure|apply|reload`
- SecretRef 在 64 个凭据目标中展开

## 关键 Gateway 配置项

| 配置项 | 默认值 | 目的 |
|---|---|---|
| `gateway.port` | `18789` | WS 监听端口 |
| `gateway.bind` | `loopback` | `loopback` 或 `0.0.0.0` |
| `gateway.mode` | — | `local` 或 `cloud`（必须设置） |
| `gateway.auth.token` | 自动 | Gateway 访问令牌 |
| `gateway.auth.mode` | `token` | `token` 或 `password` |
| `gateway.tailscale.mode` | `off` | `off`、`serve`（tailnet）、`funnel`（公开） |
| `agents.defaults.model` | — | 默认 LLM 模型 |
| `agents.defaults.model.fallbacks` | — | 回退模型链 |
| `agents.defaults.pdfModel` | — | PDF 分析模型 |

## 多智能体路由

- 通过 `agents.list[]` 将通道/账户/对等节点路由到隔离的智能体
- 每个智能体拥有自己的工作区、会话、系统提示
- CLI：`openclaw agents bindings|bind|unbind` 用于账户范围的路由

## Tailscale 集成

- `serve`：通过 `tailscale serve` 提供 tailnet 专用 HTTPS
- `funnel`：公开 HTTPS（要求 `gateway.auth.mode: "password"`）
- 启用 Serve/Funnel 时 `gateway.bind` 必须保持 `loopback`
- 可选：`gateway.tailscale.resetOnExit` 在关闭时撤销更改

## 远程 Gateway（Linux）

- 在 Linux 服务器上运行 Gateway；通过 Tailscale 或 SSH 隧道连接
- Exec 工具在 Gateway 所在位置运行；设备操作在设备节点上运行
- `openclaw gateway --port 18789 --verbose` 手动启动

## 开发通道

| 通道 | 标签 | 描述 |
|---|---|---|
| `stable` | `latest` | 标记版本 `vYYYY.M.D` |
| `beta` | `beta` | 预发布版本 `vYYYY.M.D-beta.N` |
| `dev` | `dev` | `main` 分支的 HEAD |

切换：`openclaw update --channel stable|beta|dev`

## 必要 CLI 命令

| 命令 | 目的 |
|---|---|
| `openclaw onboard` | 设置向导（gateway + workspace + channels + skills） |
| `openclaw gateway start` | 启动守护进程 |
| `openclaw gateway restart` | 重启（配置变更后必需） |
| `openclaw config set <k> <v>` | 设置配置值 |
| `openclaw config schema` | 查看配置架构 |
| `openclaw doctor --fix` | 诊断 + 自动修复问题 |
| `openclaw logs --follow` | 跟踪 Gateway 日志 |
| `openclaw update` | 更新到最新版本 |
| `openclaw models auth paste-token` | 设置提供商 API 密钥 |
| `openclaw secrets audit` | 审计凭据安全性 |
