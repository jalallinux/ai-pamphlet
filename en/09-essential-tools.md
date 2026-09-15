> [Back to index](README.md) · Previous: [08 — Prompt engineering](08-prompt-engineering.md) · Next: [10 — Hooks and commands](10-hooks-and-commands.md)

# Chapter 09 — 6 Essential MCPs and Agents

## Part A — Six MCP servers

> **How to read these entries.** Each one answers the same four questions: *what it is* and where it sits in Claude Code, *how it works*, *what to use it for*, and *how to connect it*. Skim the four headings if you already know the tool.

---

### 1. Context7 — up-to-date documentation

**What it is.** An MCP server that fetches current, version-accurate documentation for a library on demand. It sits in the **tools** layer: Claude calls it mid-answer, the same way it calls `Read` or `Bash`.

**The problem it solves.** A model's knowledge has an expiry date. Left alone it writes code that was correct for last year's version — a deprecated prop, a renamed export, a config key that moved.

**How it works.** Two tools, always used in that order:

| Tool | What it does |
|---|---|
| `resolve-library-id` | Turns a human name ("Next.js") into a Context7 ID (`/vercel/next.js`) |
| `query-docs` | Pulls the doc snippets matching your question for that ID |

The ID can be version-pinned — `/vercel/next.js/v14.3.0` — when you're stuck on an older release. Docs are indexed from the library's own repository, so the answer cites real source, not a model's recollection.

**Use it for:**
- API syntax and configuration for any library, framework, SDK, or CLI
- Version migrations ("what changed between v4 and v5?")
- Library-specific debugging, where the error message means something only in that framework

**Don't use it for:** your own business logic, refactoring, or general programming concepts. Context7 knows public libraries, not your codebase.

**Connect it:**

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

Or remote, with no local Node process:

```bash
claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp
```

**Use it:**

```
use context7: how do I write a server action with Zod validation in Next.js 15 App Router?
```

> **Pro tip — make it automatic.** Put a rule in `~/.claude/CLAUDE.md` so you never have to say "use context7":
>
> ```markdown
> For any question about a library, framework, or SDK, fetch the docs
> from Context7 before answering — even for libraries you know well.
> ```
>
> Better still, put it in `~/.claude/rules/context7.md` and import it — see [chapter 07](07-claude-md-agents-md.md).

Source: https://github.com/upstash/context7

---

### 2. Playwright — browser control

**What it is.** An MCP server that drives a real browser. It sits in the **tools** layer and gives Claude something it otherwise lacks completely: eyes and hands on the running application.

**The problem it solves.** Claude writes a UI and then cannot see it. Without a browser, "does the login button actually work?" is unanswerable — it can only re-read the source and guess.

**How it works.** This is the key idea, and it's what makes Playwright MCP better than screenshot-based approaches:

> It operates on the **accessibility tree**, not on pixels.

Claude calls `browser_snapshot` and gets back a structured description of the page — roles, labels, text, and a stable `ref` for each element. It then acts on a `ref`, not on coordinates. No vision model, no guessing where a button is, and the same snapshot works headless in CI.

Core tools you'll see in a session:

| Tool | Purpose |
|---|---|
| `browser_navigate` | Go to a URL |
| `browser_snapshot` | Read the page structure (use this before acting) |
| `browser_click`, `browser_type`, `browser_fill_form` | Interact |
| `browser_console_messages` | Read JS errors |
| `browser_network_requests` | Inspect API calls the page made |
| `browser_take_screenshot` | Capture an image — for *you* to look at, not for Claude to act on |

**Use it for:**
- Verifying a change in the real app rather than trusting the diff
- Reproducing a bug report step by step
- Responsive checks — `browser_resize` to 375px and report what breaks
- Reading console errors and failed network calls during debugging
- Generating end-to-end tests from a flow it just performed

**Connect it:**

```bash
claude mcp add --scope user playwright -- npx -y @playwright/mcp@latest
```

Useful flags, appended after the package name:

| Flag | Effect |
|---|---|
| `--headless` | No visible window — the right choice for CI |
| `--isolated` | Fresh profile each run, discarded on close |
| `--browser firefox` | `chrome`, `firefox`, `webkit`, or `msedge` |
| `--device "iPhone 15"` | Emulate a named device |

**Use it:**

```
Open localhost:3000, fill the login form with test@example.com,
submit, and tell me what the page shows
```

```
View the page at 375px width and report mobile layout problems
```

> **Cost warning.** A page snapshot is large. A ten-step browser session can cost more context than the code change that prompted it. Give it a precise task and close the browser when done.

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

**What it is.** An MCP server that gives Claude a **language server** — the same engine your IDE uses for "go to definition" and "find all references". It sits in the tools layer, but it changes how Claude *reads* your codebase, which makes it the highest-leverage server on this list for large repos.

**The problem it solves.** By default Claude reads code the way you would with `cat` and `grep`: whole files, matched by text. On a 2,000-line file, asking about one method costs you the entire file in context. And `grep "createSession"` returns the definition, every call, every comment, and every unrelated string that happens to contain it.

