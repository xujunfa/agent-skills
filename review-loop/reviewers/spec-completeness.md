# Spec Completeness Reviewer

**Purpose:** 验证 spec 文档的完整性、一致性，确认其已准备好进入 implementation planning 阶段。

## References

<!-- 此 reviewer 独立运行，无需跨文档引用。 -->

## Dimensions

| Category | What to Look For | Severity |
|----------|------------------|----------|
| Completeness | TODO 标记、占位符文本、「TBD」、不完整的 section | Critical |
| Coverage | 缺少 error handling、edge cases、integration points 的讨论 | Critical |
| Consistency | 内部矛盾、冲突的 requirements | Critical |
| Scope | 是否聚焦于单一 plan 的范围——不应覆盖多个独立子系统 | Critical |
| Architecture | 各 unit 是否有清晰的边界和定义良好的 interfaces，是否可独立理解和测试 | Critical |
| Clarity | 可能导致误解的模糊 requirements | Important |
| YAGNI | 未被请求的 features、过度工程、不必要的复杂度 | Important |

## Critical Signals

- 任何 TODO 标记或占位符文本
- 包含「to be defined later」或「will spec when X is done」的 section
- 明显比其他 section 缺乏细节的部分
- 缺少清晰边界或 interfaces 的 units——是否能在不阅读内部实现的情况下理解每个 unit 的职责？

## Pass Criteria

- **Auto-approve when:** No Critical issues found
- **Flag when:** Any Critical issue exists
- **Advisory:** Important and Minor issues are reported but do not block approval
