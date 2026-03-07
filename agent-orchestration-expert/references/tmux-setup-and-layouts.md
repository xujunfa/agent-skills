# Tmux Setup & Layouts for Agent Teams

> **Sources:** tmux-agents skill (ClawHub), tmuxinator, workmux, IttyBitty
> **Fetched:** 2026-03-07
> **Version anchor:** tmux 3.x / tmuxinator latest / workmux 2026.3

## Quick Start Options (Pick One)

### Option A: workmux (Recommended for Claude + Codex mix)

```bash
cargo install workmux  # or brew install workmux

# workmux.yaml in project root
cat > workmux.yaml << 'EOF'
agent: claude
panes:
  - command: claude --worktree frontend --tmux
  - command: claude --worktree backend --tmux
  - command: codex --full-auto
EOF

workmux new my-feature  # creates worktrees + tmux panes
workmux status          # check all agents
workmux merge           # sequential merge + cleanup
```

### Option B: IttyBitty (Claude-only, best dashboard)

```bash
git clone https://github.com/adamwulf/ittybitty && export PATH="$PWD/ittybitty:$PATH"
ib new-agent --model opus "build auth system"     # auto worktree + tmux
ib new-agent --model sonnet "write unit tests"
ib watch                                          # TUI dashboard
ib merge <agent-id>                               # merge + archive
```

Limit: Claude only, max ~10 agents/repo. Supports `ib send` inter-agent messaging.

### Option C: OpenClaw tmux-agents Skill

```bash
clawhub install tmux-agents  # install the skill

# Called via scripts (NOT `openclaw skill run`):
~/.openclaw/skills/tmux-agents/scripts/spawn.sh "claude-coder" "fix login bug" "claude-sonnet"
~/.openclaw/skills/tmux-agents/scripts/status.sh
tmux attach -t claude-coder
```

No YAML config — pure script-driven. Independent of tmuxinator.

### Option D: Tmuxinator (Full custom control)

```yaml
# ~/.config/tmuxinator/agent-team.yml
name: agent-team
root: ~/projects/my-app

on_project_start:
  - git worktree add -b feat-frontend ../my-app-frontend HEAD
  - git worktree add -b feat-backend ../my-app-backend HEAD

on_project_exit:
  - git worktree remove ../my-app-frontend --force
  - git worktree remove ../my-app-backend --force

windows:
  - orchestrator:
      layout: main-vertical
      panes:
        - claude --model opus
        - # Status monitor (empty for user)
  - frontend:
      root: ../my-app-frontend
      panes:
        - claude --model sonnet "Implement React components"
  - backend:
      root: ../my-app-backend
      panes:
        - codex --full-auto "Build API endpoints"
```

Start: `tmuxinator start agent-team` | Stop: `tmuxinator stop agent-team`

## Recommended Layouts

| Agent Count | Layout | Rationale |
|---|---|---|
| 1-2 | Single window, vertical split | Simple, full visibility |
| 3-4 | `tiled` | Equal visibility for all agents |
| 5-6 | `main-vertical` | Orchestrator gets large left pane |
| 7+ | `main-vertical` + zoom | Zoom: `prefix + z` |

### Layout ASCII Examples

**tiled (4 agents):**
```
┌──────────┬──────────┐
│ orchestr │ claude-1 │
├──────────┼──────────┤
│ codex-1  │ codex-2  │
└──────────┴──────────┘
```

**main-vertical (5 agents):**
```
┌────────────┬─────────┐
│            │claude-1 │
│ orchestr   ├─────────┤
│ (60% w)    │claude-2 │
│            ├─────────┤
│            │codex-1  │
│            ├─────────┤
│            │codex-2  │
└────────────┴─────────┘
```

## Manual Tmux Setup (No tools)

```bash
#!/bin/bash
SESSION="dev-team"
PROJECT="$HOME/projects/my-app"  # ← Replace with your path

tmux new-session -d -s "$SESSION" -n orchestrator -c "$PROJECT"
tmux send-keys -t "$SESSION:orchestrator" "claude --model opus" Enter

for i in 1 2; do
  WT_DIR="$PROJECT-worker-$i"
  git -C "$PROJECT" worktree add -b "feat-worker-$i" "$WT_DIR" HEAD
  tmux new-window -t "$SESSION" -n "claude-$i" -c "$WT_DIR"
  tmux send-keys -t "$SESSION:claude-$i" "claude" Enter
done

tmux select-layout -t "$SESSION" tiled
echo "Attach: tmux attach -t $SESSION"
```

## Session Naming Convention

- Session: `{project}-team` (e.g., `nextjs-saas-team`)
- Windows: `orchestrator`, `claude-{role}`, `codex-{n}`, `tester`, `monitor`

## Key Tmux Commands

| Action | Command |
|---|---|
| Attach | `tmux attach -t {session}` |
| List | `tmux ls` |
| Switch window | `prefix + n` / `prefix + p` |
| Zoom pane | `prefix + z` |
| Send command | `tmux send-keys -t {session}:{window} "cmd" Enter` |
| Capture output | `tmux capture-pane -t {target} -p` |
