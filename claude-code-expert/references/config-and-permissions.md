# Configuration & Permissions

> **Source:** https://code.claude.com/docs/en/permissions + https://code.claude.com/docs/en/settings
> **Fetched:** 2026-03-06
> **Version anchor:** v2.1.63

## Settings File Hierarchy (Highest to Lowest Priority)

1. **Managed settings** — Cannot be overridden by anything, including CLI args
2. **CLI arguments** — Temporary session overrides
3. **Local project** (`.claude/settings.local.json`) — Personal, gitignored
4. **Shared project** (`.claude/settings.json`) — Team-shared, git-tracked
5. **User** (`~/.claude/settings.json`) — All your projects

**Rule**: If a tool is denied at ANY level, no other level can allow it.

## Permission System

| Tool type | Example | Approval required | "Don't ask again" scope |
|---|---|---|---|
| Read-only | File reads, Grep | No | N/A |
| Bash commands | Shell execution | Yes | Permanent per project+command |
| File modification | Edit/write | Yes | Until session end |

### Permission Modes

| Mode | Behavior |
|---|---|
| `default` | Prompts for permission on first use |
| `acceptEdits` | Auto-accepts file edits for session |
| `plan` | Read-only; cannot modify files or run commands |
| `dontAsk` | Auto-denies unless pre-approved via permissions.allow |
| `bypassPermissions` | Skips ALL checks (containers/VMs only!) |

### Rule Evaluation Order: deny -> ask -> allow (first match wins)

## Permission Rule Syntax

### Bash Rules
- `Bash(npm run build)` — Exact command
- `Bash(npm run *)` — Wildcard (space + `*` enforces word boundary)
- `Bash(npm*)` — No space: matches `npm` and `npmrc` etc.
- `Bash(git * main)` — Wildcard in middle
- `Bash(* --version)` — Wildcard at start
- Claude Code is shell-operator-aware: `Bash(safe-cmd *)` won't match `safe-cmd && evil-cmd`

### Read/Edit Rules (gitignore spec)
- `//path` — Absolute path from filesystem root
- `~/path` — From home directory
- `/path` — Relative to **project root** (NOT absolute!)
- `path` or `./path` — Relative to current directory
- `*` matches single directory; `**` matches recursively
- Example: `Edit(/docs/**)` = edits in `<project>/docs/`

### Other Tools
- `WebFetch(domain:example.com)` — Domain-specific
- `mcp__server__tool` — MCP tool by server+tool name
- `Agent(Explore)` — Subagent by name

## Recommended Permission Config

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(npm test *)",
      "Bash(git commit *)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git status)"
    ],
    "deny": [
      "Bash(git push *)",
      "Bash(rm -rf *)",
      "Bash(curl *)",
      "Bash(wget *)",
      "Read(.env*)",
      "Edit(.env*)"
    ]
  }
}
```

## Security Best Practices

- Block exfiltration vectors: `curl`, `wget`, `nc`, `ssh` in global deny
- Protect sensitive files: `.env`, credentials, `.ssh/`
- Layer security: OS sandbox + permission rules + hooks
- 78% of prompt injection attempts target data exfiltration via network
- `bypassPermissions` only in isolated containers without internet
- Enterprise: use `managed-settings.json` for org-wide policies
  - macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`
  - Linux: `/etc/claude-code/managed-settings.json`

## Managed-Only Settings

| Setting | Purpose |
|---|---|
| `disableBypassPermissionsMode` | Prevent bypass mode entirely |
| `allowManagedPermissionRulesOnly` | Only managed permission rules apply |
| `allowManagedHooksOnly` | Only managed/SDK hooks allowed |
| `allowManagedMcpServersOnly` | Only managed MCP allowlist |
| `blockedMarketplaces` | Block plugin marketplace sources |
| `allow_remote_sessions` | Control remote session access |

## Working Directories

- Default: directory where Claude was launched
- Extend via: `--add-dir <path>` (startup), `/add-dir` (session), `additionalDirectories` (settings)
- Additional dirs get same permission rules as original working directory
