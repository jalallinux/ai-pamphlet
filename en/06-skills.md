> [Back to index](README.md) · Previous: [05 — Subagents](05-subagents.md) · Next: [07 — CLAUDE.md and AGENTS.md](07-claude-md-agents-md.md)

# Chapter 06 — Skills

## Definition

A skill is **a folder of instructions** that Claude loads **only when it's needed**.

The fundamental difference from `CLAUDE.md`:

| | `CLAUDE.md` | Skill |
|---|---|---|
| When it loads | **Always**, every session | Only when relevant |
| Context cost | Permanent | Zero until activated |
| Best for | Short standing rules | Multi-step procedures |

Anthropic calls this **progressive disclosure**: Claude always sees just the name + `description`; the full content loads when it becomes relevant.

## File locations

| Scope | Path |
|---|---|
| Personal | `~/.claude/skills/<skill-name>/SKILL.md` |
| Project | `.claude/skills/<skill-name>/SKILL.md` |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` |

**The command name comes from the folder name**, not the `name` field.
`deploy-staging/SKILL.md` → `/deploy-staging`

## The smallest possible skill

`~/.claude/skills/summarize-changes/SKILL.md`:

```markdown
---
description: Summarizes uncommitted changes and flags anything risky. Use when the user asks what changed, wants a commit message, or asks to review their diff.
---

## Current changes

!`git diff HEAD`

## Instructions

Summarize the changes above in two or three bullet points, then list any
risks you notice such as missing error handling, hardcoded values, or
tests that need updating. If the diff is empty, say there are no
uncommitted changes.
```

Invoke with `/summarize-changes` — or just ask "what did I change?"

## Injecting shell output

`` !`command` `` runs **before** the content reaches the model, and its output replaces the placeholder:

```markdown
Current status: !`git status --short`
Current branch: !`git branch --show-current`
```

⚠️ A non-zero exit code aborts the whole skill invocation. For commands like `grep` that naturally return 1, append `|| true`.

## Frontmatter fields

All optional; `description` is effectively required.

| Field | Purpose |
|---|---|
| `description` | **The most important one** — what it does and when |
| `when_to_use` | Extra trigger phrases |
| `argument-hint` | Argument hint, e.g. `[issue-number]` |
| `allowed-tools` | Grants tool permission for the invoking turn only |
| `disable-model-invocation: true` | Only you invoke it, never Claude automatically |
| `user-invocable: false` | Only Claude invokes it |
| `context: fork` | Runs in a subagent with no conversation history |
| `model` | Pin a specific model |
| `paths` | Auto-activate only for matching files |

## Arguments

```markdown
---
argument-hint: [issue-number]
---

Fix GitHub issue #$1. Read it with `gh issue view $1`, then implement the fix.
```

```
/fix-issue 234
```

Available variables: `$ARGUMENTS`, `$1`, `$2`, `${CLAUDE_SKILL_DIR}`, `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_SESSION_ID}`

## Supporting files

```
my-skill/
├── SKILL.md          # required — keep under 500 lines
├── reference.md      # read when needed
├── examples.md
└── scripts/
    └── helper.py     # executed, not loaded into context
```

Reference them from `SKILL.md` so Claude knows when to read them:

```markdown
For the full API reference, read `reference.md` in this skill's directory.
```

## A skill with a script

```markdown
---
name: render-chart
description: Render a chart from a CSV file
allowed-tools: Bash(${CLAUDE_SKILL_DIR}/scripts/render.sh *)
---

Run `${CLAUDE_SKILL_DIR}/scripts/render.sh <csv-file>` to render the chart.
```

Using the same variable in both places means the script runs without a permission prompt.

## Three practical skills for class

### 1. Conventional commit

`.claude/skills/commit/SKILL.md`:

```markdown
---
description: Creates a Conventional Commit from staged changes. Use when the user asks to commit.
allowed-tools: Bash(git status *) Bash(git diff *) Bash(git commit *)
---

## Staged

!`git diff --cached`

## Instructions

1. If nothing is staged, say so and stop.
2. Pick the type: feat, fix, docs, style, refactor, test, chore.
3. Write: `<type>(<scope>): <subject in imperative, under 50 chars>`
4. Add a body only if the *why* is not obvious from the subject.
5. Show the message, then commit it.

Never include "Generated with AI" or similar in the message.
```

### 2. PR review

`.claude/skills/review-pr/SKILL.md`:

```markdown
---
description: Reviews a pull request for correctness and risk. Use when asked to review a PR.
argument-hint: [pr-number]
---

## PR

!`gh pr view $1 --json title,body,files`

## Diff

!`gh pr diff $1`

## Instructions

Review in this order and stop at the first empty category:
1. Correctness — logic bugs, off-by-one, null handling, race conditions
2. Security — injection, auth checks, secrets
3. Tests — is the new behavior covered?
4. Maintainability — naming, duplication

For each finding: `file:line`, what breaks, and the fix.
Ignore style the linter already handles. If there is nothing wrong, say so.
```

### 3. graphify — a codebase knowledge graph

An example of a **large installed skill** rather than one you write yourself. It shows what progressive disclosure buys you: `SKILL.md` is a long procedure with a `references/` folder, and none of it enters context until you ask a question about the codebase.

```
~/.claude/skills/graphify/
├── SKILL.md
└── references/
```

What it does: turns a folder of files (code, docs, papers, images, video) into a persistent knowledge graph — nodes, edges, community detection — and writes it to `graphify-out/` next to your project.

```bash
/graphify                      # build a graph of the current directory
/graphify <path> --update      # re-extract only changed files
/graphify query "how does auth work?"
/graphify path "AuthModule" "Database"
/graphify explain "SessionStore"
```

Once `graphify-out/graph.json` exists, a plain question like "how does the login flow work?" is answered from the graph instead of re-reading files.

Each edge carries an audit tag — `EXTRACTED` (found in the source), `INFERRED` (the model's guess), or `AMBIGUOUS` — so you can tell fact from inference.

Complements [Serena](09-essential-tools.md): Serena answers *precise* symbol questions (who calls this function); graphify answers *structural* ones (which parts of this system cluster together).

> **Note on cost:** the initial build reads the whole corpus once — expensive. Queries afterwards are cheap. Build once, query many times.

The skill installs its own CLI on first run:

```bash
uv tool install graphifyy
```

Source: https://github.com/Graphify-Labs/graphify

## Installing ready-made skills

```bash
# Anthropic's plugin marketplace
/plugin marketplace add anthropics/claude-code
/plugin install <name>
```

## Skill vs. subagent vs. CLAUDE.md

| Need | Tool |
|---|---|
| "Always use 2-space indentation" | `CLAUDE.md` |
| "Steps to deploy to staging" | Skill |
| "A security specialist who reviews separately" | Subagent |
| "Run lint before every commit" (enforced) | Hook → [chapter 10](10-hooks-and-commands.md) |

## Sources

- Skills in Claude Code — https://code.claude.com/docs/en/skills
- Skill authoring best practices — https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices
- Anthropic's official skills — https://github.com/anthropics/skills
- Anthropic engineering post — https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
