> [Back to index](README.md) · Previous: [04 — MCP](04-mcp.md) · Next: [06 — Skills](06-skills.md)

# Chapter 05 — Subagents

## Definition

A subagent is **a Claude instance with its own context window, its own system prompt, and a restricted tool set**, which the main Claude calls for a specialized job.

## Why you need them

| Problem | Subagent solution |
|---|---|
| Searching 50 files fills your main context | The subagent searches and returns only the **conclusion** |
| A generalist agent is mediocre at security | A subagent with a security-specific system prompt |
| You want an agent that only reads, never writes | `tools: Read, Grep, Glob` |
| Several jobs at once | Several subagents in parallel |

**The core idea is context isolation.** Dirty work in a separate window, a clean summary back to the main one.

## File locations

| Location | Scope | Priority |
|---|---|---|
| `.claude/agents/` | Project (commit to git) | Higher |
| `~/.claude/agents/` | All your projects | Lower |
| `<plugin>/agents/` | Where the plugin is enabled | Lowest |

## File structure

`.claude/agents/code-improver.md`:

```markdown
---
name: code-improver
description: Scans files and suggests improvements for readability, performance, and best practices. Use after writing or modifying code.
tools: Read, Grep, Glob
model: sonnet
---

You are a code improvement specialist. For each issue you find, explain
the problem, show the current code, and provide an improved version.
```

The body is that agent's system prompt. The subagent sees **only** this, not Claude Code's main system prompt.

## Frontmatter fields

Required: `name`, `description`.

| Field | Value |
|---|---|
| `name` | lowercase + hyphens. No `:`, no leading `-` |
| `description` | **when** Claude should delegate to this agent — the most important field |
| `tools` | allowlist. Omit it and the agent inherits everything |
| `disallowedTools` | denylist |
| `model` | `sonnet` / `opus` / `haiku` / `inherit` |
| `permissionMode` | `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions` |
| `maxTurns` | turn cap |
| `skills` | skills preloaded at startup |
| `mcpServers` | MCP servers scoped to this agent |
| `memory` | `user` / `project` / `local` |
| `color` | `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |
| `isolation` | `worktree` — work on an isolated copy of the repo |

## Writing a good `description`

This field decides whether Claude reaches for the agent on its own.

❌ `description: Reviews code`
✅ `description: Reviews code for security vulnerabilities. Use PROACTIVELY after any change to auth, session handling, or database queries.`

The words `PROACTIVELY` or `MUST BE USED` make Claude delegate sooner.

## Invoking

```
# Natural language — Claude decides
Use the test-runner subagent to fix failing tests

# @-mention — guaranteed
@agent-code-reviewer look at the auth changes

# Whole session as one agent
claude --agent code-reviewer
```

See running agents:

```
/tasks
```

## Three ready-made agents for class

### 1. Security review

`.claude/agents/security-reviewer.md`:

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities. MUST BE USED after any change to authentication, authorization, input handling, or database queries.
tools: Read, Grep, Glob, Bash
model: opus
color: red
---

You are a security engineer reviewing code for vulnerabilities.

Check for, in priority order:
1. Injection — SQL, command, template, XSS
2. Broken authentication / authorization — missing checks, IDOR
3. Hardcoded secrets — keys, tokens, passwords
4. Unsafe deserialization and path traversal
5. Missing input validation at trust boundaries

For each finding report:
- File and line
- Severity: critical / high / medium / low
- Concrete failure scenario — what an attacker actually does
- The fix, as a diff

Report nothing you cannot point to a line for. No generic advice.
```

### 2. Test runner

```markdown
---
name: test-runner
description: Runs the test suite and fixes failing tests. Use PROACTIVELY after code changes.
tools: Read, Edit, Bash, Grep
model: sonnet
color: green
---

Run the project's test suite. For each failure:
1. Read the failing test and the code under test
2. Decide: is the test wrong, or the code wrong?
3. Fix the actual cause — never delete or skip a test to make it pass
4. Re-run to confirm

Report: how many passed, how many fixed, what remains broken and why.
```

### 3. Codebase explorer (read-only)

```markdown
---
name: explorer
description: Searches the codebase to answer "where is X" or "how does Y work". Use when a question requires reading many files.
tools: Read, Grep, Glob
model: haiku
color: cyan
---

Answer the question by searching the codebase. Cite every claim with
`file_path:line_number`. Read only what you need. Return a concise
answer, not file dumps. If you cannot find it, say so — don't guess.
```

## How Claude chooses

When a suitable agent exists and Claude could also do the job itself, it decides based on `description`. If it doesn't delegate, tighten the `description` or invoke explicitly with `@agent-name`.

## Sources

- Subagents — https://code.claude.com/docs/en/sub-agents
- Community agent collection — https://github.com/wshobson/agents
- Another collection — https://github.com/VoltAgent/awesome-claude-code-subagents
