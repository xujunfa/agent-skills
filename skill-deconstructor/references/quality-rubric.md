# Quality Assessment Rubric

Score each dimension 1-5 against Agent Skill best practices.

## Dimensions

### 1. Description Quality (max 5)

| Score | Criteria |
|-------|----------|
| 5 | "Use when..." format; triggers only (no workflow); rich keywords; <1024 chars |
| 4 | "Use when..." format; mostly triggers; minor workflow leakage |
| 3 | Has description but wrong format; OR contains workflow summary |
| 2 | Vague; few keywords; poor discoverability |
| 1 | Missing or empty description |

### 2. Token Efficiency (max 5)

| Score | Criteria |
|-------|----------|
| 5 | SKILL.md <500 words; heavy content in reference files; zero redundancy |
| 4 | 500-800 words; some extractable content; minimal redundancy |
| 3 | 800-1200 words; moderate redundancy |
| 2 | >1200 words; significant redundancy; no reference files despite complexity |
| 1 | Extremely verbose; extensive duplication |

### 3. Discoverability (max 5)

| Score | Criteria |
|-------|----------|
| 5 | Semantic name (verb-first or clear noun); symptom keywords in description; multiple trigger paths |
| 4 | Good name; decent keywords; most use cases covered |
| 3 | Acceptable name; some keywords; primary use case discoverable |
| 2 | Generic name; few keywords; easy to miss |
| 1 | Cryptic name or special chars; unhelpful description |

### 4. Structural Completeness (max 5)

Target sections: YAML frontmatter, Overview, When to Use, Core Steps/Pattern, Quick Reference, Common Mistakes

| Score | Criteria |
|-------|----------|
| 5 | All 6 target sections present |
| 4 | 5 of 6 sections |
| 3 | Frontmatter + Overview + Core Steps (minimum viable) |
| 2 | Missing frontmatter OR Overview |
| 1 | Unstructured; multiple critical sections missing |

### 5. Defensiveness (max 5)

Defensive elements: Red Flags list, Rationalization table, Error handling, Explicit exclusions

| Score | Criteria |
|-------|----------|
| 5 | All 4 defensive elements |
| 4 | 3 of 4 elements |
| 3 | Error handling + 1 other element |
| 2 | Minimal error handling only |
| 1 | No defensive elements |

**Type adjustment — non-Discipline skills are scored leniently on this dimension:**
- **Technique / Reference:** Baseline 3 if has basic error handling. No penalty for missing rationalization defense.
- **Pattern:** Baseline 3 if has "When NOT to use". No penalty for missing Red Flags.
- **Discipline:** Full scoring — defensiveness is critical for enforcement skills.

## Total Score

| Range | Rating | Meaning |
|-------|--------|---------|
| 21-25 | Excellent | Follows best practices rigorously |
| 16-20 | Good | Solid skill with room for improvement |
| 11-15 | Adequate | Functional but missing key quality signals |
| 6-10 | Needs Work | Significant gaps |
| 1-5 | Poor | Major issues |

## Output Format

Per dimension:
1. **Score** (N/5)
2. **Justification** — one sentence
3. **Suggestion** — specific improvement action (only if score <5)

Final output: total score, rating label, up to 3 top improvement suggestions.
