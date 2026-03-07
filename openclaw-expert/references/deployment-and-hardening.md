<!-- 更新时间: 2026-03-06，来自网络搜索（多个安全指南） -->
# 部署与加固

> Docker、网络隔离和生产环保安全最佳实践。

## RAK 威胁框架

| 向量 | 风险 | 缓解措施 |
|---|---|---|
| **Root Risk** | 通过 shell 访问主机被攻击 | Docker 非 root 用户、cap-drop、只读文件系统 |
| **Agency Risk** | 意外的破坏性操作 | Exec 审批流程、`workspaceOnly: true` |
| **Keys Risk** | 凭证盗取/泄露 | Secret Manager、凭证轮换、出站代理 |

## 八层安全防线（全部必需）

1. **Runtime 版本** — 固定到 v2026.1.29+（CVE-2026-25253 修复）
2. **Gateway 身份验证** — `gateway.auth.token` 对非 loopback 强制要求
3. **DM 策略与白名单** — `dmPolicy: "pairing"`（默认值）；显式 `allowFrom`
4. **文件系统沙箱** — `workspaceOnly: true` 将 agent 限制在工作区目录
5. **Docker 加固** — 非 root 用户、只读、cap-drop（见下文）
6. **Exec 审批流程** — 不可逆操作需人工审查
7. **SSRF 防护** — 出站代理带域名白名单
8. **Plugin 信任** — 启用前审计所有 ClawHub skills

## Docker 加固

### 最小化安全运行命令

```bash
docker run -d \
  --name openclaw \
  --user openclaw \
  --read-only \
  --tmpfs /tmp:noexec,nosuid,size=64M \
  --cap-drop=ALL \
  --security-opt=no-new-privileges \
  -p 127.0.0.1:18789:18789 \
  --cpus="1.0" \
  --memory="2g" \
  -v openclaw-data:/home/openclaw/.openclaw \
  openclaw/openclaw:latest
```

关键参数：
- `--user openclaw` — 绝不使用 root
- `--read-only` + `--tmpfs /tmp` — 不可变文件系统
- `--cap-drop=ALL` — 零 Linux capabilities
- `--security-opt=no-new-privileges` — 防止权限提升
- `-p 127.0.0.1:18789:18789` — 仅绑定到本地主机
- 资源限制：`--cpus`、`--memory`

### 容器注意事项

- 容器内 `gateway.bind` 必须设为 `"0.0.0.0"`（Docker 网络）
- 每次容器重启 = 新 token（配对设备失效）。在配置文件中持久化 token
- 将 Node 镜像固定到特定 digest，不要用 `node:22` 浮动标签
- 健康检查：`/health`、`/healthz`、`/ready`、`/readyz`（v2026.3.1+）

## 网络隔离

### 内部 Docker 网络

```yaml
networks:
  openclaw-internal:
    internal: true  # 无直接互联网路由
```

所有 agent 出站流量通过 Squid 代理，使用域名白名单。即使 agent 被完全破坏，也无法渗漏到未知服务器。

### Tailscale（推荐用于远程访问）

- `gateway.tailscale.mode: "serve"` — 仅 tailnet HTTPS
- `gateway.tailscale.mode: "funnel"` — 公共 HTTPS（需要密码认证）
- 消除所有公共端口暴露
- 启用 Serve/Funnel 时 `gateway.bind` 必须保持 `loopback`

### 禁止操作

- 直接将 gateway 暴露到公网
- 使用 `auth: none`（已在 v2026.1.29 中移除）
- 以 root 身份运行
- 在无反向代理 + TLS 的情况下绑定到 `0.0.0.0`

## 凭证管理

- 将 secrets 存储在 `~/.openclaw/.env`，权限 `chmod 600`
- 通过 `${ANTHROPIC_API_KEY}` 语法在配置中引用
- 每月轮换 API 密钥
- `openclaw secrets audit` 检查凭证卫生
- 发送邮件：使用 Resend API 而不是完整的 Gmail OAuth

## CVE 追踪

| CVE | 影响 | 修复版本 |
|---|---|---|
| CVE-2026-25253 | Gateway RCE via WebSocket | v2026.1.29+ |
| GHSA-76m6-pj3w-v7mf | Gateway 锁中的 SHA-1 弱点 | v2026.2.21+ |
| Node.js CVEs | Runtime 漏洞 | Node >= v22.12.0 |

## 规模警告

2026 年 2 月发现 42,665 个 OpenClaw 实例被公开暴露。93.4% 存在认证绕过。其中 8 个完全开放，具有完整的 shell 访问权限。不要成为其中之一。

## 监控

```bash
openclaw logs --follow                     # 所有日志
openclaw logs --filter heartbeat --follow  # 心跳活动
openclaw gateway status                     # Gateway 健康状态
openclaw doctor --fix                      # 自动诊断
```

启用全面的会话和操作日志记录以进行事件检测。
