---
name: update-module-skill
description: >
  迭代更新已有的 module-* 知识 Skill。支持反应式（基于刚完成的 bugfix/feature 自动提炼）
  和主动梳理（补充特定主题的实现细节）两种模式。
  当用户提到以下关键词时触发：
  "更新 skill"|"迭代 skill"|"补充到 skill"|"skill 需要更新"|"skill 过时了"|
  "update skill"|"update module skill"|"iterate skill"|"skill outdated"|
  "记录一下"|"写进 skill"|"加到 common-bugs"|"补充 dev-patterns"|
  "刚修完，更新一下"|"需求做完了，记录到 skill"|"总结这次改动到 skill"|
  "梳理 XX 的实现"|"补充 XX 的知识"|"整理 XX 模块的 skill"|
  "document XX"|"add knowledge about XX"|"补充关于 XX 的细节"|
  "skill 触发不准"|"skill 信息有误"|"skill 缺少 XX"|
  "大需求合入后更新 skill"|"架构改了，skill 要同步"|"skill 内容过时了"|
  "reconcile skill"|"skill 全面检查"。
  与 workflow-bugfixing 或 workflow-feature-development 的收尾阶段组合使用。
---

# 更新模块知识 Skill（Meta-Skill）

## 概览

对已有 `module-*` Skill 执行**增量更新**，而非从零重建。核心原则与 `create-module-skill` 一致：**提炼能让 Claude 在 30 秒内做出正确判断的知识**。

## Claude 使用指令

当本 Skill 被加载时，你必须：

1. **严格按照下方 4 个 Step 顺序执行**，不要跳步
2. 每个 Step 完成后，向用户展示产出并确认后再进入下一步
3. **在 Step 0 确定使用模式**（反应式 or 主动梳理），后续步骤按对应分支执行
4. **优先 Edit 增量修改**；仅当某段内容整体失效需推翻重写时，允许 Write 重写该文件

## Step 0: 变更定位

### 0.1 加载目标 Skill

