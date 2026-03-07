# CLI Commands & Flags Reference

> **Source:** https://code.claude.com/docs/en/cli-reference
> **Fetched:** 2026-03-06
> **Version anchor:** v2.1.63

## Core Commands

| Command | Purpose |
|---|---|
| `claude` | Start interactive session |
| `claude "query"` | Start with initial prompt |
| `claude -p "query"` | Non-interactive (print mode), then exit |
| `cat file \| claude -p "query"` | Process piped content |
| `claude -c` | Continue most recent conversation in current directory |
| `claude -r "session" "query"` | Resume session by ID or name |
| `claude update` | Update to latest version |
| `claude auth login/logout/status` | Authentication management |
| `claude agents` | List all configured subagents |
| `claude mcp` | Manage MCP servers |
| `claude remote-control` | Start Remote Control session from Claude.ai |
| `claude mcp serve` | Use Claude Code itself as an MCP server |

## Key Flags

### Session Management
- `--continue, -c` — Resume most recent conversation
- `--resume, -r` — Resume specific session by ID/name or show picker
- `--fork-session` — Create new session ID when resuming (with --resume/--continue)
- `--from-pr` — Resume sessions linked to a GitHub PR
- `--session-id` — Use a specific UUID for the conversation
- `--no-session-persistence` — Don't save session to disk (print mode only)

### Model & Output
- `--model` — Set model: `sonnet`, `opus`, or full name like `claude-sonnet-4-6`
- `--fallback-model` — Auto-fallback when default model overloaded (print mode)
- `--output-format` — `text`, `json`, or `stream-json` (print mode)
- `--input-format` — `text` or `stream-json` (print mode)
- `--json-schema` — Get validated JSON output matching schema (print mode)
- `--verbose` — Full turn-by-turn output

### System Prompt Customization
- `--system-prompt` — **Replace** entire default prompt
- `--system-prompt-file` — Replace with file contents
- `--append-system-prompt` — **Append** to default prompt (recommended for most cases)
- `--append-system-prompt-file` — Append file contents to default

### Permissions & Security
- `--permission-mode` — Start in specific mode: `default`, `acceptEdits`, `plan`, `dontAsk`, `bypassPermissions`
- `--allowedTools` — Tools that execute without permission prompts
- `--disallowedTools` — Tools removed from model context entirely
- `--tools` — Restrict which built-in tools Claude can use
- `--dangerously-skip-permissions` — Skip ALL permission prompts (use in sandbox only)

### Working Environment
- `--add-dir` — Add additional working directories
- `--worktree, -w` — Start in isolated git worktree
- `--ide` — Auto-connect to IDE on startup
- `--chrome` / `--no-chrome` — Toggle Chrome browser integration
- `--debug` — Debug mode with optional category filter (e.g., `"api,hooks"`)

### Budget & Limits
- `--max-budget-usd` — Max dollar spend before stopping (print mode)
- `--max-turns` — Limit agentic turns (print mode)

### MCP & Extensions
- `--mcp-config` — Load MCP servers from JSON file/string
- `--strict-mcp-config` — Only use servers from --mcp-config
- `--plugin-dir` — Load plugins from directories
- `--agents` — Define custom subagents via JSON for session
- `--agent` — Specify an agent for the current session

### Agent Teams
- `--teammate-mode` — `auto`, `in-process`, or `tmux`
- `--remote` — Create web session on claude.ai

## Interactive Session Shortcuts

- `Esc` — Stop Claude mid-action (context preserved)
- `Esc + Esc` or `/rewind` — Open rewind menu (restore conversation/code)
- `Ctrl+G` — Open plan in text editor
- `Ctrl+B` — Background a running task
- `Ctrl+O` — Toggle verbose mode
- `!command` — Run shell command directly
- `@file` — Reference file in prompt
- `/clear` — Reset context
- `/compact [instructions]` — Compact conversation with optional focus
- `/model` — Switch model mid-session
- `/hooks` — Configure hooks interactively
- `/mcp` — Manage MCP servers and authentication
- `/agents` — Create/edit subagents
- `/permissions` — Manage tool permissions
- `/rename` — Give session a descriptive name
- `/sandbox` — Enable OS-level isolation
