> [Back to index](README.md) · Previous: [11 — Workflow](11-workflow-and-best-practices.md) · Next: [13 — Lab exercises](13-lab-exercises.md)

# Chapter 12 — Cheatsheet

> Print this page and hand it out.

## Install and start

```bash
curl -fsSL https://claude.ai/install.sh | bash
cd your-project
claude
```

## In-session commands

```
/help          list commands
/init          generate CLAUDE.md
/context       what's in context
/clear         wipe context
/compact       summarize context
/cost          cost
/status        login, model
/model         switch model
/memory        edit memory
/mcp           MCP status
/tasks         running subagents
/doctor        diagnostics
/resume        continue previous session
/rewind        rewind conversation
```

## Keyboard

```
Shift+Tab    cycle modes (default → acceptEdits → plan)
Esc          interrupt
Esc Esc      edit previous message
Ctrl+R       search history
#            add to memory
!            run shell directly
@            reference a file
```

## Modes

| Mode | Runs without asking |
|---|---|
| `default` | Almost nothing |
| `plan` | Reads only, edits blocked |
| `acceptEdits` | Reads + file edits |
| `auto` | Classifier approves instead of you |
| `dontAsk` | Allowlisted tools only, rest denied |
| `bypassPermissions` | Everything ⚠️ |

```bash
claude --permission-mode plan
```

## MCP

```bash
# local (stdio)
claude mcp add <name> -- npx -y <package>

# remote (http)
claude mcp add --transport http <name> <url>

# scope
claude mcp add --scope user    <name> -- ...   # all projects
claude mcp add --scope project <name> -- ...   # .mcp.json, committable
claude mcp add --scope local   <name> -- ...   # this project, private

# manage
claude mcp list
claude mcp get <name>
claude mcp remove <name>
claude mcp login <name>
```

Tool name for permissions: `mcp__<server>__<tool>`

## Subagent

`.claude/agents/<name>.md`:

```markdown
---
name: my-agent
description: What it does and WHEN to use it. Use PROACTIVELY when ...
tools: Read, Grep, Glob
model: sonnet
---

System prompt here.
```

Invoke: `@agent-my-agent` or natural language.

## Skill

`.claude/skills/<name>/SKILL.md`:

```markdown
---
description: What it does and when to use it.
argument-hint: [arg]
allowed-tools: Bash(git diff *)
---

Context: !`git diff HEAD`

Instructions: ...  use $1 for the argument
```

Invoke: `/<folder-name>`

## CLAUDE.md

```
~/.claude/CLAUDE.md      personal, all projects
./CLAUDE.md              project, commit to git
./CLAUDE.local.md        personal, gitignore
.claude/rules/*.md       split rules (with paths:)
```

Rules: under 200 lines · headers and bullets · specific and verifiable · only what isn't derivable from the code.

For `AGENTS.md`:

```markdown
@AGENTS.md
```

## Hooks

`.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\" || true" }] }
    ]
  }
}
```

Events: `SessionStart` `UserPromptSubmit` `PreToolUse` `PostToolUse` `Stop` `SubagentStop` `PreCompact` `SessionEnd`

Exit codes: `0` proceed · `2` block + message to the model

## Permissions

```json
{
  "permissions": {
    "defaultMode": "default",
    "allow": ["Bash(npm test:*)", "Read(src/**)"],
    "ask":   ["Bash(git push:*)"],
    "deny":  ["Read(./.env)", "Bash(rm -rf:*)"]
  }
}
```

Precedence: `deny` > `ask` > `allow`

## API

```bash
# Anthropic
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-5","max_tokens":1024,"messages":[{"role":"user","content":"hi"}]}'

# OpenAI-compatible
curl $BASE_URL/v1/chat/completions \
  -H "Authorization: Bearer $KEY" \
  -d '{"model":"...","messages":[{"role":"user","content":"hi"}]}'
```

Gateway for Claude Code:

```bash
export ANTHROPIC_BASE_URL="https://gateway.example.com"
export ANTHROPIC_AUTH_TOKEN="token"
```

## Quick install: 5 MCPs

```bash
claude mcp add --scope user context7            -- npx -y @upstash/context7-mcp
claude mcp add --scope user playwright          -- npx -y @playwright/mcp@latest
claude mcp add --scope user sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
claude mcp add --scope user filesystem          -- npx -y @modelcontextprotocol/server-filesystem ~/Documents
claude mcp add --scope user --transport http github https://api.githubcopilot.com/mcp/
```

## Thinking keywords

```
think  <  think hard  <  think harder  <  ultrathink
```

## The golden pattern

```
plan mode → get a plan → correct it → approve →
write tests → implement → run tests → review it yourself → commit
```
