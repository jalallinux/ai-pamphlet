> [Back to index](README.md) · Previous: [08 — Prompt engineering](08-prompt-engineering.md) · Next: [10 — Hooks and commands](10-hooks-and-commands.md)

# Chapter 09 — 6 Essential MCPs and Agents

## Part A — Six MCP servers

### 1. Context7 — up-to-date documentation

**The problem it solves:** a model's knowledge has an expiry date. It writes code that was correct for last year's version.

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

Or remote:

```bash
claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp
```

Usage:

```
use context7: how do I write a server action with Zod validation in Next.js 15 App Router?
```

It exposes two tools: `resolve-library-id` and `query-docs`.

> **Pro tip:** add a line to `~/.claude/CLAUDE.md` — "For any library question, fetch docs from Context7 first" — to make it automatic.

Source: https://github.com/upstash/context7

---

### 2. Playwright — browser control

**The problem it solves:** Claude can't see the UI it built. With Playwright it can look, click, and write tests.

```bash
claude mcp add --scope user playwright -- npx -y @playwright/mcp@latest
```

Usage:

```
Open localhost:3000, fill the login form with test@example.com,
submit, and screenshot the result
```

```
View the page at 375px width and report mobile layout problems
```

It works off the accessibility tree rather than pixels — faster and more reliable than screenshots for interaction.

Source: https://github.com/microsoft/playwright-mcp

---

### 3. GitHub — issues, PRs, CI

```bash
claude mcp add --scope user --transport http github https://api.githubcopilot.com/mcp/
```

Then run `/mcp` in session and complete the OAuth flow.

Usage:

```
List open issues labeled bug
Review PR #123 and leave a comment
Why did CI fail on this branch?
```

> Simpler alternative: the `gh` CLI. If it's installed, Claude can use `Bash(gh ...)` with no MCP at all — lighter on context.

Source: https://github.com/github/github-mcp-server

---

### 4. Filesystem — access outside the project

```bash
claude mcp add --scope user filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/Documents ~/Projects
```

Only the paths you explicitly list are reachable. Useful for reference material, datasets, or docs outside the repo.

> In Claude Code, `--add-dir` usually suffices. This MCP helps when you want tighter control.

Source: https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem

---

### 5. Sequential Thinking — structured reasoning

**The problem it solves:** on hard problems, models jump to a conclusion. This server forces step-by-step decomposition with revision.

```bash
claude mcp add --scope user sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

Usage:

```
Using sequential thinking: why does this memory leak only happen in production?
```

Source: https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking

---

### 6. Serena — semantic code navigation

**The problem it solves:** `grep` finds text; Serena finds *symbols*. It runs a language server and lets Claude jump to a definition, list a class's methods, or find every caller — reading only the relevant symbol body instead of whole files.

```bash
claude mcp add --scope user serena -- uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant
```

Usage:

```
Find the AuthService class and show only its login method
Which symbols reference createSession?
Rename UserRepo to AccountRepo across the codebase
```

Key tools: `find_symbol`, `find_referencing_symbols`, `get_symbols_overview`, `replace_symbol_body`, `rename_symbol`, plus a project **memory** system (`write_memory` / `read_memory`) that persists notes between sessions in `.serena/memories/`.

> **Why it matters for cost:** on a large repo, "read the whole file" burns thousands of tokens. Serena reads the one function you asked about.

Source: https://github.com/oraios/serena

---

### Honorable mentions

| Server | What it does | Where |
|---|---|---|
| **PostgreSQL/DBHub** | Direct database queries | `npx -y @bytebase/dbhub` |
| **Figma** | Read designs and convert to code | https://github.com/GLips/Figma-Context-MCP |
| **Sentry** | Read production errors | https://mcp.sentry.dev/mcp |
| **Chrome DevTools** | Debugging and performance profiling | https://github.com/ChromeDevTools/chrome-devtools-mcp |

---

## Part B — OpenSpec / spec-driven development

This isn't an MCP — it's a **working method**. The central idea:

> Before writing code, write the *spec*. The spec is the source of truth, not the code.

The OpenSpec flow:

```
1. proposal   → what changes and why
2. spec       → exact behavior, in verifiable language
3. tasks      → break into small steps
4. implement  → Claude builds against the spec
5. archive    → the approved spec joins the project knowledge base
```

Install:

```bash
npm install -g @fission-ai/openspec
cd your-project
openspec init
```

```
/openspec:proposal add rate limiting to the public API
/openspec:apply
/openspec:archive
```

**Why mention it in class:** the biggest failure mode in AI-assisted coding is ambiguous requirements. A spec resolves that before you burn tokens.

Alternatives:
- **Spec Kit** (GitHub) — https://github.com/github/spec-kit
- **Kiro** (AWS) — https://kiro.dev
- OpenSpec — https://github.com/Fission-AI/OpenSpec

---

## Part C — Five essential subagents

| Agent | Job | Suggested model |
|---|---|---|
| `security-reviewer` | Security pass after auth/input/DB changes | `opus` |
| `test-runner` | Run tests and fix failures | `sonnet` |
| `explorer` | Search the codebase, read-only | `haiku` |
| `code-reviewer` | Quality review before commit | `sonnet` |
| `doc-writer` | README and API docs | `sonnet` |

Full definitions for the first three are in [chapter 05](05-subagents.md).

Ready-made collections:
- https://github.com/wshobson/agents
- https://github.com/VoltAgent/awesome-claude-code-subagents

---

## A starter configuration

`.mcp.json` at the project root (commit it):

```json
{
  "mcpServers": {
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

> **Rule:** start with two or three servers. Each one adds its tool definitions to context. Ten installed servers means a crowded context, worse tool selection, and higher cost.

---

## Sources

- MCP in Claude Code — https://code.claude.com/docs/en/mcp
- Official server list — https://github.com/modelcontextprotocol/servers
- MCP registry — https://registry.modelcontextprotocol.io
- Awesome MCP Servers — https://github.com/punkpeye/awesome-mcp-servers
