# 最佳实践与故障排除

> **来源:** https://code.claude.com/docs/en/agent-teams + https://addyosmani.com/blog/claude-code-agent-teams/ + https://claudefa.st/blog/guide/agents/agent-teams
> **获取时间:** 2026-03-06
> **版本锚点:** Claude Code 2.1.x (experimental)

## 团队规模

| 规模 | 适用场景 | 说明 |
|---|---|---|
| 1-2 Teammates | 快速并行任务 | 低复杂度、低开销 |
| 3-5 Teammates | 标准开发/评审（**推荐起点**） | 平衡并行与协调 |
| 5-7 Teammates | 大规模研究/Inventory | 协调开销接近收益上限 |
| 7+ Teammates | 极少使用 | 协调开销和消息成本通常超过时间节省 |

**经验法则：** 3 个聚焦的 Teammates 通常优于 5 个分散的。

## Token 成本

- 每个 Teammate 是独立 Claude 实例，Token 消耗线性增长
- 3 Teammate 团队 ≈ 单会话的 3-4x Token，但时间大幅缩短
- 研究/评审/新 Feature 的额外 Token 通常值得
- 常规任务用单会话更划算

**降本策略：**
- Teammates 用 Sonnet，Lead 用 Opus
- 优先 message 而非 broadcast
- 限制团队规模到实际需要
- 优化 CLAUDE.md 减少每 Teammate 的探索成本

## 文件冲突

**核心规则：** 两个 Teammates 编辑同一文件 = 覆盖。

**解决方案：**
- 拆分工作，每 Teammate 拥有不同文件集
- 使用 git worktree 隔离（`isolation: worktree`）
- 设计任务边界时明确文件 ownership

### Git Worktree 集成

```bash
claude --worktree  # 或 -w，在隔离 worktree 中启动
```

- Subagent 支持 `isolation: worktree`，自动在临时 worktree 中工作
- 无变更时自动清理
- 完成后需协调 merge 顺序——让 Lead 统一处理
- `.claude/worktrees/` 加入 `.gitignore`

## CLAUDE.md 优化

**对 Agent Teams 特别重要**——Teammates 自动加载 `CLAUDE.md`，不继承对话历史。

**规则 1：** 描述模块边界和文件 ownership 表，防止冲突
**规则 2：** 保持操作性——技术栈、入口、测试命令、数据库
**规则 3：** 定义验证信号：`npm test`、`npm run lint`、`npm run build`，让 Teammates 自验证

## 给 Teammates 足够上下文

Spawn prompt 中包含任务细节，不要假设 Teammate 知道背景：

```text
Spawn a security reviewer with the prompt: "Review src/auth/ for
vulnerabilities. Focus on token handling, session management, and input
validation. The app uses JWT tokens stored in httpOnly cookies.
Report any issues with severity ratings."
```

## 从研究和评审开始

新用户建议先做不需要写代码的并行任务：
- 评审 PR
- 研究库/方案
- 调查 Bug

这些展示并行探索价值，没有并行实现的协调挑战。

## 等待 Teammates 完成

Lead 有时会自己开始实现而非等待。解决方案：
```text
Wait for your teammates to complete their tasks before proceeding
```
或使用 **Delegate Mode**（`Shift+Tab`）。

## 监控与调整

- 定期检查 Teammates 进度
- 重定向不奏效的方法
- 综合到达的发现
- 不要让团队长时间无人监控——浪费风险增加

## 故障排除

### Teammates 不出现

- In-process 模式：按 `Shift+Down` 检查是否已在运行
- 任务可能不够复杂，Claude 判断不需要团队
- Split pane 模式：确认 `which tmux` 可用
- iTerm2：确认 `it2` CLI 已安装，Python API 已启用

### 权限弹窗过多

Teammate 权限请求上浮到 Lead。
**修复：** 启动前在权限设置中预批准常见操作。

### Teammates 遇错停止

检查输出（`Shift+Down` 或点击 pane），然后：
- 直接给额外指令
- Spawn 替代 Teammate 继续工作

### Lead 提前关闭

Lead 可能在所有任务完成前判断结束。
**修复：** 告诉它继续，或要求等待 Teammates 完成。

### 孤儿 tmux 会话

```bash
tmux ls
tmux kill-session -t <session-name>
```

### 任务状态滞后

Teammates 有时未标记任务完成 → 阻塞依赖任务。
**修复：** 手动验证任务是否实际完成，更新状态或让 Lead nudge Teammate。

## 已知限制

| 限制 | 说明 |
|---|---|
| 无会话恢复 | `/resume`、`/rewind` 不恢复 in-process Teammates。恢复后需 spawn 新 Teammates |
| 每会话一个团队 | 清理当前团队后才能创建新的 |
| 无嵌套团队 | Teammates 不能 spawn 自己的团队 |
| Lead 固定 | 创建团队的会话永远是 Lead，不可转让 |
| Spawn 时权限统一 | 所有 Teammates 继承 Lead 权限模式，spawn 后可单独改 |
| 关闭较慢 | Teammates 完成当前请求后才关闭 |
| Split pane 兼容性 | 不支持 VS Code 终端、Windows Terminal、Ghostty |
| Delegate Mode 权限 Bug | GitHub #24307: Delegate Mode 下 spawn 的 Teammates 可能缺失文件操作工具（即使设了 `bypassPermissions`）。规避：spawn 后手动确认 Teammate 工具集 |

## 已知风险

### Zombie Agent（僵尸 Agent）

**GitHub #27610:** 使用 `model: inherit` 的 Teammate 在父会话结束后可能成为僵尸——无法处理 API 调用，TeamDelete 也因 "active member(s) still present" 拒绝执行。

**恢复方式：**
```bash
rm -rf ~/.claude/teams/<team-name>   # 强制清理团队数据
```

**预防：** 正常流程中务必先关闭所有 Teammates 再关 Lead。

### 会话意外中断

**GitHub #29567:** Lead 会话意外结束时无协调关闭机制，Teammates 工作可能处于半完成状态。

**特别注意 worktree 数据丢失风险：** `isolation: worktree` 的 Agent 在异常退出时，worktree 可能被自动清理，未推送的提交将丢失。

**缓解：**
- 重要工作让 Agent 频繁 commit + push 到远程分支
- 意外中断后手动检查 `~/.claude/worktrees/` 和 `.claude/worktrees/` 是否有残留
- 对关键任务，考虑 Teammate 定期向 Lead 汇报进展（通过 TeammateIdle hook 实现）

### 非第一方 API 平台

Bedrock、Vertex、Foundry 用户：Agent Teams 在这些平台上曾有多个 bug（环境变量未传播、模型标识符错误等），已在 v2.1.41+ 陆续修复。**建议升级到最新版本。**

## Headless Mode 集成

```bash
claude -p "your task" --agent  # 无交互模式
```

- 可集成到 CI/CD、pre-commit hooks、数据处理脚本
- 共享任务列表：`CLAUDE_CODE_TASK_LIST_ID` 环境变量让多实例指向同一任务列表
- 建议：先交互验证工作流，再自动化
