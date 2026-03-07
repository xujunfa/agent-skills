# .sources.yml Template

## Purpose

`.sources.yml` is a machine-readable source registry placed at the root of each
expert skill directory. It enables `evolve-expert-skill` to:

- Re-fetch sources to detect upstream changes
- Know which reference file each source maps to
- Track staleness via `last_fetched` and `version_anchor`

---

## Full Template

```yaml
version: 1
created: "{YYYY-MM-DD}"
tool_name: "{tool_name}"
tool_nature: "{CLI tool | SDK/library | Framework | SaaS service}"

sources:
  - id: "{short_slug}"
    url: "{full_url}"
    type: "{official_docs | github_readme | api_reference | changelog | blog_post | man_page}"
    priority: {1-3}
    last_fetched: "{YYYY-MM-DD}"
    version_anchor: "{version_string_or_doc_revision}"
    mapped_to: "references/{filename}.md"
    status: "{active | deprecated | unreachable}"
```

---

## Field Rules

| Field | Required | Type | Valid values / Constraints |
|---|---|---|---|
| `version` | yes | int | Always `1` (schema version) |
| `created` | yes | date | ISO 8601 date, set at creation time |
| `tool_name` | yes | string | Matches SKILL.md frontmatter `name` without `-expert` suffix |
| `tool_nature` | yes | string | One of: `CLI tool`, `SDK/library`, `Framework`, `SaaS service` |
| `sources` | yes | array | At least 1 entry; recommended 2-5 |
| `sources[].id` | yes | string | Unique slug within file, lowercase + hyphens, e.g. `official-docs` |
| `sources[].url` | yes | string | Full URL; must be fetchable or clearly marked `unreachable` |
| `sources[].type` | yes | string | One of: `official_docs`, `github_readme`, `api_reference`, `changelog`, `blog_post`, `man_page` |
| `sources[].priority` | yes | int | `1` = primary (fetch first), `2` = secondary, `3` = supplementary |
| `sources[].last_fetched` | yes | date | ISO 8601 date of last successful fetch |
| `sources[].version_anchor` | yes | string | Version tag, commit SHA prefix, or `"latest"` if unversioned |
| `sources[].mapped_to` | yes | string | Relative path to reference file this source feeds into |
| `sources[].status` | yes | string | One of: `active`, `deprecated`, `unreachable` |

**Constraints:**
- Every `mapped_to` path must correspond to an actual file in `references/`
- A reference file may be fed by multiple sources (many-to-one is fine)
- At least one source must have `priority: 1`
- `id` values must be unique within the file

---

## Concrete Example

Fictional tool: **duckdb** CLI/library

```yaml
version: 1
created: "2026-02-20"
tool_name: "duckdb"
tool_nature: "CLI tool"

sources:
  - id: "official-docs"
    url: "https://duckdb.org/docs/"
    type: official_docs
    priority: 1
    last_fetched: "2026-02-20"
    version_anchor: "v0.10.1"
    mapped_to: "references/cli-commands.md"
    status: active

  - id: "sql-reference"
    url: "https://duckdb.org/docs/sql/introduction"
    type: api_reference
    priority: 1
    last_fetched: "2026-02-20"
    version_anchor: "v0.10.1"
    mapped_to: "references/sql-patterns.md"
    status: active

  - id: "github-changelog"
    url: "https://github.com/duckdb/duckdb/releases"
    type: changelog
    priority: 2
    last_fetched: "2026-02-20"
    version_anchor: "v0.10.1"
    mapped_to: "references/migration-guide.md"
    status: active
```

---

## Placement

```
{tool_name}-expert/
  SKILL.md
  .sources.yml        <-- here, at skill root
  references/
    cli-commands.md
    sql-patterns.md
    migration-guide.md
  experiences/
```

The file is YAML, not Markdown. Ensure proper indentation (2-space).
