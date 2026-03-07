---
name: agent-orchestration-expert
description: "Use when orchestrating multi-agent AI teams with OpenClaw + Tmux + Claude Code + Codex — covers architecture, tmux layouts, agent dispatch, worktree isolation, crash recovery, production deployment, and monitoring."
trigger_keywords:
  - agent orchestration
  - tmux agents
  - multi-agent team
  - ClawMaster
  - openclaw tmux
  - codex parallel
  - agent dispatch
  - tmux worktree
  - coding agent team
  - 24/7 agent
  - workmux IttyBitty
---

# Agent Orchestration Expert Knowledge (ClawMaster)

## Claude Usage Instructions
When the user asks about multi-agent orchestration, load the relevant reference
file(s) below on demand. Do NOT load all files at once.
Complement with `openclaw-expert` for OpenClaw platform details and
`cc-agent-teams-expert` for Claude Code native Agent Teams specifics.

## Tool Overview
- **Nature:** Orchestration pattern (OpenClaw + Tmux + coding agents)
- **Current version anchor:** OpenClaw v2026.3.x / Claude Code 2.1.x / Codex CLI 2026.2
- **Key components:** OpenClaw (brain), Tmux (persistence), Claude Code + Codex (executors)

## On-Demand Loading Guide

| Situation / Question | Load file |
|---|---|
| Architecture, agent selection, Claude vs Codex, SOUL.md, cost estimation | `references/architecture-and-decisions.md` |
| Tmux setup, tmuxinator YAML, layouts, workmux, IttyBitty, pane management | `references/tmux-setup-and-layouts.md` |
| Git worktree isolation, dispatch, merge workflow, cleanup | `references/worktree-and-dispatch.md` |
| Crash recovery, tmux-resurrect/continuum, auto-restore, 24/7 production | `references/resilience-and-production.md` |
| Monitoring, dashboard, attach/intervene, status line, log tailing | `references/monitoring-and-intervention.md` |

## Pre-Response Checklist (CoT — execute internally before every response)
1. Task complexity → single agent sufficient, or needs team?
2. Claude vs Codex → which agent type for each subtask?
3. Team size → how many agents? (prefer 3-5)
4. Layout → tiled / main-vertical / zoom?
5. Isolation → worktree per agent, file ownership boundaries?
6. Full command + config → copy-paste ready?

## Key Gotchas
- Every coding agent MUST use an independent git worktree — shared = clobbering
- OpenClaw config is **JSON5** (`openclaw.json`), not YAML
- `tmux-agents` skill installs via `clawhub install`, called via `scripts/spawn.sh`
- Codex `--full-auto` sandbox auto-limits to cwd; `--yolo` only in sandboxed CI
- Claude `--worktree --tmux` stores in `.claude/worktrees/<name>/`; no base branch flag
- tmux-resurrect saves layout but NOT processes — need auto-restart wrapper
- Never `git stash` in multi-agent repos; merge worktrees sequentially

## Response Format (Mandatory)
1. **Plan** — 1-2 sentences: what team, why this composition
2. **One-liner** — prefer OpenClaw skill or `claude --worktree --tmux`
3. **Full config** — tmuxinator.yml or openclaw.json snippet
4. **Layout ASCII** — visual of pane arrangement
5. **Attach / monitor / intervene** — commands to observe and interact
6. **Next steps** — what to do after agents complete (merge, review, cleanup)

## Maintenance
- **Version:** 1.1.0
- **Created:** 2026-03-07
- **Last updated:** 2026-03-07
- **Tool version anchor:** OpenClaw v2026.3.x / Claude Code 2.1.x / Codex 2026.2
