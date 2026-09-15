> [Back to index](README.md) · Previous: [12 — Cheatsheet](12-cheatsheet.md) · Next: [14 — Sample project](14-sample-project.md)

# Chapter 13 — Lab Exercises

Each lab takes 10–20 minutes. The order matters.

---

## Lab 1 — First session (10 min)

**Goal:** get a feel for the loop.

```bash
mkdir ~/lab-01 && cd ~/lab-01
git init
claude
```

In session:

```
Build a Python script that takes a CSV file and prints the mean of every
numeric column. Also create a sample CSV to test with.
Then run it and show me the result.
```

Then:

```
/cost
/context
```

**Discuss:** how many times did it ask permission? Why those?

---

## Lab 2 — Plan Mode (15 min)

**Goal:** feel the difference between investigating first and diving in.

On a real project (or clone a repo):

```bash
git clone https://github.com/expressjs/express ~/lab-02
cd ~/lab-02
claude
```

`Shift+Tab` until you reach plan mode. Then:

```
Explore this codebase and give me a plan for adding a rate limiting
middleware. Don't write code.
```

**Discuss:** how accurate was the plan? What in it was wrong that would have cost you 300 discarded lines if it had gone straight to code?

---

## Lab 3 — Writing `CLAUDE.md` (15 min)

**Goal:** the difference between good and bad context.

Part A — with no file:

```
Add a new endpoint for listing users.
```

Look at the result. It probably doesn't match the rest of the project's style.

Part B:

```
/init
```

Then edit the generated file — add a **Gotchas** section:

```markdown
## Gotchas
- Every endpoint must use `asyncHandler` from `src/utils/`
- Never throw directly — use `AppError`
- Every new endpoint needs an integration test
```

Run `/clear` and repeat the same request.

**Discuss:** compare the two outputs.

---

## Lab 4 — Install and use an MCP (15 min)

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
claude mcp list
```

In session:

```
/mcp
```

Then:

```
use context7: how do I write a Server Action with Zod validation in Next.js 15?
```

Now ask the same question without context7 (`/clear` first).

**Discuss:** which answer was more current? How do you spot a model guessing from stale knowledge?

---

## Lab 5 — Build a subagent (20 min)

Create `.claude/agents/security-reviewer.md`:

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities. MUST BE USED after any change to authentication, input handling, or database queries.
tools: Read, Grep, Glob
model: sonnet
color: red
---

You are a security engineer.

Check for:
1. Injection — SQL, command, XSS
2. Missing authorization checks
3. Hardcoded secrets
4. Missing input validation at trust boundaries

For each finding: file:line, severity, concrete attack scenario, the fix.
Report nothing you cannot point to a line for.
```

Create a deliberately unsafe file:

```javascript
// src/users.js
app.get('/user', (req, res) => {
  const q = `SELECT * FROM users WHERE id = ${req.query.id}`;
  db.query(q, (e, r) => res.json(r));
});
```

Then:

```
@agent-security-reviewer review src/users.js
```

**Discuss:** what did the agent find? Now change `tools` to `Read, Edit` — what changes?

---

## Lab 6 — Build a skill (15 min)

Create `.claude/skills/standup/SKILL.md`:

```markdown
---
description: Generates a daily standup summary from recent git activity. Use when asked for a standup, daily update, or "what did I do".
---

## Commits since yesterday

!`git log --since="yesterday" --oneline --author="$(git config user.name)" || true`

## Current branch

!`git branch --show-current`

## Uncommitted

!`git status --short`

## Instructions

Write a standup update in three sections:
- **Yesterday:** what was completed, in plain language (not commit messages)
- **Today:** what the uncommitted work suggests is in progress
- **Blockers:** anything that looks stuck — failing tests, TODO comments, long-lived branches

Keep it under 8 lines. No commit hashes.
```

```
/standup
```

**Discuss:** why is this a skill and not a `CLAUDE.md` entry?

---

## Lab 7 — Hooks (15 min)

`.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/guard.sh"
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/guard.sh`:

```bash
#!/usr/bin/env bash
input=$(cat)
path=$(echo "$input" | jq -r '.tool_input.file_path // empty')

case "$path" in
  *.env|*/secrets/*)
    echo "Refusing to modify protected file: $path" >&2
    exit 2
    ;;
esac
exit 0
```

```bash
chmod +x .claude/hooks/guard.sh
```

Restart the session, then:

```
Add a new variable to the .env file
```

**Discuss:** how is this different from writing "never modify `.env`" in `CLAUDE.md`?

---

## Lab 8 — Full TDD cycle (20 min)

**Goal:** experience the strongest pattern.

```
Step 1:
Write tests for a function `parseDuration(input: string): number` that
converts strings like "1h30m", "45s", "2d" into seconds. Invalid input
should throw. Don't write the implementation.

Step 2:
Run the tests and confirm they fail for the right reason.

Step 3:
Now implement until they all pass. Don't modify the tests.
```

**Discuss:** how many iterations were needed? Compare against just saying "write this function".

---

## Lab 9 — Playwright and the visual loop (20 min)

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

Create a simple HTML page and serve it. Then:

```
Open localhost:3000 and take a screenshot.
This page breaks on mobile (375px). Resize, look at it,
and fix the CSS. Screenshot again to confirm it's fixed.
```

**Discuss:** how does this compare to writing CSS without seeing the result?

---

## Lab 10 — Capstone project (45 min)

**Goal:** everything together.

Build a small CLI for managing notes (add / list / search / delete) backed by SQLite.

Requirements:
1. Write `CLAUDE.md` **before** any code
2. Start in Plan Mode
3. Implement with TDD
4. Build a review subagent and run it
5. Build a skill for conventional commits
6. Set `permissions.deny` for `.env`
7. Read the diff yourself, then commit

**Presentation:** each student shows their `CLAUDE.md` and explains why each line is there.

---

## Assessment rubric

| Skill | Sign of competence |
|---|---|
| Context management | Knows when to `/clear` |
| Plan Mode | Gets a plan before any large change |
| `CLAUDE.md` | Writes what isn't derivable from the code |
| MCP | Works with 2–3 servers, not 15 |
| Subagents | Writes a `description` that actually triggers |
| Security | Denies `.env`, reads the diff |
| Prompting | Provides constraints and a success criterion |

---

## Further reading

- Common workflows — https://code.claude.com/docs/en/common-workflows
- Interactive prompt tutorial — https://github.com/anthropics/prompt-eng-interactive-tutorial
- Anthropic Cookbook — https://github.com/anthropics/anthropic-cookbook
- Awesome Claude Code — https://github.com/hesreallyhim/awesome-claude-code
