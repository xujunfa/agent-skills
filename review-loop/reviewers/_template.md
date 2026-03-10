# <Domain> Reviewer

**Purpose:** <用一句话描述此 reviewer 验证的内容>

## References

<!-- 此 reviewer 进行跨文档校验时需要的命名引用。
     如果是独立审查（无需引用其他文档），此处留空。
     调用方 skill 在 invoke review-loop 时按名称提供这些引用。 -->

- **<name>**: <此引用是什么的简要说明>

## Dimensions

| Category | What to Look For | Severity |
|----------|------------------|----------|
| <维度 1> | <具体的审查信号> | Critical |
| <维度 2> | <具体的审查信号> | Important |
| <维度 3> | <具体的审查信号> | Minor |

<!-- Severity 指引：
     Critical  = 阻塞 approval，必须修复后才能通过
     Important = 建议修复，作为 advisory 报告（不阻塞）
     Minor     = 锦上添花，作为 advisory 报告（不阻塞）

     注意：Severity 是指引而非硬约束。Reviewer 可根据实际影响覆盖。
     不匹配任何预定义 dimension 的 issue 默认为 Important。 -->

## Critical Signals

<!-- Reviewer 必须重点关注的信号 -->
- <信号 1>
- <信号 2>

## Pass Criteria

- **Auto-approve when:** No Critical issues found
- **Flag when:** Any Critical issue exists
- **Advisory:** Important and Minor issues are reported but do not block approval