1. 确认目标模块：向用户确认要更新哪个 `module-*` skill（如果上下文不明确）
2. 读取目标 Skill 的 **SKILL.md** + 所有 **references/*.md** 文件
3. 记录最近更新时间（来自 Living Document 段落的 `最近更新` 行）

### 0.2 确定使用模式

| 信号 | 模式 | 示例 |
|---|---|---|
| 对话中刚完成了 bugfix / feature-dev | **反应式迭代** | "刚修完 revision accept 的 bug，更新 skill" |
| 用户主动提出补充/重校某个主题 | **主动梳理** | "梳理一下 accept 逻辑" / "大需求合入了，skill 需要更新" |

### 0.3 确定更新范围

根据变更内容，预判需要更新的文件（可多选）：

| 变更类型 | 可能需要更新的文件 |
|---|---|
| 新 bug 模式 / 踩坑经验 | `common-bugs.md` |
| 新扩展点 / 功能开发路径 | `dev-patterns.md` |
| 架构变更 / 新组件 / 新依赖 | `architecture.md` |
| 新类型 / 枚举 / ID 格式 | `data-model.md` |
| 新调试技巧 / Feature Flag | `debug-tooling.md` |
| 触发关键词不准 / 入口文件变更 | `SKILL.md` |

**展示给用户**：当前 Skill 概况 + 识别的模式 + 预判的更新范围，确认后进入 Step 1。

## Step 1: 增量萃取

根据 Step 0 确定的模式，执行不同的知识萃取路径：

### 模式 A: 反应式迭代

知识来源 = **当前对话上下文**（刚完成的 bugfix / feature-dev）

1. **回顾对话历史**，提取：
   - 问题的根因和排查路径（→ common-bugs.md）
   - 修改了哪些文件、为什么（→ dev-patterns.md / architecture.md）
   - 发现的隐含约定或陷阱（→ SKILL.md 注意事项 / common-bugs.md）
   - 新增的类型 / 枚举 / ID 格式（→ data-model.md）

2. **泛化检查**：对话上下文可能是一次非典型的 debug 过程。对每条提取的知识判断：
   - **通用规律**（大部分场景都会遇到）→ 写入标准分类
   - **特定条件下才触发**（需要特定数据/状态组合）→ 写入并标注触发条件
   - **极端边界**（极低概率复现）→ 写入 Edge Cases 表或不写入

3. **对比已有知识**：将提取的知识与目标 Skill 已有内容对比，识别：
   - 需要**新增**的知识点
   - 需要**修正**的过时信息
   - 需要**补充细节**的已有条目

### 模式 B: 主动梳理

知识来源 = **代码和 git 历史**（需要重新阅读）

1. **确认梳理范围**：向用户确认具体要梳理什么
   - **单主题**（如 "accept 逻辑"）→ 定向代码阅读，只读相关文件
   - **大范围**（如 "大需求合入后全面检查"）→ 逐个 reference 文件审计，拿已有内容对照新代码
2. **定向 git 考古**：`git log --oneline --grep="<关键词>"` 或 `git diff <合入前commit>..HEAD -- <模块目录>` 查找变更
3. **Knowledge Pruning**（大范围梳理时必须执行，单主题时对涉及范围执行）：
   对每条已有知识执行存活检查——
   - 知识描述的文件/函数/行为**已删除** → 标记 `STALE`，建议删除
   - 文件存在但**行为已变更**（签名/逻辑/数据结构改变） → 标记 `DRIFT`，需修正
   - **仍然有效** → 保留，更新 `最近更新` 时间戳
   将 pruning 结果（STALE/DRIFT/VALID 清单）展示给用户确认后再执行删除或修正。
4. **按维度萃取**：同模式 A 的提取维度（含泛化检查）

### 知识质量关卡（两种模式通用）

每条提取的知识必须通过 **30 秒判断测试**：

```
❌ "修改了 revision-service-impl.ts 的 handleAccept 方法"
   （Claude 通过 grep 就能找到，写了等于没写）

✅ "Accept REPLACE 时，如果段落包含嵌套的 inline 修订，
    必须先展开 inline 修订再处理段落合并，否则 inline 内容会丢失。
    排查：检查 flattenInlineRevisions() 是否在 mergeParagraphs() 之前调用"
   （经验陷阱 + 具体排查路径）
```

**展示给用户**：提取的知识清单（标注新增 / 修正 / 补充），确认后进入 Step 2。

## Step 2: 增量写入

### 2.1 Diff 预览

在修改之前，先生成**变更摘要**展示给用户：

```
计划修改：
─────────────────────────────
[common-bugs.md]  + 新增 "Accept REPLACE 嵌套 inline 修订" 分类（~15 行）
[SKILL.md]        ~ 修改注意事项第 4 条措辞
[dev-patterns.md] (无变更)
─────────────────────────────
```

用户确认后执行修改。

### 2.2 执行写入规则

1. **优先使用 Edit 工具**（保留用户手动编辑的内容）；但当某段内容整体失效需推翻重写时，允许用 Write 重写该文件
2. **新增内容遵循已有文件的格式惯例**（如 common-bugs.md 的「症状 → 排查路径 → 已知陷阱」格式）
3. **文件超限处理**：修改后若文件超过 200 行，需精简已有内容或拆分
4. **SKILL.md 同步**：如果新增了 references 文件或修改了入口文件，同步更新 SKILL.md 的对应表格

## Step 3: 轻量验证

只验证增量变更。如果本次变更涉及 3 个以上 reference 文件，改用 `create-module-skill/references/quality-checklist.md` 的完整检查清单。

### 更新检查清单

- [ ] 新增内容通过 30 秒判断测试（不是 grep 能找到的信息搬运）
- [ ] 修改后的文件仍在 200 行以内
- [ ] 如果修改了 references 文件，SKILL.md「按需加载指引」已同步
- [ ] 如果修改了入口文件路径，SKILL.md「关键文件速查」已同步
- [ ] 如果修改了 description 触发词，用 3 个 prompt 快速验证触发准确性
- [ ] 新增内容与已有内容无重复或矛盾

### 快速场景验证（仅当变更较大时执行）

用本次实际变更构造验证：
- 更新了 common-bugs.md → 用新增 bug 的症状描述验证能否 3 步定位
- 更新了 dev-patterns.md → 用新增模式验证能否列出完整文件清单

## Step 4: 交付

### 4.1 确保 Living Document 段落

检查目标 module skill 的 SKILL.md 末尾是否有「维护与迭代」段落。**如果没有，追加：**

```markdown
## 维护与迭代（Living Document）

当使用过程中发现以下情况时，请主动建议迭代：
- 新 bug 模式 / 隐含坑 → 补充到 `common-bugs.md`
- 新扩展点 / 架构变更 → 更新 `architecture.md` 或 `dev-patterns.md`
- Claude 触发不准、读错文件、或建议不符实际 → 调整 `SKILL.md`

系统性迭代请使用 `/update-module-skill`。

最近更新：YYYY-MM-DD — 更新摘要
上次审计：YYYY-MM-DD — 审计结论摘要
```

如果目标 Skill 仍使用旧的 `当前版本：x.y.z` 格式，替换为上述 `最近更新 + 上次审计` 格式。

### 4.2 更新时间戳

在目标 Skill 的 Living Document 段落中更新 `最近更新` 行：
- 格式：`最近更新：YYYY-MM-DD — 本次变更的一句话摘要`
- 如果本次是 Mode B 主动梳理，同时更新 `上次审计` 行

### 4.3 更新摘要

向用户展示：
- 本次修改的文件列表和变更概要
- 建议的后续迭代方向（萃取过程中发现的其他可补充但本次未处理的主题）
