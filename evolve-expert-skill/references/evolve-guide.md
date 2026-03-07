# Evolve Mode — Full Workflow

Evolve updates reference knowledge from external sources when a tool/library
releases new versions or content becomes stale.

---

## E1 — Version Probe

1. Run one WebSearch: `"{tool_name} latest version release"`.
2. Compare result with `version_anchors` in `.sources.yml`.
3. Decision table:

| Probe Result | Action |
|---|---|
| Version unchanged | Inform user. Ask if they want to force-update. |
| Version changed | Proceed to E2. |
| Cannot determine | Treat as changed. Proceed to E2. |

If user declines force-update → stop with summary.

---

## E2 — Incremental Fetch

Only fetch sources whose `version_anchor` is outdated. For each source in
`.sources.yml`:

1. **Current version matches** → skip (no fetch needed).
2. **Outdated version** → fetch via WebFetch or WebSearch.
3. **Deprecated/inaccessible** → skip, mark in `.sources.yml`:
   ```yaml
   status: inaccessible
   last_error: "{reason}"
   last_attempted: {YYYY-MM-DD}
   ```

### Retry Policy

- On fetch failure: retry **once**.
- On second failure: mark as inaccessible, continue with remaining sources.
- Never block the entire evolve on a single source failure.

### Fetch Output

For each successfully fetched source, produce a diff summary:
- Key sections that changed
- New sections added
- Sections removed or deprecated

---

## E3 — Incremental Distill

Compare fetched content against existing reference files.

| Comparison | Action |
|---|---|
| No meaningful change | Update `last_fetched` date only |
| Content updated | **Edit** the reference file (preserve structure) |
| New topic discovered | **Create** new reference file |
| Topic deprecated/removed | Add `> DEPRECATED as of {version}` marker at top |

### Rules

- Always use Edit (not Write) for existing files.
- Preserve the file's existing structure and section ordering.
- Add a `<!-- Updated: {YYYY-MM-DD} from {source} -->` comment at the top.
- If a reference file grows beyond 200 lines, split into two files.

---

## E4 — New Source Discovery (Conditional)

**Only run when:**
- Major version change detected (e.g., v2 → v3), OR
- User explicitly requests source discovery, OR
- Last evolve was >30 days ago (check `.sources.yml` `last_evolved`).

### Search Queries

1. `"{tool} {version} guide OR tutorial OR migration"`
2. `"{tool} {version} breaking changes"`

### Evaluation

For each candidate source:

1. Check it is not already in `.sources.yml` (match by URL or domain+path).
2. Assess quality: official docs > reputable blogs > community posts.
3. Present candidates to user for confirmation:
   ```
   Found 2 new sources:
   1. [Migration Guide v3] https://... (official docs)
   2. [Breaking Changes] https://... (blog post)
   Add to .sources.yml? (y/n for each)
   ```
4. Only add user-confirmed sources.

---

## E4.5 — Agent Research 补充（可选）

Evolve 的 E2-E4 依赖内置 WebSearch/WebFetch，对于以下场景可能覆盖不足：
- 大版本升级（v2→v3）涉及架构级变化，社区迁移经验难以通过关键词搜到
- 工具生态发生重大变化（如被收购、换维护者、核心 API 重设计）
- 现有信源质量下降（文档过时、社区迁移到新平台）

### 触发条件

E4 完成后，评估已更新的 references 是否充分覆盖重要变更。如果有明显缺口：

```
E2-E4 已完成。发现以下覆盖缺口：
- ⚠️ {缺口描述，如"v3 迁移的社区实践经验缺失"}

是否启动 agent-research 补充深度研究？
1. 是 → 生成研究 Brief，委派外部 Agent
2. 否 → 跳过，继续 E5
```

### 自动化模式

用户可以在触发 evolve 时指定跳过 agent-research（如"evolve xx-expert, 自动跑完"或"evolve xx-expert --no-research"），此时 E4.5 整体跳过，直接进入 E5。

### 结果合并

agent-research 返回的 `research_result` 中：
- findings 作为额外输入，按 E3 规则写入对应 reference files
- sources 中的 URL 补充到 `.sources.yml`（type 由 URL 推断，priority 默认 3）

---

## E5 — Trigger Evolution + Wrap-up

### SKILL.md Updates (same as Distill D4)

1. Extract new keywords from updated references.
2. Compare with SKILL.md `description` — append if new (keep <1024 chars).
3. Bump version: **minor bump** (e.g., 1.0.1 → 1.1.0).

### .sources.yml Updates

For each source processed:
- Update `version_anchor` to new version.
- Update `last_fetched` to today.
- Update `last_evolved` timestamp.

### Output Format

```
## Evolve Complete

- **Tool:** {tool_name}
- **Version:** {old_version} → {new_version}
- **Sources checked:** {N}
- **Sources updated:** {N}
- **Sources skipped:** {N} (unchanged/inaccessible)
- **New sources added:** {N}
- **References modified:** {list of files}
- **References created:** {list or "none"}
- **SKILL.md version:** {old} → {new}
- **Keywords added:** {list or "none"}
```

### Change Summary

After the table, provide a brief (3-5 bullet) summary of the most important
changes a developer should know about. Focus on:
- Breaking changes
- New capabilities
- Deprecated features
- Migration steps (if any)
