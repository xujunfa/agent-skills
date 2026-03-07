# 集成协议

其他 skill 在研究阶段接入 agent-research 的规范。

## 集成模式：A→B 递进

```
Phase A: 内置搜索（调用方 skill 自行执行）
  → WebSearch 获取权威信源和基础信息
  → 评估覆盖度

Phase B: 覆盖度评估
  → ✅ 已充分 → 跳过，继续原流程
  → ⚠️ 有缺口 → 询问用户是否启动 agent-research
  → 用户也可在任何时候主动要求启动

Phase C: Agent Research 补充
  → 构建 research_context 传入
  → agent-research 接管（Brief 生成 → 委派 → 验收）
  → 结构化结果返回调用方
```

**关键：Phase A 不被跳过。** 内置搜索快速建立基础认知，agent-research 在此基础上补充深度。用户也可主动要求跳过 Phase A 直接进入 Brief 生成。

## 调用方传入的上下文

调用方 skill 在切入 agent-research 时，需要提供以下上下文（自然语言描述即可，不需要严格 YAML 格式）：

| 字段 | 含义 | 示例 |
|------|------|------|
| purpose | 为什么需要研究 | "为 Turborepo 创建 expert skill" |
| scope | 研究范围 | "构建工具、缓存机制、CI 集成" |
| depth | 深度 | quick / standard / deep |
| existing_knowledge | Phase A 已获取的信息 | "官方文档显示..." |
| output_needs | 调用方需要什么 | "需要 quick-start、core-api、common-errors 三个维度的知识" |

## 适合接入的 Skill

| Skill | 接入点 | 改动方式 |
|-------|--------|----------|
| `create-expert-skill` | Step 2 (Source Discovery) 之后 | 在 source-discovery-guide.md 末尾加覆盖度评估 + agent-research 引用 |
| `evolve-expert-skill` | Evolve Mode E2-E4 | 在 evolve-guide.md 的 Fetch 阶段加 agent-research 选项 |

## 不适合接入的场景

- **module-* skills** — 研究对象是本地源码，不需要网络搜索
- **简单版本探测** — WebSearch 一轮足够，不需要外部 Agent
- **快速事实查询** — "X 的最新版本是什么"用 WebSearch 更快

## 接入示例：create-expert-skill

**当前 Step 2 流程（不变）：**
执行 source-discovery-guide.md 的分层搜索 → 展示发现的源 → 用户确认

**新增（Step 2 之后）：**
评估已确认源的覆盖度。判断信号：
- 是否缺少某个知识维度（如有文档但无社区实践）
- 用户需要的 output_needs 是否都有信源覆盖
- 信源的深度是否足以支撑 ≤200 行的 reference file

如果有缺口：
```
Phase A 搜索已完成，发现以下信源：
[已确认的源列表]

覆盖度评估：
- ✅ 官方文档：已覆盖
- ✅ GitHub 仓库：已覆盖
- ⚠️ 社区实践/踩坑经验：缺少
- ⚠️ 与同类工具对比：缺少

建议启动 agent-research 补充深度研究。是否继续？
```

用户确认后，构建 research_context 切入 agent-research 流程。

## 结果回流

agent-research 输出的 `research_result` 回到调用方后：
- findings 中的每个 answer 作为额外知识输入
- sources 中的 URL 可以补充到 `.sources.yml`
- confidence: low 的 findings 在写入 reference file 时标注"待验证"
