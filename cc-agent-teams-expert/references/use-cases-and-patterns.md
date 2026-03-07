# 用例与模式

> **来源:** https://code.claude.com/docs/en/agent-teams + https://www.anthropic.com/engineering/building-c-compiler + https://addyosmani.com/blog/claude-code-agent-teams/ + https://claudefa.st/blog/guide/agents/agent-teams
> **获取时间:** 2026-03-06
> **版本锚点:** Claude Code 2.1.x (experimental)

## 最强用例（按价值排序）

### 1. Competing Hypotheses 调试（最高 ROI）

多 Teammates 并行测试不同假设，互相挑战，最终存活的理论更可能是根因。

```text
Users report the app exits after one message. Spawn 5 teammates
to investigate different hypotheses. Have them talk to each other
to disprove each other's theories, like a scientific debate.
```

**为什么有效：** 串行调查有锚定偏差——一旦探索了一个理论，后续被它拉偏。多独立调查者主动反驳彼此可消除此偏差。

### 2. 多视角 Code Review

```text
Create a team to review PR #142. Spawn three reviewers:
- Security implications
- Performance impact
- Test coverage validation
```

单人评审倾向于一次只关注一类问题。拆分为独立领域 = 安全/性能/测试都获得彻底审查。Lead 综合所有发现。

### 3. 跨层 Feature 开发

前端、后端、测试各由不同 Teammate 负责，消除上下文切换开销：

```text
Build the user notification system with a team:
- Backend: REST API + WebSocket service
- Frontend: notification panel + real-time updates
- Tests: API integration tests + UI component tests
```

### 4. 新模块/Feature 并行开发

Teammates 各自拥有不同组件，互不干扰：

```text
Create a team with 4 teammates to refactor these modules in parallel.
```

### 5. 多视角设计探索

```text
Design a CLI tool for tracking TODOs. Create a team:
- UX perspective
- Technical architecture
- Devil's advocate
```

Lead 综合各角度发现，得出平衡方案。

### 6. 大规模 Inventory/Classification

Teammates 分割大数据集，独立处理各段落。适合需要系统扫描但可并行的任务。

### 7. 研究与文献调查

```text
Research three competing auth libraries in parallel. Each teammate
evaluates one library: security, DX, and performance. Share findings.
```

### 8. 文档生成

多 Teammates 各负责不同模块的文档，Lead 统一风格和结构。

## 标志性案例：16 Agent 编译器

**Anthropic 官方压力测试**——16 个 Claude Agent 用两周构建了 C 编译器。

| 指标 | 数据 |
|---|---|
| Agent 数量 | 16（并行） |
| 会话数 | ~2,000 |
| 产出 | 100,000 行 Rust 代码 |
| 总成本 | ~$20,000 |
| 成果 | 编译 Linux 6.9（x86/ARM/RISC-V）、QEMU、FFmpeg、SQLite、PostgreSQL、Doom |
| 测试通过率 | GCC torture test suite 99% |

**协调方式：** 基于文件锁的任务认领。每个 Agent 通过写入 `current_tasks/` 中的文本文件"锁定"任务。Git 同步强制竞争 Agent 选择不同任务。

**Agent 工作循环：**
1. 认领任务锁
2. 在本地 `/workspace` 工作
3. Pull 上游变更
4. Merge 其他 Agent 的修改
5. Push 变更 + 解除锁
6. 无限循环

**基础设施：** 每个 Agent 在独立 Docker 容器中运行，共享仓库挂载到 `/upstream`。

**关键教训：**
- 高质量测试套件是自主 Agent 的关键——Agent 自主解决问题，测试决定质量
- 输出最小化噪声——"打印几行输出，将重要信息写入文件让 Claude 按需查找"
- 专业化分工：除核心编译器外，专门 Agent 负责文档、代码质量、性能优化、架构重构

## 通用模式

> **完整编排模式（含决策树、通信拓扑、Prompt 模板）见 `orchestration-design-guide.md`。** 以下为快速参考。

| 模式 | 一句话 |
|------|--------|
| Wave Execution | Lead 分发一批任务 → 并行 → 综合 |
| Progressive Refinement | 共享中间发现 → 迭代 → 收敛 |
| Specialization | 每 Teammate 拥有特定领域，不交叉 |
| Builder-Validator Chain | Builder 实现 → Validator 审查 → 反馈循环 |
| Red Team / Blue Team | 对抗式辩论调试（社区模式） |
| Orchestrator-Only | Lead 只协调不写码 + Self-Claim（社区模式） |

## 企业级采用

| 企业 | 成果 |
|---|---|
| TELUS | 13,000+ 自定义 AI 方案，工程代码交付速度 +30%，节省 500,000+ 小时 |
| Zapier | 89% AI 采用率，800+ Agent 内部部署 |
| Rakuten | 在 vLLM（1250 万行代码库）上测试复杂技术任务 |

## Anti-Patterns（何时不用 Agent Teams）

> **完整反模式（含真实失败案例）见 `orchestration-design-guide.md`。** 以下为快速检查表。

- 工作严格串行
- 多人编辑同一文件（紧耦合）
- Workers 无需互相通信（→ 用 Subagents 或 `/batch`）
- 简单并行执行（→ 用 async workflows）
- 单一简单任务（→ 直接做，不需要 Teams 开销）
