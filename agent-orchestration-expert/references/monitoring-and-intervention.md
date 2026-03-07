# Monitoring & Intervention

> **Sources:** OpenClaw CLI docs (docs.openclaw.ai/cli), tmux, workmux, IttyBitty
> **Fetched:** 2026-03-07
> **Version anchor:** OpenClaw v2026.3.x / tmux 3.x

## Attach & Observe

```bash
tmux attach -t agent-team              # attach to team session
tmux attach -t agent-team -r           # read-only (safe observation)
tmux select-window -t agent-team:orchestrator  # jump to specific window
```

## Tool-Specific Dashboards

```bash
# IttyBitty — real-time TUI dashboard
ib watch

# workmux — status overview
workmux status

# OpenClaw — Web UI Mission Control
openclaw dashboard
```

## OpenClaw Agent Management (Verified Commands)

```bash
openclaw dashboard                           # Web UI — monitor all agents
openclaw agents list                         # list configured agents (NOT "agents status")
openclaw agents bindings --agent coder       # view routing for specific agent
openclaw logs                                # view gateway + agent logs

# To send message to specific agent: use bound channel (Telegram/Discord/etc)
# or inject via Dashboard web UI. No `openclaw agents send` command exists.

# To read agent session logs directly:
cat ~/.openclaw/agents/<agent-id>/sessions/*.jsonl | tail -20
```

## Tmux Status Line for Agent Monitoring

```bash
# ~/.tmux.conf — agent-aware status bar
set -g status-interval 5
set -g status-right '#{continuum_status} | Agents: #(tmux list-windows | wc -l | tr -d " ") | %H:%M'
set -g pane-active-border-style fg=green,bold
```

## Log Tailing & Output Capture

```bash
# Capture current pane content to file
tmux capture-pane -t agent-team:claude-1 -p > /tmp/claude-1-output.txt

# Capture with history (last 5000 lines)
tmux capture-pane -t agent-team:claude-1 -p -S -5000 > /tmp/claude-1-full.txt

# Live tail a specific agent's pane to file
tmux pipe-pane -t agent-team:codex-1 -o 'cat >> /tmp/codex-1.log'
tmux pipe-pane -t agent-team:codex-1      # stop logging
```

## Intervention Patterns

### Send Input to Agent

```bash
tmux send-keys -t agent-team:claude-1 "Focus on auth module first" Enter
tmux send-keys -t agent-team:codex-1 C-c   # stop agent
tmux send-keys -t agent-team:claude-1 Escape
```

### Restart a Single Agent

```bash
tmux send-keys -t agent-team:claude-1 C-c
sleep 2
tmux send-keys -t agent-team:claude-1 "claude --model opus 'Continue from where you left off'" Enter
```

### Add a New Agent Mid-Session

```bash
git worktree add -b hotfix-123 ../myapp-hotfix HEAD
tmux new-window -t agent-team -n "hotfix" -c ../myapp-hotfix
tmux send-keys -t agent-team:hotfix "claude 'Fix issue #123'" Enter
```

### Remove an Agent

```bash
tmux send-keys -t agent-team:codex-2 "exit" Enter  # graceful
tmux kill-window -t agent-team:codex-2              # force
git worktree remove ../myapp-codex-2                # cleanup
```

## Broadcast to All Agents

```bash
for WINDOW in $(tmux list-windows -t agent-team -F '#{window_name}'); do
  tmux send-keys -t "agent-team:$WINDOW" "Please commit your current progress" Enter
done
```

## Progress Dashboard Script

```bash
#!/bin/bash
SESSION="${1:-agent-team}"
echo "=== Agent Team Status: $SESSION ==="
for WINDOW in $(tmux list-windows -t "$SESSION" -F '#{window_index}:#{window_name}'); do
  IDX="${WINDOW%%:*}"; NAME="${WINDOW#*:}"
  LAST_OUTPUT=$(tmux capture-pane -t "$SESSION:$IDX" -p | tail -3 | sed 's/^/    /')
  PID=$(tmux list-panes -t "$SESSION:$IDX" -F '#{pane_pid}' | head -1)
  STATUS=$(kill -0 "$PID" 2>/dev/null && echo "RUNNING" || echo "DEAD")
  echo "[$STATUS] $NAME (window $IDX)"
  echo "$LAST_OUTPUT"
  echo ""
done
```

## Git Status Across All Worktrees

```bash
#!/bin/bash
PROJECT="${1:-$PWD}"
echo "=== Git Status for All Agent Worktrees ==="
git -C "$PROJECT" worktree list | while read -r WT_PATH HASH BRANCH; do
  echo ""
  echo "--- $WT_PATH ($BRANCH) ---"
  git -C "$WT_PATH" status --short
  AHEAD=$(git -C "$WT_PATH" rev-list --count @{upstream}..HEAD 2>/dev/null || echo "N/A")
  echo "Commits ahead: $AHEAD"
done
```

## Alerting (Optional)

```bash
# macOS notification
osascript -e 'display notification "Agent crashed!" with title "Agent Team Alert"'
# Slack webhook
curl -s -X POST -H 'Content-type: application/json' \
  --data '{"text":"Agent crashed in session agent-team"}' "$SLACK_WEBHOOK_URL"
```
