# MCP Servers & Hooks

> **Source:** https://code.claude.com/docs/en/mcp + https://code.claude.com/docs/en/hooks-guide
> **Fetched:** 2026-03-06
> **Version anchor:** v2.1.63

## MCP Server Transports

| Transport | Use case | Status |
|---|---|---|
| **HTTP** | Remote/cloud services (recommended) | Current standard |
| **stdio** | Local processes needing system access | Stable |
| **SSE** | Remote servers | **Deprecated**, use HTTP |

## Adding MCP Servers

```bash
# HTTP (recommended for remote)
claude mcp add --transport http notion https://mcp.notion.com/mcp

# stdio (local)
claude mcp add --transport stdio --env API_KEY=xxx airtable -- npx -y airtable-mcp-server

# With auth header
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer token"

# From JSON config
claude mcp add-json weather '{"type":"http","url":"https://api.weather.com/mcp"}'

# Import from Claude Desktop
claude mcp add-from-claude-desktop
```

**Flag ordering**: All options BEFORE server name. `--` separates name from command+args.

## MCP Scopes

| Scope | Storage | Sharing |
|---|---|---|
| `local` (default) | `~/.claude.json` under project path | Private, current project only |
| `project` | `.mcp.json` in project root | Git-tracked, team-shared |
| `user` | `~/.claude.json` | Private, all your projects |

Precedence: local > project > user (same-name servers)

## Key MCP Tips

- `claude mcp list/get/remove` for server management
- `/mcp` in session for status and OAuth authentication
- `MCP_TIMEOUT=10000` env var for startup timeout
- `MAX_MCP_OUTPUT_TOKENS=50000` for large tool outputs (default: 25,000)
- `ENABLE_CLAUDEAI_MCP_SERVERS=false` to opt out of claude.ai servers
- `.mcp.json` supports env var expansion: `${VAR}`, `${VAR:-default}`
- Tool Search auto-enables when MCP tools exceed 10% context; `ENABLE_TOOL_SEARCH=auto:5` for custom threshold
- MCP resources accessible via `@server:protocol://resource/path` in prompts

---

## Hooks System

### Hook Events

| Event | When | Can Block? |
|---|---|---|
| `SessionStart` | Session begins/resumes/clears/compacts | No |
| `UserPromptSubmit` | User submits prompt, before processing | No |
| `PreToolUse` | Before tool call executes | **Yes** (exit 2) |
| `PostToolUse` | After tool call succeeds | No |
| `PostToolUseFailure` | After tool call fails | No |
| `PermissionRequest` | Permission dialog appears | Yes |
| `Notification` | Claude needs attention | No |
| `SubagentStart/Stop` | Subagent lifecycle | No |
| `Stop` | Claude finishes responding | Yes (continue) |
| `ConfigChange` | Config file changes | Yes |
| `PreCompact` | Before context compaction | No |
| `SessionEnd` | Session terminates | No |
| `TaskCompleted` | Task marked complete | Yes |
| `InstructionsLoaded` | CLAUDE.md loaded | No |
| `WorktreeCreate/Remove` | Worktree lifecycle | Replace behavior |

### Hook Types

| Type | Mechanism |
|---|---|
| `command` | Run shell command; stdin=JSON, exit code=decision |
| `http` | POST to URL; response body=decision |
| `prompt` | Single-turn LLM evaluation (Haiku default) |
| `agent` | Multi-turn subagent with tool access |

### Exit Codes (command hooks)
- **0**: Proceed. stdout → Claude's context (SessionStart/UserPromptSubmit)
- **2**: **Block** the action. stderr → Claude's feedback
- **Other**: Proceed. stderr logged (visible in verbose mode Ctrl+O)

### Common Hook Patterns

**Auto-format after edits:**
```json
{"hooks":{"PostToolUse":[{"matcher":"Edit|Write","hooks":[
  {"type":"command","command":"jq -r '.tool_input.file_path' | xargs npx prettier --write"}
]}]}}
```

**Block protected files:**
```json
{"hooks":{"PreToolUse":[{"matcher":"Edit|Write","hooks":[
  {"type":"command","command":".claude/hooks/protect-files.sh"}
]}]}}
```

**Re-inject context after compaction:**
```json
{"hooks":{"SessionStart":[{"matcher":"compact","hooks":[
  {"type":"command","command":"echo 'Reminder: use Bun, not npm. Run bun test before committing.'"}
]}]}}
```

**Desktop notification:**
```json
{"hooks":{"Notification":[{"matcher":"","hooks":[
  {"type":"command","command":"osascript -e 'display notification \"Claude needs attention\" with title \"Claude Code\"'"}
]}]}}
```

**Stop hook with completion check (prevent infinite loop!):**
```bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0  # Allow stop — prevents loop
fi
# ... your verification logic
```

### Hook Configuration Scopes
- `~/.claude/settings.json` — All projects
- `.claude/settings.json` — Project, shared
- `.claude/settings.local.json` — Project, personal
- Managed policy settings — Organization-wide
- Plugin/Skill/Agent frontmatter — Scoped to component
