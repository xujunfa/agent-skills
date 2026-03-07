---
name: claude-code-expert
description: "Use when working with Claude Code — covers CLI commands, best practices, CLAUDE.md configuration, permissions, hooks, MCP servers, subagents, skills, plugins, agent teams, troubleshooting, and advanced workflows."
trigger_keywords:
  - claude code
  - claude cli
  - CLAUDE.md
  - settings.json
  - claude mcp
  - claude hooks
  - subagents
  - claude permissions
  - /clear
  - /compact
  - claude -p
  - --dangerously-skip-permissions
---

# Claude Code Expert Knowledge

## Claude Usage Instructions
When the user asks about Claude Code, load the relevant reference
file(s) below on demand. Do NOT load all files at once.

## Tool Overview
- **Nature:** CLI tool (agentic coding assistant)
- **Current version anchor:** v2.1.63
- **Official docs:** https://code.claude.com/docs/en/overview

## On-Demand Loading Guide

| Situation / Question | Load file |
|---|---|
| CLI commands, flags, keyboard shortcuts, slash commands | `references/cli-commands.md` |
| Best practices, workflow patterns, CLAUDE.md writing, prompt engineering | `references/best-practices.md` |
| settings.json, permissions, security, managed settings, sandboxing | `references/config-and-permissions.md` |
| MCP servers, hooks, automation, transport types, hook events | `references/mcp-and-hooks.md` |
| Subagents, skills, plugins, agent teams, extensibility decisions | `references/agents-and-skills.md` |

## Key Gotchas
- Context window is the #1 resource to manage; use `/clear` between tasks, subagents for research
- CLAUDE.md should be 50-200 lines; bloated files cause Claude to ignore rules
- `bypassPermissions` only in isolated containers/VMs without internet; never in production
- Permission rules: deny > ask > allow (first match wins); denied at ANY level = blocked everywhere
- MCP SSE transport is deprecated; use HTTP for remote servers
- Stop hooks must check `stop_hook_active` to prevent infinite loops
- Subagents cannot spawn other subagents; chain from main conversation
- `/path` in Read/Edit rules is relative to **project root**, NOT absolute; use `//path` for absolute

## Maintenance
- **Version:** 1.0.0
- **Created:** 2026-03-06
- **Tool version anchor:** v2.1.63
