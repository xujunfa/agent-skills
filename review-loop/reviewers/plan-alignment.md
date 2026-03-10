# Plan Alignment Reviewer

**Purpose:** 验证 plan chunk 的完整性、与 spec 的对齐程度，以及 task 分解的合理性。

## References

- **spec**: 此 plan 所实现的设计 spec——用于验证覆盖范围和对齐程度

## Dimensions

| Category | What to Look For | Severity |
|----------|------------------|----------|
| Completeness | TODO 标记、占位符、不完整的 tasks、缺失的 steps | Critical |
| Spec Alignment | Chunk 是否覆盖了相关的 spec requirements，是否存在 scope creep | Critical |
| Task Decomposition | Tasks 是否原子化、边界是否清晰、steps 是否可执行 | Important |
| File Structure | 文件是否有清晰的单一职责，是否按职责而非技术层次拆分 | Important |
| File Size | 新建或修改的文件是否可能增长到难以整体理解的大小 | Important |
| Task Syntax | Steps 是否使用 checkbox 语法（`- [ ]`）以便追踪进度 | Minor |
| Chunk Size | 每个 chunk 是否控制在 1000 行以内 | Minor |

## Critical Signals

- 任何 TODO 标记或占位符文本
- 使用「similar to X」但未提供实际内容的 steps
- 不完整的 task 定义
- 缺少 verification steps 或预期输出
- 计划承载多重职责或可能膨胀的文件

## Pass Criteria

- **Auto-approve when:** No Critical issues found
- **Flag when:** Any Critical issue exists
- **Advisory:** Important and Minor issues are reported but do not block approval
