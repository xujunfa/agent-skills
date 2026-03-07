# Resilience & Production Deployment

> **Sources:** tmux-resurrect, tmux-continuum, OpenClaw docs, community patterns
> **Fetched:** 2026-03-07
> **Version anchor:** tmux-resurrect latest / tmux-continuum latest

## tmux-resurrect + continuum (Mandatory for 24/7)

### Installation

```bash
# ~/.tmux.conf:
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
# Install: prefix + I
```

### Recommended Configuration

```bash
# --- Resurrect ---
set -g @resurrect-capture-pane-contents 'on'
set -g @resurrect-strategy-vim 'session'
set -g @resurrect-strategy-nvim 'session'

# --- Continuum ---
set -g @continuum-restore 'on'          # auto-restore on tmux start
set -g @continuum-save-interval '5'     # save every 5 min
set -g @continuum-boot 'on'             # auto-start tmux on boot (macOS)

# --- Status bar ---
set -g status-right 'Continuum: #{continuum_status} | %H:%M'
```

### What Resurrect Saves vs Doesn't

| Saved | NOT Saved |
|---|---|
| Window/pane layout | Running processes (agents!) |
| Working directories | Shell history |
| Pane contents (if enabled) | Environment variables |

**Critical:** After restore, layouts recover but agents are NOT running. Need auto-restart.

## Agent Auto-Restart

### Method 1: tmuxinator (Recommended)

`tmuxinator start agent-team` recreates everything including agent commands.

### Method 2: Wrapper script with retry

```bash
#!/bin/bash
# agent-wrapper.sh — Auto-restart on crash
AGENT_CMD="$1"; MAX_RETRIES=5; RETRY=0
while [ $RETRY -lt $MAX_RETRIES ]; do
  echo "[$(date)] Starting: $AGENT_CMD"
  eval "$AGENT_CMD"
  [ $? -eq 0 ] && break
  RETRY=$((RETRY + 1))
  echo "[$(date)] Crashed. Retry $RETRY/$MAX_RETRIES in $((RETRY * 10))s..."
  sleep $((RETRY * 10))
done
```

### Method 3: launchd (macOS system-level)

```xml
<!-- ~/Library/LaunchAgents/com.agent-team.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>com.agent-team</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/tmuxinator</string>
    <string>start</string>
    <string>agent-team</string>
  </array>
  <key>RunAtLoad</key><true/>
  <key>KeepAlive</key><true/>
</dict>
</plist>
```

`launchctl load ~/Library/LaunchAgents/com.agent-team.plist`

## Production Deployment Targets

| Platform | Setup | Notes |
|---|---|---|
| Local Mac Mini | tmux + tmuxinator + launchd | Best for personal 24/7 |
| AWS Lightsail | SSH + tmux + systemd | $5/mo, persistent |
| Docker | OpenClaw Docker image + tmux | See openclaw-expert |
| Home server | tmux + Tailscale for remote | Secure networking |

## Security Hardening

- **Sandbox workspaces:** each agent confined to its worktree directory
- **Independent auth:** don't share API keys across agents
- **Minimal permissions:** Codex `--sandbox workspace-write` not `--yolo`
- **Prompt injection defense:** never pipe agent output into another agent's prompt unsanitized
- **Git safety:** never stash, never force-push, scope commits per agent

## Health Check Script

```bash
#!/bin/bash
SESSION="${1:-agent-team}"
if ! tmux has-session -t "$SESSION" 2>/dev/null; then
  echo "ALERT: Session $SESSION not found! Restarting..."
  tmuxinator start "$SESSION"
  exit 1
fi
for WINDOW in $(tmux list-windows -t "$SESSION" -F '#{window_name}'); do
  PANE_PID=$(tmux list-panes -t "$SESSION:$WINDOW" -F '#{pane_pid}')
  if ! kill -0 "$PANE_PID" 2>/dev/null; then
    echo "WARN: $WINDOW process dead."
  fi
done
echo "Health check complete."
```

Cron: `*/5 * * * * /path/to/health-check.sh >> /var/log/agent-team.log 2>&1`

## Cost Management for 24/7 Operations

| Team Composition | Per-session | Daily (10 sessions) | Monthly |
|---|---|---|---|
| 1 Opus + 2 Sonnet + 2 Codex | ~$13 | ~$130 | ~$3,900 |
| 1 Sonnet + 4 Codex | ~$9 | ~$90 | ~$2,700 |
| Single Opus | ~$4 | ~$40 | ~$1,200 |

**Cost reduction:** Use Sonnet for orchestrator (unless truly complex), Codex for all batch work, set `activeHours` in OpenClaw heartbeat to avoid idle costs.

## Commit Discipline for Multi-Agent

- Commit early, commit often — unpushed worktree commits lost on crash
- Each agent pushes to its own branch — never to main/develop
- Use conventional commits: `feat(frontend): add auth flow [agent-claude-1]`
- Orchestrator reviews and merges via sequential merge workflow
