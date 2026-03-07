# Expert Skill Template

Template for generating `{tool}-expert/SKILL.md` and its reference files.

---

## 1. SKILL.md Frontmatter Template

```yaml
---
name: {tool_name}-expert
description: "Use when working with {tool_display_name} — covers {brief_scope}."
trigger_keywords:
  - {tool_name}
  - {alias_1}
  - {cli_command_1}
  - {core_concept_1}
  - {config_file_name}
  - {common_error_fragment}
---
```

**Naming rule:** Always `{tool_name}-expert`. Use lowercase, hyphens only.
Example: `ruff-expert`, `prisma-expert`, `aws-s3-expert`.

---

## 2. SKILL.md Body Structure

```markdown
# {tool_display_name} Expert Knowledge

## Claude Usage Instructions
When the user asks about {tool_display_name}, load the relevant reference
file(s) below on demand. Do NOT load all files at once.

## Tool Overview
- **Nature:** {CLI tool | SDK/library | Framework | SaaS service}
- **Current version anchor:** {version}
- **Official docs:** {url}

## On-Demand Loading Guide

| Situation / Question | Load file |
|---|---|
| {situation_1} | `references/{file_1}.md` |
| {situation_2} | `references/{file_2}.md` |
| {situation_3} | `references/{file_3}.md` |

## Key Gotchas
- {gotcha_1}
- {gotcha_2}
- {gotcha_3}

## Maintenance
- **Version:** 1.0.0
- **Created:** {date}
- **Tool version anchor:** {tool_version}
```

SKILL.md must stay **<= 80 lines**. Push details into reference files.

---

## 3. Reference File Format

Every reference file starts with a metadata header:

```markdown
# {Title}

> **Source:** {url}
> **Fetched:** {YYYY-MM-DD}
> **Version anchor:** {tool_version or doc_revision}

## {Section}
- Actionable point (not prose)
- Decision-enabling facts
...
```

**Rules:**
- Max **200 lines** per reference file
- Content must be distilled, not copied — rewrite for Claude's decision-making
- Each bullet should pass the 30-second judgment test
- Group by task/scenario, not by source document structure

---

## 4. Adaptive Knowledge Partitioning

| Tool nature | Recommended reference files |
|---|---|
| **CLI tool** | `cli-commands.md`, `config-and-flags.md`, `error-patterns.md`, `migration-guide.md` |
| **SDK / library** | `api-surface.md`, `patterns-and-idioms.md`, `error-handling.md`, `integration.md` |
| **Framework** | `project-structure.md`, `core-concepts.md`, `routing-and-middleware.md`, `deployment.md` |
| **SaaS service** | `api-reference.md`, `auth-and-setup.md`, `rate-limits-and-quotas.md`, `webhooks.md` |

Select 2-5 files based on actual content volume. Not every partition is required.

**Example — ruff-expert (CLI tool):**
```
references/
  cli-commands.md        # check, format, rule selection
  config-and-flags.md    # pyproject.toml [tool.ruff], key flags
  error-patterns.md      # common rule violations, fixes
```

---

## 5. Trigger Keyword Generation Rules

Extract from distilled content:

| Category | Examples (ruff) | Extraction method |
|---|---|---|
| Tool name + aliases | `ruff`, `ruff-linter` | Name, common aliases |
| CLI commands | `ruff check`, `ruff format` | Top-level subcommands |
| Core concepts | `rule selection`, `fixable rules` | Repeated domain terms |
| Error fragments | `E501`, `F401`, `I001` | Common codes/messages |
| Config file names | `pyproject.toml ruff`, `ruff.toml` | Config file references |

**Rules:**
- Minimum 4 trigger keywords, maximum 12
- Include at least one from each populated category
- Prefer phrases a developer would actually type when asking for help
- Include the bare tool name as the first keyword always
