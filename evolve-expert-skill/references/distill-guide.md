# Distill Mode — Full Workflow

Distill captures troubleshooting/usage experience from the current conversation
into `experiences/` and updates SKILL.md metadata.

---

## D1 — Experience Extraction

Scan the conversation for a resolved problem. Fill each field:

| Field | Where to Find |
|---|---|
| Problem | User's initial complaint / error message |
| Root cause | Assistant's diagnosis or confirmed cause |
| Solution | Steps that resolved the issue |
| Verification | How success was confirmed (test, output, etc.) |
| Tags | Classify from standard list below |
| Tool version | Version mentioned in conversation or SKILL.md |
| Platform | OS / runtime / environment mentioned |

Mark any unknown field as `"Unknown"`.

### Standard Tag List

```
install, config, performance, API, deploy, migration,
upgrade, compatibility, auth, data, CLI, build, test, runtime
```

Assign 1-3 tags. Prefer specific over generic.

---

## D2 — Dedup Check

Goal: avoid duplicate experiences with minimal token usage.

### Phase 1: Fast Path (Tag Matching)

1. List all files in `experiences/`.
2. For each file, read ONLY lines 1-6 (metadata header).
3. Check for exact match on: **(tags + version + platform)**.
4. If no match found → skip Phase 2 → proceed to D3 (WRITE NEW).
5. If match found → enter Phase 2.

**Key rule: NEVER read all experience files fully.**

### Phase 2: Slow Path (Semantic Fallback)

Only entered when Phase 1 finds a tag match. Read the matched file(s) fully.

| Existing Entry | New Experience | Decision |
|---|---|---|
| Same problem, same solution | — | **SKIP** (inform user) |
| Same problem, different solution | — | **MERGE** (append alternative) |
| Same problem, more context | — | **MERGE** (enrich existing) |
| Different problem | — | **WRITE NEW** |

When merging, use Edit to append an `## Alternative Solution` or
`## Additional Context` section to the existing file.

---

## D3 — Write Experience File

### File Naming

```
exp-{NNN}-{slug}.md
```

- `{NNN}` = zero-padded 3-digit sequential number (check existing files for next).
- `{slug}` = 2-4 word kebab-case summary (e.g., `ssl-cert-expired`).

### Entry Format

```markdown
# EXP-{NNN}: {Short title}

- **Tags:** {tag1}, {tag2}
- **Tool version:** {version}
- **Platform:** {platform}
- **Date:** {YYYY-MM-DD}
- **Source:** conversation

## Problem
{1-3 sentences describing the symptom}

## Root Cause
{1-3 sentences explaining why it happened}

## Solution
{Numbered steps or code snippet that fixed it}

## Verification
{How the fix was confirmed}
```

### Quality Bar

- Every field must be filled (use "Unknown" if necessary).
- Solution must be actionable — no vague advice.
- Keep total file under 60 lines.

---

## D4 — Trigger Evolution

After writing the experience:

1. Extract 2-5 new keywords from the experience.
2. Read SKILL.md `description` field.
3. Compare keywords — if any are genuinely new, append them.
4. Keep description under 1024 characters.
5. Update SKILL.md version: **patch bump** (e.g., 1.0.0 → 1.0.1).
6. Increment experience count in SKILL.md.

---

## Output Format

```
## Distill Complete

- **Action:** {WRITE NEW | MERGE | SKIP}
- **File:** experiences/{filename}
- **Tags:** {tags}
- **Version bumped:** {old} → {new}
- **Experience count:** {N}
- **Keywords added:** {list or "none"}
```
