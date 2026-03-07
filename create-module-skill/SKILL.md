---
name: create-module-skill
description: >
  Use when creating a new module-level knowledge Skill for a large frontend repository.
  当用户提到 "创建模块 skill"|"为 XX 模块建知识库"|"create module skill"|
  "新建模块 skill"|"给 XX 模块做 skill"|"module skill 模板"|
  "为这个模块创建开发指南"|"build knowledge base for module"|
  "我想给 XX 模块做个知识索引" 时触发。
---

# 创建模块知识 Skill

## 概览

系统化地为大型前端仓库中的某个模块创建知识 Skill。产出轻量索引 SKILL.md + references/ 深度知识文件，使后续需求开发和 bug 修复不再从零开始。

核心原则：**提炼能让 Claude 在 30 秒内做出正确判断的知识 — 关系、决策逻辑、踩坑经验、隐含约定。**

## Claude 使用指令

当本 Skill 被加载时，你必须：

1. **严格按照下方 5 个 Step 顺序执行**，不要跳步
2. 每个 Step 完成后，向用户展示产出并确认后再进入下一步
3. **Step 1 和 Step 2 合并执行**：一次性读取代码，同时产出模块地图和 5 维度知识，避免重复读文件浪费 token
4. 在 Step 3 填充模板时，读取 `references/skill-template.md` 获取标准模板
5. 在 Step 3 填充 references 文件时，读取 `references/reference-templates.md` 获取各文件模板
6. 完成后，读取 `references/quality-checklist.md` 逐项自检

## Step 0: 前置检查

在开始前执行：

1. **检查已有 module-* skill**：搜索 `~/.claude/skills/module-*`，确认是否已存在目标模块或关联模块的 skill
   - 已存在同名 → 提示用户是否要覆盖/增量更新
   - 存在关联模块（如目标模块依赖的模块） → 记录下来，Step 3 时在 architecture.md 中做交叉引用

## Step 1+2: 模块侦察 & 知识萃取（合并执行）

**一次性读取代码**，同时完成侦察和萃取，避免重复读文件。

### 侦察产出（模块原始地图）

向用户确认模块的主目录后执行：

1. 列出模块目录树（2 层深度）
2. 查找关联目录（extensions、plugins、components 等）
3. 识别所有 service/controller 文件及其 class 名
4. 识别 types、const、interface 等类型定义文件
5. 统计 TODO/FIXME 数量和分布
6. 查找相关测试文件

### 萃取产出（5 维度知识）

读取代码的同时，按 5 个维度提取知识。**所有维度完成后一次性展示给用户，集中收集反馈。**

| 维度     | Claude 提取                               | 向用户追问                                   |
| -------- | ----------------------------------------- | -------------------------------------------- |
| 架构     | 读 service 入口，追踪核心调用链和组件关系 | "这个流程有没有遗漏？有没有隐含的依赖？"     |
| 数据模型 | 读 types + const，整理接口、枚举、ID 格式 | "有没有不明显的数据约定或格式陷阱？"         |
| 常见 bug | 读 TODO/FIXME + git log bugfix commits    | "你踩过哪些坑？哪些 bug 反复出现？"          |
| 开发模式 | 找已有 feature 的扩展点，归纳修改路径     | "加新功能通常要改哪些文件？有什么隐含步骤？" |
| 调试工具 | 找 logger、feature flags、运行时检查      | "你平时怎么调试这个模块？有什么技巧？"       |

**知识质量标准**：这条知识能否在 30 秒内让 Claude 做出正确判断？

```
❌ "accept 逻辑在 revision-service-impl.ts 的 handleRevisionStatusChange 中"
   （Claude 通过 grep 在 30 秒内也能定位到，写了等于没写）

✅ "Accept REPLACE 类型时必须同时处理 REPLACE_ADD 和 REPLACE_DELETE，
    只处理一个会导致残留标记。排查：检查 calcRevisionHandleInfo() 的 subType 分支"
   （经验陷阱 + 具体排查路径，让 Claude 跳过试错直接命中）
```

**展示给用户**：模块地图 + 5 维度知识，一次性确认边界和知识是否完整。

## Step 3: 填充模板

1. 读取本 Skill 的 `references/skill-template.md`，创建目标模块的 `SKILL.md`
2. 读取本 Skill 的 `references/reference-templates.md`，创建各 `references/*.md` 文件
3. 默认目录结构（可根据模块特点调整）：
   ```
   ~/.claude/skills/module-[名称]/
   ├── SKILL.md
   └── references/
       ├── architecture.md
       ├── data-model.md
       ├── common-bugs.md
       ├── dev-patterns.md
       └── debug-tooling.md
   ```
4. **灵活调整 references 文件**：5 个文件是推荐默认，但应根据模块特点调整：
   - 如果模块没有 debug 工具（无 logger、无 FG），可省略 `debug-tooling.md`
   - 如果模块跨模块集成是核心痛点，可新增 `cross-module-integration.md`
   - 如果 Mobile/PC 差异是主要复杂度来源，可新增 `platform-differences.md`
   - 调整后需同步更新 SKILL.md 的「按需加载指引」表格
5. 如果 Step 0 发现了关联 module-* skill，在 architecture.md 的「模块间依赖」中加入交叉引用：
   `详见 module-[关联模块名] skill`
6. 逐个文件展示给用户确认

## Step 4: 场景验证

用具体可操作的场景验证 skill 是否有效：

1. **Bugfix 场景**：构造一个具体的症状描述（基于 common-bugs.md 中的某个分类），验证通过 skill 能否在 3 步内定位到正确的排查入口。来源：检索 git log 中的 fix: 提交，或请用户提供真实 MR 链接。切勿自行捏造场景。
2. **Feature 场景**：构造一个具体的扩展需求（基于 dev-patterns.md 中的某个模式），验证通过 skill 能否列出完整的需要修改的文件清单。
3. **触发准确性**：至少 5 个应触发 prompt + 3 个不应触发 prompt，验证 description 的关键词覆盖。

向用户报告验证结果，根据发现调整内容。

## Step 5: 交付 & 迭代建议

1. 读取 `references/quality-checklist.md` 逐项自检
2. 确认所有文件已写入正确位置
3. 告知用户迭代策略：
   - 每次修 bug 后，补充到 `common-bugs.md`（格式：症状 → 排查路径 → 已知陷阱）
   - 每次做需求后，补充到 `dev-patterns.md`（格式：需要修改的文件清单）
   - 如果触发有误触发或漏触发，调整 SKILL.md 的 description 关键词
   - 模块架构变更时及时更新 architecture.md
