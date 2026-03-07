# Seven-Dimension Analysis Framework

Detailed rules for analyzing a Skill across 7 dimensions. Each dimension produces structured data consumed by the deliverable generators.

## D1: Quick Take

| Field | Source | Output Format |
|-------|--------|---------------|
| Name | Frontmatter `name` | Exact string |
| Summary | Synthesize overview + description | One natural-language sentence |
| Type | Content-based classification | Technique / Discipline / Pattern / Reference (or compound) |
| Audience | Infer from triggers + use cases | "For [who] working on [what]" |

**Type classification signals:**

| Type | Primary Signals |
|------|----------------|
| Technique | Step 1/2/3 structure, CLI commands, tool invocations, concrete operations |
| Discipline | "MUST"/"NEVER"/"REQUIRED", Red Flags tables, rationalization defense |
| Pattern | "When to Use", "Core Pattern", Before/After comparisons, mental models |
| Reference | Dense tables, API listings, configuration catalogs, searchable indexes |

If signals overlap, use compound label (e.g., "Technique + Reference"). Primary type = stronger signals.

## D2: Positioning & Triggers

**Extract these fields:**

1. **Problem statement** — From overview or first paragraph.
   Format: "Solves [problem] for [users] by [approach]."

2. **Trigger conditions** — From frontmatter `description` + "When to Use" section.
   Extract as discrete tags (keyword or short phrase, 2-5 words max).
   - Include both English and non-English trigger terms if present
   - Include symptom keywords (error messages, states, user actions)
   - Aim for 5-15 tags

3. **Exclusions** — From "When NOT to use" section (if present).
   Extract as negative tags prefixed with "Not for:".

## D3: Architecture Overview

1. Scan the full directory tree. Exclude: `node_modules/`, `.git/`, `venv/`, `__pycache__/`, `dist/`, `build/`, `*.lock`
2. For each file, read the first 20 lines to infer its role
3. Mark `SKILL.md` as `[PRIMARY]`
4. Annotate each file with a one-line role

**Role inference heuristics:**
- `references/*.md` → "Reference: [topic from filename/content]"
- `scripts/*` → "Script: [action from docstring or filename]"
- `*.test.*` or `tests/` → "Test: [what it tests]"
- `README.md` (not SKILL.md) → "Project documentation"
- `.yaml`/`.json`/`package.json` → "Config: [purpose]"
- Other `.md` → "Documentation: [topic]"

## D4: Core Workflow

**Extraction process:**
1. Find the main step sequence in SKILL.md: look for `Step N`, `Phase N`, `### ` headers with sequential actions, or numbered lists with actions
2. Per step: extract a short label + one-line description
3. Identify decision points (if/else, conditions, validations) → diamond nodes
4. Identify error/fallback branches → labeled edges
5. Identify loops/retries → back-edges

**Mermaid rules:**
- Use `flowchart TD` (top-down)
- Max 10-12 nodes; summarize if workflow has more steps
- `[]` for action steps, `{}` for decisions, `([])` for start/end
- Label conditional edges with `|Yes|` / `|No|` or descriptive text
- Keep node labels short (3-6 words)

**Example:**
```
flowchart TD
    A([Start]) --> B[Acquire source]
    B --> C{SKILL.md exists?}
    C -->|No| D[Error: not valid]
    C -->|Yes| E[Analyze 7 dimensions]
    E --> F[Generate deliverables]
    F --> G([Deliver & open])
```

## D5: Design Highlights

**Scan for these patterns (in priority order):**

| Pattern | Detection Signals |
|---------|------------------|
| Failsafe / Validation | Input checks, `if not exists`, retry logic, fallback chains |
| Rationalization Defense | "Red Flags" table, "Excuse vs Reality" table, warning lists |
| Graceful Degradation | Multi-level fallback, optional dependencies, "if available" logic |
| Progressive Disclosure | Layered content (overview → reference files), conditional reading |
| Cross-Skill Composition | References to other skills, sub-skill invocation patterns |
| Innovative Interaction | Unique UX patterns, creative approaches not seen in standard skills |

**Per highlight, output:**
- **Title**: 2-4 word label
- **Explanation**: One sentence on why this is notable
- **Evidence**: Direct quote from the skill (keep under 50 words)

If no highlights found, state: "Straightforward skill with no notable design patterns."

## D6: Dependencies & Integrations

| Category | What to Scan For |
|----------|-----------------|
| External tools | CLI commands: `python3`, `npm`, `git`, `playwright`, `docker`, etc. |
| Skill references | `REQUIRED SUB-SKILL`, `@skills/`, skill names in instructions |
| APIs | HTTP URLs, endpoint patterns, `fetch`/`curl`/`WebFetch` usage |
| System requirements | OS-specific instructions, runtime versions, config paths |
| Config files | Referenced paths like `~/.config/...`, environment variables |

Group by category. For each item, note what it's used for in 5-10 words.

## D7: Quality Assessment

Apply scoring from `references/quality-rubric.md`. Produce:
- Five individual scores (N/5) with one-sentence justifications
- Total score (N/25)
- Up to 3 specific improvement suggestions (for dimensions scoring <5)
