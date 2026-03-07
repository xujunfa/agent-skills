# Dedup Strategy for experiences/

Efficient deduplication that scales to 50+ experience files without
reading all content. Two-phase approach: fast tag scan, then selective
semantic check.

---

## Phase 1: Tag Matching (Fast)

### What to Read

For each file in `experiences/`, read **lines 1-6 only** (the metadata header).
These lines contain:
```
# EXP-{NNN}: {title}

- **Tags:** {tags}
- **Tool version:** {version}
- **Platform:** {platform}
- **Date:** {date}
```

### Match Criteria

A file is a **candidate match** when ALL of:
- **2+ tags overlap** with the new experience
- **Same version** (or both "Unknown")
- **Same platform** (or both "Unknown")

### Example

New experience: `Tags: config, auth | Version: 3.2.1 | Platform: Linux`

```
exp-012-ssl-cert.md   → Tags: auth, deploy   | 3.2.1 | Linux → 1 tag overlap  → NO MATCH
exp-017-env-vars.md   → Tags: config, auth   | 3.2.1 | Linux → 2 tag overlap  → MATCH ✓
exp-023-api-rate.md   → Tags: API, config    | 3.1.0 | Linux → version differs → NO MATCH
```

### Fast Path Outcomes

- **Zero matches** → proceed directly to WRITE NEW (skip Phase 2).
- **1+ matches** → enter Phase 2 for each matched file.

---

## Phase 2: Semantic Judgment (Slow)

Only entered when Phase 1 produces matches. Read the **full content** of each
matched file.

### Decision Matrix

| Existing Problem | Existing Solution | New Experience | Decision |
|---|---|---|---|
| Same problem | Same solution | — | **SKIP** |
| Same problem | Different solution | — | **MERGE** (alternative) |
| Same problem | Same solution | More context/detail | **MERGE** (enrich) |
| Different problem | — | — | **WRITE NEW** |

### "Same problem" Heuristic

Two problems are "same" when they share:
- The same error message or symptom, OR
- The same root cause (even if symptoms differ slightly)

When uncertain, prefer WRITE NEW over SKIP (false negatives are worse than
mild duplication).

### Merge Format

When merging, append to the existing file using Edit:

```markdown
## Alternative Solution

> Added {YYYY-MM-DD} from conversation

{Steps for the alternative approach}
```

Or for additional context:

```markdown
## Additional Context

> Added {YYYY-MM-DD} from conversation

{New details, edge cases, or platform-specific notes}
```

Do NOT rewrite the original sections. Only append.

---

## Edge Cases

| Scenario | Handling |
|---|---|
| **3+ matches in Phase 1** | Run Phase 2 on all matches. If multiple are "same problem", merge into the most recent one only. |
| **Archived entries** (files in `experiences/archive/`) | Ignore during dedup — only scan top-level `experiences/`. |
| **Empty directory** (no existing files) | Skip dedup entirely → WRITE NEW. |
| **Malformed headers** (missing tags/version lines) | Treat as "no match" for that file — never fail the whole dedup. |
| **Conflicting merges** (two matches want different decisions) | Prefer MERGE over SKIP. If one says WRITE NEW and another says MERGE, do MERGE. |

---

## Token Budget

**Target: <500 tokens for 50 experience files.**

| Phase | Token Cost | Notes |
|---|---|---|
| Phase 1 (header scan) | ~200 tokens | 50 files x ~4 tokens per 6-line header |
| Phase 2 (semantic) | ~200 tokens | 1-3 matched files x ~80 tokens each |
| Decision output | ~50 tokens | Action + reasoning |
| **Total** | **~450 tokens** | Well under 500 budget |

### How to Stay in Budget

- **Never** read full files in Phase 1.
- **Cap** Phase 2 at 3 files max. If more than 3 matches, pick the 3 with
  highest tag overlap.
- Use Read tool with line ranges (lines 1-6) instead of reading whole files.
