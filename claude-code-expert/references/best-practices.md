# Best Practices & Workflow Patterns

> **Source:** https://code.claude.com/docs/en/best-practices + https://shipyard.build/blog/claude-code-cheat-sheet/
> **Fetched:** 2026-03-06
> **Version anchor:** v2.1.63

## #1 Rule: Context Window is Your Most Precious Resource

- Performance degrades as context fills; track usage via custom status line
- `/clear` between unrelated tasks — the single most impactful habit
- After 2 failed corrections: `/clear` and write a better initial prompt
- Auto-compaction triggers at ~95% capacity; customize via `/compact <instructions>`
- Use `Esc+Esc` > "Summarize from here" for partial compaction

## Verification: Highest-Leverage Practice

- **Always** give Claude a way to verify its work (tests, screenshots, expected output)
- Without verification criteria, you become the only feedback loop
- Use Chrome extension for UI verification loops
- "Write tests first, then implement" outperforms "implement, then test"
- For visual changes: paste screenshot + ask Claude to compare after implementing

## The 4-Phase Workflow

1. **Explore** (Plan Mode) — Read files, understand code, no changes
2. **Plan** (Plan Mode) — Create detailed implementation plan; `Ctrl+G` to edit plan
3. **Implement** (Normal Mode) — Code against the plan; verify with tests
4. **Commit** — `commit with a descriptive message and open a PR`

Skip planning for small/obvious tasks. Plan when uncertain, multi-file, or unfamiliar.

## Prompt Engineering (CIF Pattern)

- **Context**: Reference specific files with `@`, paste images, pipe data
- **Intent**: State exactly what you want, not vague goals
- **Format**: Specify output format, test expectations, constraints

| Bad | Good |
|---|---|
| "add tests for foo.py" | "write a test for foo.py covering the edge case where user is logged out. avoid mocks." |
| "fix the login bug" | "users report login fails after session timeout. check auth flow in src/auth/, especially token refresh. write a failing test, then fix it" |
| "make the dashboard look better" | "[paste screenshot] implement this design. take a screenshot and compare. list differences and fix them" |

## CLAUDE.md Configuration

- Run `/init` to generate starter, then refine over time
- Sweet spot: 50-200 lines per file; beyond that, prune aggressively
- For each line ask: "Would removing this cause Claude to make mistakes?" If not, cut it
- Use emphasis ("IMPORTANT", "YOU MUST") to improve adherence
- Check into git for team sharing; compounds in value over time
- Use `@path/to/import` syntax to include other files
- Monorepos: place at multiple levels (root + subdirectories)
- Skills > CLAUDE.md for domain knowledge loaded only when relevant

### CLAUDE.md Placement
- `~/.claude/CLAUDE.md` — Global, all projects
- `./CLAUDE.md` — Project root, git-tracked team-shared
- `./CLAUDE.local.md` — Project root, gitignored personal
- Parent/child directories — Auto-loaded hierarchically in monorepos

### What to Include vs Exclude
| Include | Exclude |
|---|---|
| Bash commands Claude can't guess | Things Claude can figure out from code |
| Code style rules differing from defaults | Standard language conventions |
| Test instructions and runners | Detailed API docs (link instead) |
| Repo etiquette (branch naming, PR rules) | Info that changes frequently |
| Architectural decisions | File-by-file codebase descriptions |
| Common gotchas | Self-evident practices |

## Context-Saving Techniques

- **Subagents** for investigation — explore in separate context, return summary
- **Scope investigations narrowly** or use subagents to prevent context bloat
- **Use `/rename`** for descriptive session names (e.g., "oauth-migration")
- **Handoff files**: Before ending long session, ask Claude to write HANDOFF.md
- **Resume** with `claude -c` (most recent) or `claude -r` (pick session)

## Scaling & Automation

- `claude -p "prompt"` for CI, pre-commit hooks, scripts
- `--output-format json` for programmatic parsing
- Fan out: generate task list, loop `claude -p` per file with `--allowedTools`
- Writer/Reviewer pattern: Session A implements, Session B reviews fresh
- Agent teams for sustained parallelism across sessions

## Common Failure Patterns to Avoid

1. **Kitchen sink session** — mixing unrelated tasks. Fix: `/clear` between tasks
2. **Correction spiral** — 3+ corrections in a row. Fix: `/clear`, write better prompt
3. **Over-specified CLAUDE.md** — too long, rules get lost. Fix: prune ruthlessly
4. **Trust-then-verify gap** — no verification criteria. Fix: always provide tests/checks
5. **Infinite exploration** — unscoped "investigate". Fix: scope narrowly or use subagents

## Quick Wins

- Use `gh` CLI for GitHub operations (rate limits on unauthenticated API)
- Install code intelligence plugins for typed languages
- Let Claude interview you for large features: "Interview me using AskUserQuestion tool"
- Treat sessions like branches — different workstreams get separate contexts
- Checkpoints persist across sessions; rewind works even after restart
