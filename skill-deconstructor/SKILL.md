---
name: skill-deconstructor
description: >-
  Use when analyzing, understanding, or reviewing an Agent Skill from any source
  (GitHub URL, local path, file URL). Triggers: "deconstruct skill", "analyze
  skill", "skill review", "skill breakdown", "what does this skill do", "skill
  architecture", "skill quality", "skill card", "skill visualization",
  "understand this skill", "evaluate skill".
---

# Skill Deconstructor

Systematically deconstruct any Agent Skill into a structured report and visual HTML card. Input a Skill source, output a 7-dimension analysis with quality scoring.

## When to Use

- Found an interesting Skill on GitHub and want to understand it quickly
- Need to evaluate a Skill's quality before installing
- Want to create a visual "card" to showcase or share a Skill
- Reviewing your own Skill's structure and completeness

## Workflow

### Step 1: Acquire Source

Determine input type and fetch the Skill:

| Input | Detection | Action |
|-------|-----------|--------|
| GitHub URL | Contains `github.com` | `git clone --depth 1 <url> /tmp/skill-deconstructor/<name>` |
| Local path | Filesystem path | Resolve to absolute path, use directly |
| File URL | Other HTTP(S) URL | WebFetch to `/tmp/` |

**Validate:** Confirm `SKILL.md` exists in the acquired directory. If not found, stop with error: "No SKILL.md found — this is not a valid Agent Skill."

**Read strategy:**
- Read: `SKILL.md`, `*.md`, `*.yaml`, `*.yml`, `*.py`, `*.js`, `*.ts`, `*.sh`, `*.csv`
- Ignore: `node_modules/`, `.git/`, `venv/`, `__pycache__/`, `*.lock`, binaries
- Scan full directory tree for architecture overview
- For files >500 lines: read first 100 + last 20 lines only

### Step 2: Seven-Dimension Analysis

**Read `references/analysis-framework.md`** for detailed analysis rules per dimension.

Analyze the Skill in order:

1. **Quick Take** — Name, one-line summary, type classification (Technique/Discipline/Pattern/Reference), target audience
2. **Positioning & Triggers** — Problem solved, trigger conditions as tags, exclusions
3. **Architecture Overview** — File tree with per-file role annotations
4. **Core Workflow** — Extract main step sequence into Mermaid `flowchart TD`
5. **Design Highlights** — Identify notable patterns (failsafes, rationalization defense, graceful degradation, progressive disclosure)
6. **Dependencies & Integrations** — External tools, skill references, APIs, system requirements
7. **Quality Assessment** — Score against 5 criteria per `references/quality-rubric.md`

### Step 3: Detect Auxiliary Skills

Before generating HTML, check for available design-enhancing skills using a 3-level fallback:

**UI Design Enhancement:**
1. If `ui-ux-pro-max` is in available skills → invoke with `--design-system` using keywords: `"documentation skill card visualization minimal clean professional light"`
2. Else search available skills for names containing "ui" / "design" / "frontend"
3. Else use built-in styles from `references/html-template.html`

**Diagram Enhancement:**
1. If `mermaid-diagrams` is in available skills → invoke for flowchart best practices
2. Else use basic Mermaid `flowchart TD` syntax directly

**Principle:** Auxiliary skills raise the quality ceiling but never lower the functional floor. Without any auxiliary skills, output is still complete, accurate, and well-styled.

### Step 4: Generate Deliverables

**Language:** All generated content defaults to **Chinese (Simplified)**. Professional terms, tool names, file names, code identifiers keep their original English. Follow strict Chinese-English typography: add a space between Chinese and Latin characters/numbers (e.g., `支持 Mermaid 图表` not `支持Mermaid图表`).

Generate two files from the Step 2 analysis:

**README.md:**
- Seven dimensions as H2 sections (section titles in Chinese)
- Mermaid diagrams in fenced code blocks
- Quality scores as a table

**index.html:**
- **Read `references/html-template.html`** for the complete HTML/CSS structure and layout
- Self-contained single file: inline CSS + Mermaid.js via CDN
- Set type-based accent color via CSS variables:

| Type | `--accent` | `--accent-light` |
|------|-----------|-----------------|
| Technique | `#2563EB` | `#EFF6FF` |
| Discipline | `#EA580C` | `#FFF7ED` |
| Pattern | `#7C3AED` | `#F5F3FF` |
| Reference | `#059669` | `#ECFDF5` |

- Replace all placeholder content in the template with actual analysis results
- Set Mermaid `themeVariables` colors to match the accent (use actual hex values, not CSS vars)

### Step 5: Deliver

```bash
mkdir -p skill-deconstructions/<skill-name>
```

Write both files to `skill-deconstructions/<skill-name>/`.

Open the HTML in the default browser:
```bash
open skill-deconstructions/<skill-name>/index.html   # macOS
xdg-open skill-deconstructions/<skill-name>/index.html  # Linux
```

Tell the user: output directory, total quality score (N/25), and top improvement suggestion.

## Error Handling

| Scenario | Action |
|----------|--------|
| Clone fails | Prompt: check URL validity and network connection |
| No SKILL.md | Error: "Not a valid Agent Skill" — stop |
| No YAML frontmatter | Note it, deduct in quality score, continue analysis |
| Mermaid CDN unreachable | Flowchart area shows raw Mermaid source as readable text |
| File >500 lines | Read first 100 + last 20 lines, mark middle as "[...]" |
