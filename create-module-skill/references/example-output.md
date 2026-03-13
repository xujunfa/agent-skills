# 填充风格示例

以虚构的 "文档协同编辑（collab-editing）" 模块为例，展示各文件填充后的风格和详略度。**这是风格参考，不是完整模板。** 实际产出请基于真实代码。

---

## 示例 1: SKILL.md 片段

```markdown
---
name: module-collab-editing
description: >
  专为 SuperDoc 中 collab-editing 模块提供知识索引。
  当用户提到以下关键词组合时触发：
  "协同编辑" + ("bug"|"功能"|"需求"|"排查"|"架构"),
  "collab"|"实时协作"|"OT 冲突"|"光标同步"|"编辑锁"|
  "多人编辑不一致"|"协同模式切换"|"离线编辑"。
  与 workflow-bugfixing 或 workflow-feature-development 组合使用。
---

# collab-editing 模块知识索引

## 模块定位（一句话）

collab-editing 是 SuperDoc 的实时多人协同编辑引擎，基于 OT 算法处理操作变换，
通过 WebSocket 同步编辑状态，支持在线/离线双模式。

## 关键文件速查

| 领域     | 入口文件                                    |
| -------- | ------------------------------------------ |
| 类型定义 | `src/collab/types/operation.ts`            |
| OT 核心  | `src/collab/ot/transform-engine.ts`        |
| 同步服务 | `src/collab/sync/sync-service.ts`          |
| 状态管理 | `src/collab/state/collab-state-manager.ts` |
| WebSocket| `src/collab/transport/ws-client.ts`        |

## 按需加载指引

| 你正在做什么                    | 读取                          |
| ------------------------------ | ----------------------------- |
| 理解 OT 变换流程、组件关系      | `references/architecture.md`  |
| 理解 Operation 类型、冲突模型   | `references/data-model.md`    |
| 排查同步丢失、光标跳动等 bug    | `references/common-bugs.md`   |
| 添加新操作类型、新协同功能      | `references/dev-patterns.md`  |
| 配置调试工具、查看同步日志      | `references/debug-tooling.md` |
```

---

## 示例 2: architecture.md 片段

```markdown
## 核心调用链

### 场景 1：用户输入文字（最常见操作）

```
1. Editor.onInput() — 捕获用户输入
2. -> OperationBuilder.buildInsert(pos, text) — 构造 Insert Operation
3. -> CollabStateManager.applyLocal(op) — 本地立即应用（乐观更新）
4. -> SyncService.send(op) — 通过 WebSocket 发送到服务端
5. -> 服务端广播 transformed op 给其他客户端
6. -> SyncService.onRemoteOp(transformedOp) — 接收远端操作
7. -> TransformEngine.transform(localPending, remoteOp) — OT 变换
8. -> CollabStateManager.applyRemote(transformedOp) — 应用远端操作
```

### 关键设计决策

| 决策              | 选择了什么              | 为什么不选另一种                          |
| ----------------- | ---------------------- | ---------------------------------------- |
| 冲突解决算法       | OT（操作变换）          | CRDT 内存开销大，文档型编辑 OT 更成熟      |
| 本地应用策略       | 乐观更新 + 回滚        | 等服务端确认延迟太高，影响输入手感          |
| 离线处理           | 操作队列 + 重连重放     | 本地快照方案合并冲突率高，用户体验差        |
```

---

## 示例 3: common-bugs.md 片段

```markdown
## 1. 多人编辑时内容丢失（线性排查格式）

**症状**：两人同时编辑同一段落，其中一人的修改消失。

**排查路径**（按可能性从高到低）：
1. 先检查 OT 变换是否正确 → `transform-engine.ts` 的 `transformInsertInsert()`，
   打印 `op1.pos` 和 `op2.pos` 确认位置偏移计算
2. 然后检查操作序列号是否连续 → `sync-service.ts` 的 `validateSeqNumber()`，
   断点看 `lastAckedSeq` 和 `pendingOps.length`
3. 再检查 WebSocket 是否有丢包 → 浏览器 Network 面板过滤 WS 帧，对比发送数和确认数

**已知陷阱**：当 Insert 和 Delete 操作在同一位置时，`transformInsertDelete()` 必须
先处理 Delete 再偏移 Insert，顺序反了会导致光标后所有内容偏移错位。这是历史上复现率最高的 bug。

---

## 2. 光标位置跳动（决策树格式）

```
光标跳到错误位置？
├── 只在收到远端操作后跳？
│   ├── 是 → 检查 CursorTransformer.transformCursor() 的位置映射逻辑
│   │        — 关注 `collapsedRange` 是否被正确处理
│   └── 否 → 检查 Editor.setCursorPos() 是否被非协同逻辑调用
├── 只在特定操作类型后跳？
│   └── → 检查该操作类型在 OperationBuilder 中是否正确设置了 cursorAfter 属性
└── 重连后跳？
    └── → 检查 SyncService.replay() 是否在重放时更新了光标映射表
```

**已知陷阱**：CursorTransformer 只处理 Insert/Delete，如果新增了 Format 操作类型
但没扩展 transformCursor()，光标位置在格式变更后不会更新。新增操作类型时必须同步扩展。
```

---

## 示例 4: dev-patterns.md 片段

```markdown
## 模式 1: 添加新的操作类型（如 "SetHeading"）

**需要修改的文件（按顺序）：**

1. `src/collab/types/operation.ts` — 在 OperationType 枚举新增 `SET_HEADING`，
   定义 `ISetHeadingOp` 接口（必须包含 `pos`, `level` 字段）
2. `src/collab/ot/transform-engine.ts` — 新增 `transformSetHeading*()` 系列方法，
   处理与 Insert/Delete/Format 的两两变换（4 个方法）
3. `src/collab/ot/operation-builder.ts` — 新增 `buildSetHeading(pos, level)` 工厂方法
4. `src/collab/state/collab-state-manager.ts` — 在 `applyOperation()` 的 switch 中
   新增 `SET_HEADING` 分支
5. `src/collab/cursor/cursor-transformer.ts` — 扩展 `transformCursor()` 处理新类型
6. `tests/collab/ot/transform-set-heading.test.ts` — 新增变换测试（至少覆盖：
   同位置冲突、跨位置、与 Delete 交叉 3 种场景）

**参考实现**：`Format` 操作类型在 PR #2847 中添加，结构完全一致。

**隐含步骤（容易遗漏）：**
- [ ] CursorTransformer 必须同步扩展（遗漏 → 光标跳动 bug）
- [ ] 离线操作队列的序列化/反序列化需要支持新类型
- [ ] 服务端 OT 引擎也需同步更新（不在本模块，需通知后端）
```

---

## 风格要点

1. **具体到函数名和字段名** — 不是 "检查变换逻辑"，而是 "检查 `transformInsertDelete()` 的位置偏移计算"
2. **写 "为什么" 而非 "是什么"** — 不是 "使用 OT 算法"，而是 "CRDT 内存开销大，OT 更适合文档编辑"
3. **已知陷阱要有因果链** — 不是 "注意顺序"，而是 "顺序反了会导致光标后所有内容偏移错位"
4. **隐含步骤标注遗漏后果** — 不是 "记得更新 CursorTransformer"，而是 "遗漏 → 光标跳动 bug"
5. **排查路径按概率排序** — 最常见的原因放第一步，不要按代码调用顺序排