**How it works.** Serena starts a real language server for your project's languages and exposes **symbol-level** operations. Claude asks for a *symbol path* — `AuthService/login` — and gets back exactly that method's body.

| Tool | What it does |
|---|---|
| `get_symbols_overview` | The skeleton of a file — classes and methods, no bodies |
| `find_symbol` | Read one symbol by path, e.g. `AuthService/login` |
| `find_referencing_symbols` | True callers, resolved by the compiler — not text matches |
| `find_implementations` | Who implements this interface |
| `replace_symbol_body` | Rewrite one method without touching the rest of the file |
| `rename_symbol` | Project-wide rename, correctly scoped |
| `get_diagnostics_for_file` | Compiler and linter errors, as your IDE sees them |

**The memory system.** Serena also gives Claude a durable per-project notebook in `.serena/memories/` — `write_memory`, `read_memory`, `list_memories`. Unlike `CLAUDE.md`, these load *on demand*, so architecture notes written in one session cost nothing until a later session needs them.

**Use it for:**
- Any repo large enough that "read the file" is wasteful
- "Who actually calls this?" before you change a signature
- Safe renames and single-method rewrites
- Onboarding to an unfamiliar codebase — overview first, then drill in

**Connect it.** Install once, then add it:

```bash
uv tool install -p 3.13 serena-agent
claude mcp add --scope user serena -- serena start-mcp-server --context claude-code --project-from-cwd
```

`--context claude-code` matters: it trims Serena's tool list so it doesn't duplicate Claude Code's own `Read`/`Edit`/`Bash`. `--project-from-cwd` activates whichever project you launched from.

> Shortcut: `serena setup claude-code` does the same wiring for you. If the server is slow to boot, `export MCP_TIMEOUT=60000`.

**Use it:**

```
Give me an overview of the symbols in src/auth/service.ts
Show me only the login method of AuthService
Which symbols reference createSession?
Rename UserRepo to AccountRepo across the codebase
```

> **Why it matters for cost.** On a large repo, "read the whole file" burns thousands of tokens per question. Serena reads the one function you asked about. On a long session this is often the difference between one compaction and five.

Source: https://github.com/oraios/serena · Docs: https://oraios.github.io/serena

---

### Honorable mentions

| Server | What it does | Where |
|---|---|---|
| **PostgreSQL/DBHub** | Direct database queries | `npx -y @bytebase/dbhub` |
| **Supabase** | Database, auth, and storage management | https://github.com/supabase-community/supabase-mcp |
| **Figma** | Read designs and convert to code | https://github.com/GLips/Figma-Context-MCP |
| **Sentry** | Read production errors | https://mcp.sentry.dev/mcp |
| **Chrome DevTools** | Debugging and performance profiling | https://github.com/ChromeDevTools/chrome-devtools-mcp |

---

## Part B — OpenSpec / spec-driven development

**What it is.** Not an MCP server — a **working method**, delivered as a set of skills and slash commands that `openspec init` installs into your project. Its place in Claude Code is the layer *above* everything else in this chapter: MCP servers give Claude capability, OpenSpec gives it an agreed definition of done.

**The central idea:**

> Before writing code, write the *spec*. The spec is the source of truth, not the code.

**The problem it solves.** The dominant failure mode in AI-assisted coding is not bad code — it's confidently correct code that solves the wrong problem. You asked for rate limiting; you got per-process in-memory limiting, which silently does nothing behind a load balancer. Neither of you was wrong, because neither of you said it. A spec forces that conversation to happen *before* the tokens are spent.

**How it works.** Everything lives in an `openspec/` directory as plain Markdown — no custom syntax, no database:

```
openspec/
├── specs/                        # the approved, current truth
└── changes/
    ├── add-rate-limiting/
    │   ├── proposal.md           # what changes, and why
    │   ├── design.md             # how (optional, for hard ones)
    │   ├── specs/                # the new behavior, as requirements
    │   └── tasks.md              # the implementation checklist
    └── archive/
        └── 2026-01-23-add-dark-mode/
```

Requirements are written in verifiable `WHEN` / `THEN` scenarios, so "done" is testable rather than debatable.

The lifecycle, as slash commands:

| Command | Stage |
|---|---|
| `/opsx:explore` | Investigate the problem before proposing anything |
| `/opsx:propose` | Draft the proposal and requirements — **you review here** |
| `/opsx:apply` | Claude implements against the approved spec |
| `/opsx:archive` | Move the completed change into `specs/` as the new truth |

**Connect it:**

```bash
npm install -g @fission-ai/openspec@latest
cd your-project
openspec init
```

`init` asks which tools you use and writes the matching integration — for Claude Code that's `.claude/skills/openspec-*/` and `.claude/commands/opsx/`. Both are ordinary [skills](06-skills.md) and [commands](10-hooks-and-commands.md), so nothing here is magic: you can open and edit them. Run `openspec update` after upgrading to regenerate them.

**Use it:**

```
/opsx:propose add rate limiting to the public API
```

Read what it proposes. Argue with it. *Then*:

```
/opsx:apply
/opsx:archive
```

> **The discipline, not the tool.** The value is the review gate between `propose` and `apply`. If you approve every proposal unread, you've added ceremony and gained nothing.

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
