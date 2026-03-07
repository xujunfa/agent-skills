# Git Worktree Isolation, Dispatch & Merge

> **Sources:** Claude Code docs, Codex CLI docs, workmux, IttyBitty, community gists
> **Fetched:** 2026-03-07
> **Version anchor:** git 2.x / Claude Code 2.1.x / Codex 2026.2

## Why Worktrees Are Mandatory

- Shared working directory = file clobbering between agents
- Each agent MUST have its own isolated git worktree
- Worktrees share `.git` but have independent working trees
- Branches merge back via normal git merge/PR workflow

## Claude Code Native Worktree (Recommended)

```bash
claude --worktree --tmux                    # auto worktree + tmux session
claude --worktree feature-auth --tmux       # named worktree
```

**Specific behavior (verified):**
- **Path:** `.claude/worktrees/<name>/` (under repo root)
- **Branch:** `worktree-<name>` (from default remote branch)
- **Cleanup:** auto-removed on normal exit if no changes; manual `git worktree remove` on crash
- **In tmux:** creates new session (not window), even if already in tmux
- **No base branch flag:** for custom base, use manual `git worktree add` first
- Add `.claude/worktrees/` to `.gitignore`

### Custom Base Branch Workaround

```bash
git worktree add ../my-feature -b feature-auth develop  # custom base
cd ../my-feature && claude --tmux                        # run Claude in it
```

## Codex in Worktree

- `--full-auto` sandbox **auto-limits to cwd** (macOS Seatbelt / Linux Landlock)
- Codex correctly senses it's in a worktree subdirectory
- `codex exec "prompt" --json` outputs reliable NDJSON event stream
- `--skip-git-repo-check` available for non-git directories

```bash
cd .claude/worktrees/my-task
codex exec "implement feature X" --full-auto --json > output.ndjson
```

## Parallel Agent Dispatch Script

```bash
#!/bin/bash
# dispatch-team.sh — Spawn N agents with isolated worktrees
# Usage: ./dispatch-team.sh /path/to/project prompts.txt
PROJECT="$1"; PROMPTS_FILE="$2"
SESSION="$(basename $PROJECT)-team"

tmux new-session -d -s "$SESSION" -n "control"
AGENT_NUM=0

while IFS= read -r PROMPT; do
  AGENT_NUM=$((AGENT_NUM + 1))
  BRANCH="agent-$AGENT_NUM"
  WT_DIR="$PROJECT-$BRANCH"
  git -C "$PROJECT" worktree add -b "$BRANCH" "$WT_DIR" HEAD
  tmux new-window -t "$SESSION" -n "agent-$AGENT_NUM" -c "$WT_DIR"
  ESCAPED=$(printf '%q' "$PROMPT")
  tmux send-keys -t "$SESSION:agent-$AGENT_NUM" "claude -p $ESCAPED" Enter
done < "$PROMPTS_FILE"

tmux select-layout -t "$SESSION" tiled
echo "Attach: tmux attach -t $SESSION"
```

## Merge Workflow (Critical Post-Completion Step)

**Recommended: Orchestrator sequential merge** (one branch at a time, avoids compound conflicts).

```bash
# 1. Check all worktree branches
git worktree list
workmux status       # or: ib diff <agent-id>

# 2. Merge sequentially (independent tasks first, dependent later)
git checkout main
git merge worktree-feature-auth        # first agent
git merge worktree-api-endpoints       # second agent (may conflict on shared files)

# 3. If conflict on shared files (package.json, config, etc.)
git merge worktree-package-update
# Edit conflicted file → git add → git commit
# Or let orchestrator fix: claude "resolve merge conflict in package.json"

# 4. Cleanup
git worktree remove .claude/worktrees/feature-auth
git branch -d worktree-feature-auth
git worktree prune
```

**With tools:**
- workmux: `workmux merge feature-auth` (auto rebase + cleanup)
- IttyBitty: `ib merge <agent-id>` (merge + archive logs)

**Minimize conflicts by design:** assign non-overlapping file domains per agent.

## Reference Scripts: tmx-claude / tmx-worktree

> **Note:** These are community reference implementations from a [GitHub Gist](https://gist.github.com/andynu/13e362f7a5e69a9f083e7bca9f83f60a). Not pre-installed — copy scripts manually or use workmux/IttyBitty instead.

- `tmx-claude -d ~/projects/myapp "Fix flaky test"` — dispatch Claude to dir (no isolation)
- `tmx-worktree -b feature-auth "Implement OAuth"` — create worktree + branch + tmux + Claude

## Multi-Agent Safety Rules

- NEVER `git stash` — other agents may be working
- NEVER switch branches in a shared worktree
- Scope commits to YOUR changes only
- Merge worktrees **sequentially**, not simultaneously
- Commit frequently — unpushed worktree commits are lost on crash
