# Subagents, Skills & Extensibility

> **Source:** https://code.claude.com/docs/en/sub-agents + https://code.claude.com/docs/en/best-practices
> **Fetched:** 2026-03-06
> **Version anchor:** v2.1.63

## Subagents Overview

Subagents run in **separate context windows** with custom system prompts, tool access, and permissions. They return summaries to the main conversation, preserving your context.

### Built-in Subagents

| Agent | Model | Tools | Purpose |
|---|---|---|---|
| **Explore** | Haiku (fast) | Read-only | Codebase search/analysis |
| **Plan** | Inherited | Read-only | Research for plan mode |
| **general-purpose** | Inherited | All | Complex multi-step tasks |
| **Claude Code Guide** | Haiku | Read-only | Questions about Claude Code features |

### Creating Custom Subagents

**Scope & Priority** (highest to lowest):
1. `--agents` CLI flag — Session only, JSON format
2. `.claude/agents/` — Project-level, git-tracked
3. `~/.claude/agents/` — User-level, all projects
4. Plugin `agents/` — Where plugin is enabled

**File format** (Markdown + YAML frontmatter):
```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Grep, Glob, Bash  # Omit to inherit all
model: sonnet  # sonnet, opus, haiku, or inherit
permissionMode: default
maxTurns: 50
skills: [api-conventions]
memory: user  # user, project, or local
background: false
isolation: worktree  # Optional: isolated git worktree
---
System prompt goes here as Markdown body.
```

### Key Subagent Features

- **Resume**: `"Continue that code review"` — retains full conversation history
- **Background mode**: `Ctrl+B` or ask "run this in the background"
- **Persistent memory**: `memory: user|project|local` — cross-session learning
- **Hooks in frontmatter**: `PreToolUse`, `PostToolUse`, `Stop` per-subagent
- **Tool restriction**: `tools` (allowlist) or `disallowedTools` (denylist)
- **Agent spawning control**: `tools: Agent(worker, researcher)` limits which sub-subagents
- **Subagents CANNOT spawn other subagents** — chain from main conversation instead

### When to Use Subagents vs Main Conversation

**Use subagents when:**
- Task produces verbose output you don't need in main context
- You want tool restrictions or specific permissions
- Work is self-contained and can return a summary
- Running parallel research on independent topics

**Use main conversation when:**
- Frequent back-and-forth or iterative refinement needed
- Multiple phases share significant context
- Making quick, targeted changes
- Latency matters (subagents start fresh)

---

## Skills

Skills are folders with `SKILL.md` in `.claude/skills/` that Claude loads **on demand** when relevant. Unlike CLAUDE.md (always loaded), skills don't bloat every conversation.

```markdown
# .claude/skills/api-conventions/SKILL.md
---
name: api-conventions
description: REST API design conventions for our services
---
# API Conventions
- Use kebab-case for URL paths
- Use camelCase for JSON properties
- Always include pagination for list endpoints
```

### Skills vs Subagents vs CLAUDE.md

| Feature | CLAUDE.md | Skills | Subagents |
|---|---|---|---|
| When loaded | Every session | On demand | When delegated |
| Context impact | Always present | Only when relevant | Separate window |
| Modifies code | N/A (instructions) | Can include workflows | Yes (has own tools) |
| Best for | Universal project rules | Domain knowledge, workflows | Isolated task execution |

### Executable Skills (Slash Commands)

```markdown
# .claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---
Analyze and fix the GitHub issue: $ARGUMENTS.
1. Use `gh issue view` to get details
2. Search codebase for relevant files
...
```

Invoke with `/fix-issue 1234`. Use `disable-model-invocation: true` for workflows with side effects.

---

## Plugins

Plugins bundle skills, hooks, subagents, and MCP servers into installable units.

- Browse/install with `/plugin`
- Install code intelligence plugins for typed languages (symbol navigation, auto error detection)
- Plugin MCP servers auto-start when plugin enabled
- Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths

---

## Agent Teams

For sustained parallelism with inter-agent communication, use agent teams instead of subagents.

- Subagents: within single session, no cross-communication
- Agent teams: separate sessions, shared tasks, messaging, team lead
- Configure with `--teammate-mode`: `auto`, `in-process`, or `tmux`

---

## Extension Priority Guide

| Need | Use |
|---|---|
| Permanent instructions for all sessions | CLAUDE.md |
| Domain knowledge loaded when relevant | Skills |
| Isolated task execution with tool control | Subagents |
| Deterministic actions at lifecycle points | Hooks |
| External tool/service integration | MCP servers |
| Package of multiple extensions | Plugins |
| Parallel work across sessions | Agent Teams |
