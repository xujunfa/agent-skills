# Architecture & Agent Selection Decision Tree

> **Sources:** OpenClaw docs (docs.openclaw.ai), Claude Code docs, Codex CLI Reference
> **Fetched:** 2026-03-07
> **Version anchor:** OpenClaw v2026.3.x / Claude Code 2.1.x / Codex 2026.2

## Core Architecture Stack

```
┌─────────────────────────────────────────────┐
│  OpenClaw (Orchestrator Brain)              │
│  - SOUL.md (agent persona/behavior)         │
│  - JSON5 config: agents + bindings          │
│  - 13,700+ skills on ClawHub                │
├─────────────────────────────────────────────┤
│  Tmux (Persistence Engine)                  │
│  - Shared session, user attaches anytime    │
│  - tiled / main-vertical layouts            │
│  - resurrect + continuum for crash recovery │
├─────────────────────────────────────────────┤
│  Coding Agents (Executors)                  │
│  - Claude Code: complex/architecture/long   │
│  - Codex CLI: fast/batch/auto-approve       │
│  - Each in independent git worktree         │
└─────────────────────────────────────────────┘
```

**No OpenClaw?** Use pure Tmux + tmuxinator. Skip OpenClaw sections; all tmux/worktree/agent patterns still apply.

## SOUL.md (OpenClaw Agent Persona)

Pure Markdown, one per agent, placed at workspace root (`~/.openclaw/workspace-<id>/SOUL.md`).
Defines persona, tone, boundaries. **Not** same as CLAUDE.md (Claude Code specific).

```markdown
# SOUL.md
## Core Truths
**Be genuinely useful, not performatively helpful.** Skip pleasantries — act.
**Have opinions.** When asked "is this good?", give a clear stance.
## Boundaries
- Never execute destructive commands without explicit --force.
- Keep responses under 300 words unless asked to expand.
```

## Agent Selection Decision Tree (Strict)

| Task Type | Agent | Flags / Notes |
|---|---|---|
| Complex architecture / large refactor / deep reasoning | **Claude Code** | `--model opus`, `--worktree --tmux` |
| Quick feature / batch edits / simple impl / high parallelism | **Codex** | `--full-auto` or `-m gpt-5-codex` |
| Testing & validation | **Dedicated Tester** | Isolated pane, runs test suite continuously |
| Research & planning | **OpenClaw Researcher** | Use research skills via ClawHub |
| Code review | **Claude Code** (read-only) | `claude --print` or review prompt |
| Multi-file migration | **Codex x N** in parallel | N worktrees, one codex per worktree |

## Claude Code Key Flags

- `claude --worktree` — auto-create worktree at `.claude/worktrees/<name>/`
- `claude --worktree --tmux` — worktree + persistent tmux session (recommended)
- `claude -p "prompt"` — non-interactive one-shot
- `claude --model opus` — use Opus for complex tasks
- **Limitation:** no `--base-branch` flag; for custom base: `git worktree add` manually first
- Agent Teams: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude`

## Codex CLI Key Flags

- `codex exec "prompt"` — non-interactive scripted run
- `codex --full-auto` — auto-approve workspace writes (sandbox = cwd)
- `codex --dangerously-bypass-approvals-and-sandbox` (alias `--yolo`) — CI only!
- `codex -m gpt-5-codex` — coding-optimized model
- `codex exec --json` — NDJSON event stream (for pipeline integration)

## OpenClaw Multi-Agent Config (JSON5)

```json5
// ~/.openclaw/openclaw.json
{
  "agents": {
    "defaults": { "workspace": "~/.openclaw/workspace", "model": { "primary": "anthropic/claude-sonnet-4-6" } },
    "list": [
      { "id": "main", "default": true, "name": "Orchestrator", "workspace": "~/.openclaw/workspace-main" },
      { "id": "coder", "name": "Coder Agent", "workspace": "~/.openclaw/workspace-coder",
        "model": { "primary": "anthropic/claude-opus-4-6" } }
    ]
  },
  "bindings": [
    { "agentId": "main", "match": { "channel": "telegram" } },
    { "agentId": "coder", "match": { "channel": "discord", "guildId": "dev-guild" } }
  ],
  "tools": { "agentToAgent": { "enabled": true, "allow": ["main", "coder"] } }
}
```

Routing: `bindings[]` matches most-specific-first (peer → accountId → channel).
Inter-agent comms: `tools.agentToAgent.enabled: true` + allowlist (or `sessions_send`).

## Cost Estimation (per mid-size feature, 5-10 files, ~300k tokens)

| Agent | $/session | Best for |
|---|---|---|
| Claude Opus 4.6 | ~$4 | Orchestrator, complex architecture |
| Claude Sonnet 4.6 | ~$2 | Standard implementation |
| Codex gpt-5-codex | ~$1.4 | Batch edits, fast iteration |

| Team Composition | Estimated Total |
|---|---|
| 1 Orchestrator(Opus) + 2 Sonnet + 2 Codex | ~$13 |
| Pure Claude Teams (1 Opus + 4 Sonnet) | ~$15-18 |
| Single Claude Opus session | ~$4-5 |

## Team Sizing Guidelines

| Project Scale | Recommended Team | Layout |
|---|---|---|
| Small feature (1-2 files) | 1 Claude Code | Single pane |
| Medium feature (3-10 files) | 1 Orchestrator + 2 Workers | tiled |
| Large feature / refactor | 1 Orchestrator + 2 Claude + 2 Codex | main-vertical |
| Migration / batch ops | 1 Orchestrator + N Codex | tiled (N=4-8) |
