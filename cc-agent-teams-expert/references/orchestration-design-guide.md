# 编排设计指南

> **来源:** 外部 Agent 研究（2026-03-06），基于 Anthropic 官方博客、Addy Osmani、ClaudeFast、VoltAgent、Medium/Reddit/X 社区实战
> **获取时间:** 2026-03-06
> **版本锚点:** Claude Code 2.1.x (experimental)

## 适用性决策树

```
任务到来
  │
  ├─ 工作者之间需要直接沟通/辩论/挑战假设？
  │   ├─ 是 → Agent Teams（Mesh 通信 + 共享任务列表）
  │   └─ 否 ↓
  │
  ├─ 需要并行执行独立子任务？
  │   ├─ 是（结果汇总即可，无需互聊）→ Subagents（Hub-and-Spoke，低 token）
  │   └─ 否（顺序/单文件/紧耦合）→ 单 Agent
  │
  └─ 复杂度过滤（附加）
      ├─ < 5-6 个独立子任务 → 单 Agent / Subagents
      ├─ 跨层/Competing Hypotheses/多维审查 → Agent Teams
      └─ 团队 >5 或 无明确文件边界 → 拆成阶段性小团队
```

**核心判断句：** "他们需要互相发消息吗？" 是 → Teams，否 → Subagents。

### ROI 拐点

| 指标 | 数据 |
|------|------|
| 甜点规模 | 3-5 Teammates + 每个 5-6 任务 |
| Token 成本 | 约单 Agent 的 3-4× |
| 时间节省 | 复杂任务 5-10× |
| 覆盖协调成本前提 | 必须有明确可并行的独立作用域 + 沟通价值 |

## 反模式（3 个真实失败案例）

### 反模式 1：小/顺序任务滥用 Teams

4 人 Teams 做简单 ERP 模块 → Lead 忘协调 → token 耗尽只完成一半。
**教训：** 单 Agent 5 分钟搞定的事不要 Teams。

### 反模式 2：无文件所有权导致冲突

两个 Teammate 同时编辑同一文件互相覆盖。C 编译器早期也因"多 Agent 抢同一 bug"而冲突。
**教训：** spawn prompt 必须声明目录边界。

### 反模式 3：Lead 不委托 + 过大团队

Lead 自己写代码 + 团队 >5 → 死循环；Teammates 闲置。
**教训：** 必须开 Delegate Mode（`Shift+Tab`）或在 prompt 中明确 "never implement code yourself"。

## 角色设计

### 常见角色模板

| 角色 | 模式 | 典型工具 |
|------|------|----------|
| Builder / Implementer | 默认 | Read, Write, Edit, Bash |
| Reviewer / Validator | Plan mode | Read, Grep, Glob |
| Researcher / Explorer | 只读 | Read, Grep, Glob, WebSearch |
| Critic / Devil's Advocate | 默认 | 全部（用于反驳） |
| Specialist（安全/性能/测试） | 按需 | 按领域限制 |

### C 编译器 16 Agent 分工（Anthropic 官方）

- **多数 General Agents：** 核心问题解决者——认领任务、锁文件、持续迭代
- **5 个 Specialized Agents：**
  1. 去重 Agent（coalescing duplicate code）
  2. 性能优化 Agent（improve compiler performance）
  3. 代码生成优化 Agent（output efficient compiled code）
  4. Rust 设计批评 Agent（critique + structural changes）
  5. 文档 Agent（documentation）
- **协调方式：** 无单一 Lead，所有 Agent 通过 `current_tasks/` 文件锁 + Git merge 自协调

## 通信拓扑

| 拓扑 | 优点 | 缺点 | 适用 |
|------|------|------|------|
| Star（全过 Lead） | 控制强 | Lead 瓶颈 | 简单审查、汇总式任务 |
| Mesh（直接通信） | 真实协作、自协调 | 消息成本高 | 复杂开发、Competing Hypotheses |
| Pipeline（链式） | 顺序清晰 | 不能真正并行 | 研究→实现→审查的阶段式工作 |

**推荐：** 复杂任务用 Mesh + 文件锁；简单用 Star + Delegate Mode。

## 编排模式（含超越官方的模式）

### 官方模式（4 种）

1. **Wave Execution** — Lead 分发一批任务 → 并行 → 综合
2. **Progressive Refinement** — 共享中间发现 → 迭代优化 → 逐步收敛
3. **Specialization** — 每 Teammate 拥有特定领域，不交叉
4. **Builder-Validator Chain** — Builder 实现 → Validator 审查 → 反馈循环

### 社区模式（2 种新增）

#### Red Team / Blue Team（对抗式）

**场景：** 调试、设计争议、根因分析

```text
Create an agent team to debug why the app exits after one message.
Spawn 5 teammates, each investigating a different hypothesis.
Have them talk to each other to disprove each other's theories
like a scientific debate.
End with: (1) consensus root cause, (2) reproducer, (3) fix plan.
```

**为什么有效：** 消除锚定偏差——多独立调查者主动反驳彼此，存活的理论更可能是真正根因。

#### Orchestrator-Only + Self-Claim（类 MapReduce）

**场景：** 大规模任务队列、后端处理流水线

```text
Create an agent team for <goal>. I want the lead to focus on
orchestration only (never touch code). Break work into 5-6 tasks
per teammate with clear deliverables and dependencies.
Have teammates self-claim unblocked tasks; lead synthesizes
progress and resolves blockers.
```

可搭配 TaskCompleted hook 强制质量门控。

## Prompt 模板

### Lead Prompt 模板

```text
Create an agent team to <明确目标>.
Spawn <N> teammates:
- Teammate 1: <角色 + 文件边界 + 交付物>
- Teammate 2: <角色 + 文件边界 + 交付物>
- ...

Have them coordinate through the shared task list and mailbox.
I want YOU (lead) to focus ONLY on orchestration: break tasks,
resolve blockers, synthesize results. Never implement code yourself.
Wait for your teammates to complete their tasks before proceeding.
Use Delegate Mode.
```

**关键点：** 必须包含 "Never implement code yourself" + "Wait for teammates"，否则 Lead 会自己干活。

### Teammate Spawn Prompt 要素清单

每个 Teammate 的 spawn prompt **必须包含**：

1. **角色 + 作用域** — "Security reviewer for src/auth/"
2. **文件边界** — "只编辑 src/api/users/"
3. **聚焦点 & 交付物** — "Focus on JWT... Deliver 10-bullet summary + severity ratings"
4. **成功标准** — "tests must pass, lint clean"
5. **协调指令** — "use shared task list, message others if blocker"
6. **上下文补充** — "The app uses httpOnly cookies..."

### CLAUDE.md 团队优化模板

```markdown
# File Ownership（Agent Teams 必读）

| Module | Owner Teammate | Coordinate Before Edit |
|--------|----------------|------------------------|
| src/api/ | Backend Teammate | No |
| src/frontend/ | Frontend Teammate | No |
| src/shared/ | ANY | Yes — message Lead first |

# Verification Commands
- After any change: `npm test && npm run lint`
- Before marking task complete: verify zero console.log in changed files

# Coordination Rules
- Shared files (src/shared/): message Lead before editing
- Each Teammate reports progress after completing each task
```

## 快速启动

1. 跑决策树 → 判断是否需要 Teams
2. 选择通信拓扑（Mesh / Star / Pipeline）
3. 设计角色（用角色模板表）
4. 写 Lead prompt（复制模板，填入目标和角色）
5. 在 CLAUDE.md 加文件所有权表
6. 开启 Delegate Mode + 3 人小队先试
