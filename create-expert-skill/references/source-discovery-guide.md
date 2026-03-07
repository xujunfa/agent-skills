# Source Discovery Guide

## Layered Search Strategy

Execute layers in order. **Stop early** if 3-5 high-quality sources are found after any layer.

### Layer 1 — Official Documentation (Priority 1)

- **Query**: `"{tool} official documentation"`
- **Target**: Getting-started guides, API references, configuration specs
- **Why first**: Most authoritative; sets baseline for all other layers

### Layer 2 — GitHub Repository (Priority 2)

- **Query**: `"{tool} github repository"`
- **Target**: README, CHANGELOG, examples/ directory, issue tracker patterns
- **Why**: Source of truth for open-source tools; reveals real-world usage and known issues

### Layer 3 — Troubleshooting & FAQ (Priority 3)

- **Query**: `"{tool} common issues troubleshooting OR FAQ"`
- **Target**: Stack Overflow threads, official FAQ, migration guides, known gotchas
- **Why**: Fills the gap between docs and practice

### Layer 4 — Community Resources (Priority 3-4)

- **Query**: `"awesome-{tool}" OR "{tool} community resources guide"`
- **Target**: Curated lists, tutorials by recognized experts, comparison articles
- **Why**: Surfaces knowledge not in official docs (patterns, anti-patterns, ecosystem)

### Layer 5 — Version Info (Priority 2, conditional)

- **Query**: `"{tool} changelog latest version release"`
- **Target**: Current stable version, breaking changes, deprecation notices
- **When**: Only execute if layers 1-2 did NOT reveal the current version
- **Why**: Ensures the skill targets the right version

---

## Quality Filters

Apply these filters to every candidate source before presenting to the user.

| Filter | Rule | Exception |
|---|---|---|
| **Freshness** | Exclude if older than 3 years | Timeless concept docs (RFCs, specs, seminal design docs) |
| **Quality** | Exclude AI-generated SEO, zero-star forks, content farms | None |
| **Diversity** | Max 2 results from the same domain | Official docs may have 3 if covering distinct topics |
| **Relevance** | Exclude pages about different tools with similar names | None |
| **Accessibility** | Note if login/paywall required | Mark as "optional" — never auto-include gated content |

**Content farm signals**: thin content with excessive ads, keyword-stuffed titles, scraped Stack Overflow answers without attribution.

---

## Presentation Format

After filtering, present sources to the user in two groups.

### Template

```
## Discovered Sources for {tool}

### Recommended (auto-included unless you exclude)
1. [{title}]({url}) — {type} — {one-line reason}
2. [{title}]({url}) — {type} — {one-line reason}
3. [{title}]({url}) — {type} — {one-line reason}

### Optional (included only if you confirm)
4. [{title}]({url}) — {type} — {one-line reason}
5. [{title}]({url}) — {type} — {one-line reason}

**Reply with:**
- "confirm" — accept Recommended as-is
- "+URL" — add a source (I'll infer its type)
- "+URL as {type}" — add with explicit type
- "-{number}" — exclude a Recommended source
- Any combination, e.g.: "confirm, -2, +https://example.com"
```

### Source Type Labels

Use these labels for the `{type}` field:

| Label | Meaning |
|---|---|
| `official-docs` | Vendor/maintainer documentation |
| `repo` | GitHub/GitLab repository |
| `tutorial` | Step-by-step guide or walkthrough |
| `troubleshooting` | FAQ, common issues, debugging guides |
| `community` | Awesome list, curated resource, forum thread |
| `changelog` | Release notes, version history |
| `api-ref` | API reference or specification |
| `blog` | Blog post by a recognized expert or team |

---

## User-Provided Sources

When the user adds a source with `+URL`:

### Type Inference Rules

| Domain Pattern | Inferred Type |
|---|---|
| `docs.*`, `*.readthedocs.io` | `official-docs` |
| `github.com`, `gitlab.com` | `repo` |
| `stackoverflow.com`, `*.stackexchange.com` | `troubleshooting` |
| `medium.com`, `dev.to`, `*.blog` | `blog` |
| URL path contains `changelog` or `releases` | `changelog` |
| URL path contains `api` or `reference` | `api-ref` |

### Ambiguity Handling

If the domain does not match any pattern above:
1. Ask: _"What type of source is this? (docs / tutorial / troubleshooting / blog / other)"_
2. If user replies with `+URL as {type}`, use the explicit type directly

### Priority Assignment

| Inferred Type | Assigned Priority |
|---|---|
| `official-docs`, `api-ref` | 1 |
| `repo`, `changelog` | 2 |
| `tutorial`, `troubleshooting` | 3 |
| `community`, `blog` | 4 |

User-provided sources always enter the **Recommended** group regardless of priority — the user has explicitly chosen them.

---

## Phase B: 覆盖度评估与深度研究

源确认完成后，评估已确认源是否充分覆盖所有需要的知识维度。

### 评估信号

对照 Step 1 中用户确认的关注领域和工具性质对应的知识分区，检查：
- 每个知识分区（如 quick-start, core-api, common-errors）是否有对应信源
- 信源中是否包含社区实践/踩坑经验（不只是官方文档）
- 信源的深度是否足以支撑 ≤200 行的 reference file

### 覆盖度结果

- **充分**：所有知识分区有对应信源 → 继续 Step 3（用户确认）
- **有缺口**：展示缺口，询问是否启动 `agent-research` 补充深度研究
  - 是 → 传入 research_context（purpose, scope, existing_knowledge, output_needs），`agent-research` 接管
  - 否 → 继续使用现有源，在 Step 4 distill 时尽力补充
