# Quality Checklist — Post-Creation Validation

Run this checklist after generating an expert skill to verify correctness.

---

## 1. Structure Checks

| # | Check | Pass criteria |
|---|---|---|
| S1 | SKILL.md exists | File present at `{tool}-expert/SKILL.md` |
| S2 | SKILL.md line count | `wc -l SKILL.md` <= 80 |
| S3 | Valid frontmatter | Has `---` delimiters, `name`, `description`, `trigger_keywords` |
| S4 | Name pattern | `name` field ends with `-expert` |
| S5 | .sources.yml exists | File present at `{tool}-expert/.sources.yml` |
| S6 | .sources.yml valid | Parseable YAML, has `version: 1`, `tool_name`, `sources` array |
| S7 | references/ populated | Directory contains >= 2 `.md` files |
| S8 | experiences/ exists | Directory present (may be empty at creation) |
| S9 | Reference file sizes | Every file in `references/` is <= 200 lines |

**Quick structural validation command:**
```bash
SKILL_DIR="{path_to_skill}"
echo "=== Structure ===" && \
test -f "$SKILL_DIR/SKILL.md" && echo "S1 PASS: SKILL.md exists" && \
LINE_CT=$(wc -l < "$SKILL_DIR/SKILL.md") && \
  [ "$LINE_CT" -le 80 ] && echo "S2 PASS: SKILL.md $LINE_CT lines" && \
test -f "$SKILL_DIR/.sources.yml" && echo "S5 PASS: .sources.yml exists" && \
REF_CT=$(ls "$SKILL_DIR/references/"*.md 2>/dev/null | wc -l) && \
  [ "$REF_CT" -ge 2 ] && echo "S7 PASS: $REF_CT reference files" && \
test -d "$SKILL_DIR/experiences" && echo "S8 PASS: experiences/ exists"
```

---

## 2. Content Checks

| # | Check | Pass criteria |
|---|---|---|
| C1 | Description format | `description` starts with "Use when" |
| C2 | Trigger keywords | `trigger_keywords` has 4-12 entries |
| C3 | Loading guide table | SKILL.md contains an "On-Demand Loading Guide" table |
| C4 | Table-file alignment | Every file in the loading guide table exists in `references/` |
| C5 | Metadata headers | Each reference file has Source, Fetched, Version anchor fields |
| C6 | Actionable knowledge | Spot-check: pick 3 random bullets — each enables a decision or action |
| C7 | Source mapping | Every `mapped_to` in .sources.yml points to an existing reference file |
| C8 | No raw copy-paste | Reference content is distilled, not verbatim docs |

**Content spot-check command:**
```bash
SKILL_DIR="{path_to_skill}"
echo "=== Content ===" && \
grep -c "Use when" "$SKILL_DIR/SKILL.md" && \
grep -c "trigger_keywords" "$SKILL_DIR/SKILL.md" && \
grep -c "On-Demand Loading Guide" "$SKILL_DIR/SKILL.md" && \
for f in "$SKILL_DIR/references/"*.md; do
  echo "--- $(basename $f) ---"
  head -6 "$f" | grep -E "(Source|Fetched|Version)"
done
```

---

## 3. Consistency Checks

| # | Check | Pass criteria |
|---|---|---|
| X1 | tool_name match | `.sources.yml` `tool_name` == SKILL.md `name` minus `-expert` |
| X2 | Source count | Number of `sources` entries in .sources.yml >= number of reference files |
| X3 | URL format | All `url` fields start with `https://` |
| X4 | Version anchors | `version_anchor` in .sources.yml aligns with SKILL.md tool version |
| X5 | Maintenance section | SKILL.md ends with Maintenance section containing version + date |
| X6 | Date consistency | `.sources.yml` `created` matches SKILL.md creation date |

---

## 4. Output Template

On successful validation, print:

```
Expert skill created successfully.

  Skill:      {tool_name}-expert
  Location:   {absolute_path}
  Files:      SKILL.md, .sources.yml, {N} reference files, experiences/
  Ref files:  {list of reference filenames}
  Triggers:   {comma-separated trigger keywords}
  Sources:    {N} sources registered

All {total_checks} checks passed.
```

On failure, list failing checks:

```
Expert skill validation FAILED.

  FAILED checks:
    {check_id}: {description} — {actual_value}
    ...

Fix the above issues before delivering the skill.
```

---

## Automated Full Check

Combine all checks into a single pass:

```bash
SKILL_DIR="{path_to_skill}"
PASS=0; FAIL=0
run_check() {
  if eval "$2"; then
    echo "PASS $1"; PASS=$((PASS+1))
  else
    echo "FAIL $1: $3"; FAIL=$((FAIL+1))
  fi
}
run_check "S1" "test -f '$SKILL_DIR/SKILL.md'" "SKILL.md missing"
run_check "S2" "[ $(wc -l < '$SKILL_DIR/SKILL.md') -le 80 ]" "SKILL.md too long"
run_check "S5" "test -f '$SKILL_DIR/.sources.yml'" ".sources.yml missing"
run_check "S7" "[ $(ls '$SKILL_DIR/references/'*.md 2>/dev/null | wc -l) -ge 2 ]" "<2 refs"
run_check "S8" "test -d '$SKILL_DIR/experiences'" "experiences/ missing"
echo "---"
echo "Passed: $PASS  Failed: $FAIL"
```
