---
name: agent-research
description: >
  中间件 skill：为需要深度研究的任务生成高质量"研究任务书"（Brief），
  用户委派给外部 AI Agent（Grok/ChatGPT/Manus 等）执行，
  回收结果后进行结构化提取和质量验收。
  当用户提到以下关键词时触发：
  "research brief"|"委派研究"|"agent research"|"外部搜索"|
  "生成研究任务"|"delegate research"|"深度研究"|"研究任务书"
---

# Agent Research

将"机械搜索"升级为"智能委派研究"。生成高质量 Brief 让外部 Agent 执行，回收结果后验收输出。

## 入口模式

| 入口 | 触发 | 起始步骤 |
|------|------|----------|
| 独立使用 | 用户直接描述研究需求 | Step 1 |
| 协议调用 | 调用方 skill 传入 research_context | Step 2 |

## 流程

### Step 1: 需求理解（独立使用时）

与用户对话明确：
- 研究主题和背景
- 范围（聚焦什么、排除什么、时效要求）
- 深度：`quick`（单 brief）| `standard`（1-2 brief）| `deep`（2-3 brief 并行）
- 验收标准（研究完成的定义）

### Step 2: Phase A — 内置快速搜索

执行 2-3 轮 WebSearch 获取权威信源和基础信息，锁定官方文档、GitHub 仓库等权威来源。
- ✅ 已充分 → 直接输出结果，流程结束
- ⚠️ 有缺口 → 告知用户，进入 Step 3

### Step 3: Phase B — 生成 Brief(s)

**⚠️ 关键步骤 — 遵循 `references/brief-generation-guide.md`**

基于 Step 2 结果 + 研究需求，智能生成 1~N 个 Brief：
- Brief 必须智能生成，严禁机械填充模板
- 每个 Brief 独立可执行，可同时发给不同 Agent
- 展示给用户，说明"请将以下 Brief 粘贴给您的外部 Agent"

### Step 4: 结果回收与质量网关

**遵循 `references/quality-gateway-guide.md`**

用户粘贴外部 Agent 结果后：
1. 结构化提取（按研究问题逐项提取答案）
2. 验收检查（比对 Brief 中的验收标准）
3. 结果路由：
   - ✅ 全部通过 → 输出结构化结果
   - ⚠️ 部分缺失 → 展示缺口，生成补充 Brief（回到 Step 3）
   - ❌ 严重不足 → 建议换 Agent 或调整方向

### Step 5: 结构化输出

输出格式见 `references/quality-gateway-guide.md`。协议调用传递给调用方；独立使用询问是否保存。
其他 skill 接入方式见 `references/integration-protocol.md`。

## Edge Cases

| Case | Action |
|------|--------|
| WebSearch 已充分覆盖 | 跳过 Brief 生成，直接输出 |
| 用户主动要求跳过 WebSearch | 直接从 Step 3 开始 |
| 外部 Agent 结果完全跑偏 | 分析原因，重新生成更精确的 Brief |
| 用户粘贴多个 Agent 的结果 | 合并后统一验收 |
| 补充 Brief 超过 2 轮 | 建议用户直接补充缺失信息或降低验收标准 |

## Maintenance

- Version: 1.0.0
- Created: 2026-03-06
- Design: `docs/plans/2026-03-06-agent-research-design.md`
