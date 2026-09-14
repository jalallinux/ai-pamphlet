> [Back to index](README.md) · Previous: [09 — Essential tools](09-essential-tools.md) · Next: [11 — Workflow](11-workflow-and-best-practices.md)

# Chapter 10 — Hooks, Settings, and Plugins

## Part A — Hooks

### Definition

A hook is a **shell command** that runs at a fixed point in Claude Code's lifecycle — **regardless of what the model decides**.

Compared to `CLAUDE.md`:

| | `CLAUDE.md` | Hook |
|---|---|---|
| Nature | A request | Enforcement |
| Guarantee | None | Yes |
| Best for | Style and conventions | Lint, format, blocking |

If you wrote a line in `CLAUDE.md` starting with "always..." or "never..." and the model sometimes ignores it — make it a hook.

### Common events

| Event | When |
|---|---|
| `SessionStart` | Session begins |
| `UserPromptSubmit` | After you send a message, before it reaches the model |
| `PreToolUse` | Before a tool runs — **can block it** |
| `PostToolUse` | After a tool succeeds |
| `Stop` | When Claude finishes a response |
| `SubagentStop` | A subagent finishes |
| `PreCompact` | Before context compaction |
| `SessionEnd` | Session ends |

(There are 33+ events in total.)

### Structure

Three nesting levels: event → matcher group → handler array.

`.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/lint-check.sh"
          }
        ]
      }
    ]
  }
}
```

Omitting `matcher` makes it fire on every occurrence.

### Three useful hooks

**1. Auto-format after every edit**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**2. Block edits to sensitive files**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/block-sensitive.sh"
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/block-sensitive.sh`:

```bash
#!/usr/bin/env bash
# hook input arrives as JSON on stdin
input=$(cat)
path=$(echo "$input" | jq -r '.tool_input.file_path // empty')

case "$path" in
  *.env|*/secrets/*|*/.git/*)
    echo "Refusing to modify protected file: $path" >&2
    exit 2   # exit code 2 = block, and tell the model why
    ;;
esac
exit 0
```

Exit code contract:
- `0` → proceed
- `2` → block, show `stderr` to the model
- anything else → non-blocking error

**3. Notify when work finishes**

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Done'"
          }
        ]
      }
    ]
  }
}
```

> ⚠️ **Security warning:** hooks run with your full user privileges and without confirmation. Never copy-paste a hook from an untrusted source. Before committing `.claude/settings.json` to a shared repo, read every hook it contains.

---

## Part B — Slash commands

In recent versions, **skills supersede slash commands**. The old format still works:

```
.claude/commands/deploy.md        →  /deploy
.claude/commands/frontend/test.md →  /frontend:test
```

For anything new, use a [skill](06-skills.md).

### Important built-in commands

| Command | What it does |
|---|---|
| `/help` | Full list |
| `/clear` | Wipe context — always do this between unrelated tasks |
| `/compact` | Summarize context while keeping key points |
| `/context` | What is in context, and how much |
| `/cost` | Session cost |
| `/status` | Login, model, active settings |
| `/memory` | Edit memory files |
| `/mcp` | MCP server status |
| `/tasks` | Running subagents |
| `/model` | Switch model |
| `/init` | Generate a starting `CLAUDE.md` |
| `/doctor` | Diagnose config problems |
| `/review` | Code review |
| `/resume` | Continue a previous session |
| `/rewind` | Rewind the conversation |

### Keyboard shortcuts

| Key | Action |
|---|---|
| `Shift+Tab` | Cycle permission modes |
| `Esc` | Interrupt Claude |
| `Esc Esc` | Edit the previous message |
| `Ctrl+R` | Search history |
| `#` at line start | Quick-add to memory |
| `!` at line start | Run a shell command directly |
| `@` | Reference a file |

---

## Part C — Settings

### File hierarchy

| File | Scope |
|---|---|
| `~/.claude/settings.json` | All your projects |
| `.claude/settings.json` | Project — commit to git |
| `.claude/settings.local.json` | Project, personal — gitignore it |
| managed settings | Org policy — cannot be overridden |

Precedence: managed > local > project > user.

### A complete example

```json
{
  "permissions": {
    "defaultMode": "default",
    "allow": [
      "Bash(npm run test:*)",
      "Bash(npm run lint:*)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ],
    "ask": [
      "Bash(git push:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Bash(rm -rf:*)",
      "Bash(curl:*)"
    ]
  },
  "env": {
    "NODE_ENV": "development"
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\" || true" }
        ]
      }
    ]
  }
}
```

---

## Part D — Plugins

A plugin is a ready-made bundle of skills + agents + MCP servers + hooks.

```
/plugin marketplace add anthropics/claude-code
/plugin install <plugin-name>
/plugin list
```

Community marketplaces:
- https://github.com/anthropics/claude-code
- https://github.com/davila7/claude-code-templates

---

## Sources

- Hooks — https://code.claude.com/docs/en/hooks
- Hooks guide — https://code.claude.com/docs/en/hooks-guide
- Settings — https://code.claude.com/docs/en/settings
- Settings reference — https://code.claude.com/docs/en/settings-reference
- Commands — https://code.claude.com/docs/en/commands
- Plugins — https://code.claude.com/docs/en/plugins
