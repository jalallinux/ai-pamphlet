> [Back to index](README.md) · Previous: [02 — OpenAI-Compatible API](02-openai-compatible-api.md) · Next: [04 — MCP](04-mcp.md)

# Chapter 03 — Claude Code Permission Modes

## The concept

A mode decides **what Claude asks you about before doing it**. It is the single dial balancing speed against control.

## The modes

| Mode (config value) | UI name | Runs without asking | Best for |
|---|---|---|---|
| `default` | Manual | Almost nothing; asks for every edit and command | Sensitive code, unfamiliar code |
| `plan` | Plan | Reads only; **edits blocked** until you approve a plan | Investigating before changing |
| `acceptEdits` | Accept Edits | Reads, file edits, common filesystem commands (`mkdir`, `mv`, ...) | Fast iteration on code you're reviewing |
| `auto` | Auto | A second model (the classifier) reviews actions instead of you | Hands-off work (default on Pro/Max/Team) |
| `dontAsk` | Don't Ask | Only pre-approved tools; anything else is **denied** | Locked-down CI and scripts |
| `bypassPermissions` | Bypass | Everything, no questions | Only inside an isolated container/VM |

## Plan Mode — the habit worth building

The cycle:

```
1. Enter Plan Mode
2. Ask: "Explore the codebase and give me a plan for adding X"
3. Claude only reads and searches — nothing is modified
4. You read the plan and correct it
5. Approve → Claude starts implementing
```

Why it matters: the cheapest place to fix a mistake is *before* code is written. Reading a 10-line plan beats reviewing a 400-line wrong diff.

## Switching modes

**In session:** press `Shift+Tab` to cycle.

Cycle: `default` → `acceptEdits` → `plan` → back to `default`
(From `auto`, the first press goes to `default`.)

The status bar shows the active mode:
```
⏸ manual mode on
⏵⏵ accept edits on
⏸ plan mode on
⏵⏵ auto mode on
```

**At launch:**

```bash
claude --permission-mode plan
claude --permission-mode acceptEdits
claude -p "run the test suite" --permission-mode dontAsk --allowedTools "Bash(npm test)" "Read"
```

**Persistently**, in `.claude/settings.json`:

```json
{
  "permissions": {
    "defaultMode": "default"
  }
}
```

> Note: `auto` and `bypassPermissions` do **not** take effect from a project settings file (security reason). Only from user or managed settings.

## Permission rules

Independent of mode, you can define exact rules:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:*)",
      "Bash(git status)",
      "Read(src/**)"
    ],
    "ask": [
      "Bash(git push:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./secrets/**)",
      "Bash(rm -rf *)"
    ]
  }
}
```

Precedence: `deny` > `ask` > `allow`.

`deny` always wins — even under `bypassPermissions`. Always write a `deny` rule for sensitive files (`.env`, keys).

## Practical guidance for class

| Situation | Mode |
|---|---|
| First time on a project | `plan` |
| Large refactor | `plan` → approve → `acceptEdits` |
| Writing tests, running repeatedly | `acceptEdits` |
| Sensitive production code | `default` |
| CI | `dontAsk` + a tight allowlist |
| `bypassPermissions` on your laptop | ❌ no |

## Sources

- Permission modes — https://code.claude.com/docs/en/permission-modes
- Configure permissions — https://code.claude.com/docs/en/permissions
- CLI reference — https://code.claude.com/docs/en/cli-reference
- Sandboxing — https://code.claude.com/docs/en/sandboxing
