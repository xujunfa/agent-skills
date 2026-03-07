---
name: create-expert-skill
description: >-
  Use when creating a new expert-level knowledge skill for an external tool,
  library, framework, or service. Searches official sources, distills knowledge,
  and generates a self-evolving expert skill.
  Triggers: "create expert skill", "build expert knowledge",
  "expert skill for XX", "become XX expert",
  "create knowledge base for XX", "XX expert knowledge",
  "XX expert skill", "for XX create expert".
---

# Create Expert Skill

## Overview

Create a self-evolving expert skill for any external tool/library/framework/service. Searches authoritative sources, distills knowledge, and generates a structured skill that improves over time via `evolve-expert-skill`.

NOT for internal codebase modules — use `create-module-skill` instead.

## Workflow

### Step 1 — Intent Clarification

Confirm (skip if user already provided):
1. **Tool name & nature** — exact name, type (CLI / SDK / framework / service)
2. **User level** — beginner (focus quick-start) or experienced (focus troubleshooting)
3. **Focus areas** — which aspects matter most

### Step 2 — Source Discovery

Execute 3-5 WebSearch calls. Read `references/source-discovery-guide.md` for the layered search strategy and quality filters.

### Step 3 — User Confirms Sources

Present discovered sources grouped by recommendation level. User can confirm, add (+URL), or exclude (-number). Read `references/source-discovery-guide.md` for the presentation format.

After confirmation, assess coverage against Step 1 focus areas. If gaps exist, offer `agent-research` for deeper research. See `references/source-discovery-guide.md` Phase B.

### Step 4 — Deep Fetch & Distill

For each confirmed source: WebFetch -> distill to <=200 lines -> write to `references/*.md` with source metadata header. Read `references/expert-skill-template.md` for the reference file format and adaptive partitioning rules.

### Step 5 — Forge Output

1. Generate `SKILL.md` using template from `references/expert-skill-template.md`
2. Generate `.sources.yml` using template from `references/sources-yml-template.md`
3. Auto-generate trigger keywords from distilled content
4. Run validation checklist from `references/quality-checklist.md`

Output creation summary with file counts and trigger keywords.

### Edge Cases

| Case | Action |
|------|--------|
| No search results | Allow manual URL input |
| WebFetch needs login | Skip, mark `inaccessible` in .sources.yml |
| Tool name ambiguity | Ask user to clarify in Step 1 |
| Target skill already exists | Ask: rebuild or switch to evolve-expert-skill? |

## Living Document

Iterate this skill via direct editing when workflow issues are found.
Current version: 1.0.0
