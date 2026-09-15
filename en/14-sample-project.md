> [Back to index](README.md) · Previous: [13 — Lab exercises](13-lab-exercises.md)

# Chapter 14 — Sample Project: URL Shortener

One project, ten steps, every chapter of this handbook used once. Budget 45–90 minutes.

The point is not the shortener. The point is that each step has a **deliberate failure** that a different Claude Code feature is supposed to catch. If a student skips a step, the corresponding bug survives — and that is the lesson.

---

## Why this project

| Property | What it exercises |
|---|---|
| Real security surface (open redirect, injection) | Subagent review — ch. 05 |
| Pure logic worth testing (slug, URL validation) | TDD — ch. 11 |
| A visible page | Playwright loop — ch. 04 |
| A new-ish stdlib API | MCP for docs — ch. 04 |
| Secrets in a `.env` | `permissions.deny` — ch. 10 |
| Non-obvious concurrency rule | `CLAUDE.md` gotchas — ch. 07 |

## Stack: zero dependencies

`node:http` + `node:sqlite` + `node:test`. No `npm install`, nothing to break in a classroom, no framework to learn. All three ship with Node.

```bash
node --version
```

- **Node 24+** — `node:sqlite` is stable, run plainly: `node src/server.js`
- **Node 22** — works but experimental: `node --experimental-sqlite src/server.js`
- **Node 20 or older** — swap in `express` + `better-sqlite3`; the steps are unchanged

Verified API surface (Node 22.23):

```js
const { DatabaseSync } = require('node:sqlite');
const db = new DatabaseSync(':memory:');
db.exec('CREATE TABLE t(slug TEXT PRIMARY KEY, clicks INTEGER DEFAULT 0)');
db.prepare('INSERT INTO t(slug) VALUES (?)').run('abc');
db.prepare('UPDATE t SET clicks = clicks + 1 WHERE slug = ?').run('abc');
db.prepare('SELECT * FROM t WHERE slug = ?').get('abc');
// → [Object: null prototype] { slug: 'abc', clicks: 1 }
```

---

## Step 0 — Setup and baseline

```bash
mkdir ~/lab-shortener && cd ~/lab-shortener && git init && claude
```

First two commands inside the session, before anything else:

```
/context
/cost
```

Write both numbers down. At the end of the project you will compare.

---

## Step 1 — Locks before code

Write `.claude/settings.json` **by hand**, before the model has written a single line:

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Edit(./.env)",
      "Write(./.env)"
    ]
  }
}
```

**Discussion:** this is different from writing *"never touch `.env`"* in `CLAUDE.md`. That is a request the model can reason its way around. This is a rule the harness enforces before the tool call ever runs. Rules for things that must not happen; `CLAUDE.md` for things that should happen. See ch. 10.

---

## Step 2 — `CLAUDE.md`

```
/init
```

Then edit the generated file and add the things that are **not discoverable from the code** — because there is no code yet:

```markdown
## Gotchas

- The click counter must be incremented in one atomic statement:
  `UPDATE links SET clicks = clicks + 1 WHERE slug = ?`
  Never SELECT-then-UPDATE — that is a lost-update race.
- Redirects are http/https only. Any other scheme (javascript:, data:, file:, vbscript:)
  must be rejected, both on write and on read.
- Slugs are exactly 7 chars from [a-zA-Z0-9]. On collision, retry — never overwrite.
- `/api/*` routes must be matched before the `/:slug` catch-all.

## Commands

- test: `node --test`
- run:  `node src/server.js`
```

**Discussion:** every line above is a fact that cannot be recovered by reading the repository. That is the test for whether something belongs in `CLAUDE.md` — see ch. 07.

---

## Step 3 — Plan Mode

`Shift+Tab` until the footer says `plan mode`. Then:

```
Design a URL shortener using only the Node standard library:
node:http for the server, node:sqlite for storage, no npm dependencies.

Routes:
  POST /api/links        { url } -> { slug, shortUrl }
  GET  /:slug            -> 302 redirect, increments the click counter
  GET  /api/stats/:slug  -> { url, clicks, createdAt }
  GET  /                 -> HTML dashboard

Give me the table schema, the file layout, and the failure modes you expect.
Do not write code.
```

**Discussion:** what did the plan miss? Common omissions worth surfacing in class:

- No index consideration on `slug` (it is the primary key — fine here, but did it say so or assume it?)
- Route ordering: `/:slug` swallowing `/api/links`
- What happens on a duplicate URL — new slug, or reuse the old one? The spec above doesn't say. Did the plan ask, or silently pick?

That last one is the real lesson: a good plan surfaces the ambiguity instead of resolving it quietly.

---

## Step 4 — MCP for documentation

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

Inside the session:

```
/mcp
```

Then:

```
use context7: what is the exact node:sqlite API in Node 22 and 24?
DatabaseSync, prepare, run, get, all — and which flag is needed on 22.
```

**Discussion:** `node:sqlite` landed recently and changed shape between releases. Ask the same question after `/clear` *without* context7 and compare. This is the clearest demonstration of why MCP docs beat model memory — see ch. 04.

---

## Step 5 — TDD on the pure logic

`/clear` first. Then send **three separate messages**. Do not combine them.

Message 1:

```
src/lib.js will export three functions: generateSlug(), validateUrl(input),
and normalizeUrl(input).

Write only the tests, in test/lib.test.js, using node:test and node:assert.

Cover:
- generateSlug: length 7, charset [a-zA-Z0-9] only, two consecutive calls differ
- validateUrl: accepts http and https; rejects javascript:, data:, file:,
  the empty string, whitespace, and malformed input
- normalizeUrl: strips the fragment, preserves the query string

Do not write the implementation.
```

Message 2:

```
Run `node --test`. Confirm the tests fail for the right reason —
missing function, not a syntax error in the test file.
```

Message 3:

```
Now write src/lib.js so every test passes. Do not modify the tests.
```

**Why three messages:** in one message the model writes the tests and the implementation together, and the tests end up describing the code it happened to write rather than the behaviour you asked for. Splitting them forces the tests to be written against the spec. See ch. 11.

**Discussion:** how many rounds did step 3 take? Compare against just saying *"write these three functions."*

---

## Step 6 — Database and server

```
Create src/db.js using node:sqlite. Table `links`:
  slug TEXT PRIMARY KEY, url TEXT NOT NULL,
  clicks INTEGER NOT NULL DEFAULT 0, created_at TEXT NOT NULL

Export: createLink(url), getLink(slug), recordClick(slug), getStats(slug).

createLink retries up to 5 times on slug collision.
recordClick must be atomic — see the Gotchas section in CLAUDE.md.
```

Then:

```
Create src/server.js with node:http. Routes per the plan.

GET /:slug must run validateUrl on the stored URL before emitting a
Location header — even though the URL was validated at write time.
```

**Discussion:** why validate twice? Because the database is a *different trust boundary* than the HTTP request. Rows can arrive by migration, by a fixture, by a second writer, by a future admin endpoint. Validation belongs at the boundary you are crossing right now, not once at the door of the building.

---

## Step 7 — Security subagent

Create `.claude/agents/security-reviewer.md`:

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities. MUST BE USED after any change to redirect handling, database queries, or user input parsing.
tools: Read, Grep, Glob
model: sonnet
color: red
---

You are a security engineer reviewing a URL shortener.

Check for:
1. Open redirect — any path from user input to a Location header
2. SQL injection — string interpolation into a query instead of a bound parameter
3. Missing rate limiting on link creation
4. SSRF — does the server ever fetch the target URL itself?
5. Slug enumeration — is the slug space small or sequential enough to walk?

For each finding report: file:line, severity, a concrete attack request, the fix.
Report nothing you cannot point to a specific line for.
```

Then:

```
@agent-security-reviewer review src/
```

**Discussion — two questions:**

1. The `tools` list is read-only. Add `Edit` and re-run. What changes? (It patches things instead of reporting them — and you stop reading the findings. Separating *finding* from *fixing* is deliberate.)
2. Did it flag rate limiting? That is an *absence*, not a line of code. Absences are the hardest class of finding for a reviewer that is told to cite a line. How would you reword the agent prompt to catch missing controls?

---

## Step 8 — Dashboard and the visual loop

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

```
Add an HTML dashboard at GET /. A form to shorten a URL, and a table of
recent links with their click counts. Inline CSS, no framework, no build step.
```

Now close the loop — the model must see its own output:

```
Start the server, open localhost:3000, take a screenshot.
Then resize to 375px wide and screenshot again.
Fix whatever is broken and re-screenshot until it is actually correct.
```

**Discussion:** compare against writing the CSS and letting the student check by hand. The loop is the whole point — an agent that can observe its output converges; one that cannot, guesses. See ch. 11.

---

## Step 9 — A Skill for commits

Create `.claude/skills/commit/SKILL.md`:

```markdown
---
description: Stages and commits with a conventional commit message derived from the actual diff. Use when asked to commit.
---

## Changes

!`git status --short`

## Diff

!`git diff HEAD`

## Instructions

Write a conventional commit message (feat/fix/docs/test/refactor/chore).
Subject under 60 characters, imperative mood.
Include a body only if the *why* is not obvious from the diff.
Never restate file names the diff already shows.
Stage and commit. Do not push.
```

```
/commit
```

**Discussion:** why is this a Skill and not a line in `CLAUDE.md`? Because it runs shell commands to gather fresh context at invocation time, and it is only relevant when committing. Loading it into every turn would be waste. See ch. 06.

---

## Step 10 — Read the diff yourself

```bash
git log --oneline
git show --stat HEAD
git show HEAD
```

**Classroom rule:** no commit counts unless the student has read the diff themselves. The whole handbook is about moving faster; none of it is about shipping code you have not looked at.

Finally, compare against the numbers from step 0:

```
/cost
/context
```

---

## The deliberate failures

Each row is a bug the project is designed to contain. It should be caught at exactly one place — and if that step was skipped, it ships.

| Failure | Caught by | Chapter | Lesson |
|---|---|---|---|
| Open redirect to `javascript:` | security subagent | 05 | Review is a separate job from writing |
| Slug collision overwrites a link | TDD test | 11 | Tests written before code, from the spec |
| Lost-update race on the click counter | `CLAUDE.md` gotcha | 07 | Knowledge that is not in the code |
| `/api/links` swallowed by `/:slug` | Plan Mode | 03 | Design review before 300 lines exist |
| Mobile layout breaks at 375px | Playwright loop | 04 | An agent that can see its output converges |
| `.env` gets modified | `permissions.deny` | 10 | A rule, not a request |
| Wrong `node:sqlite` API from memory | context7 MCP | 04 | Fresh docs beat model recall |

---

## Extensions, if there is time

Each one adds exactly one new concept and nothing else:

- **Rate limiting** on `POST /api/links` — an in-memory token bucket. Now re-run the security subagent: does it still flag the control as missing?
- **A `PreToolUse` hook** that rejects any `Edit` writing a raw SQL string containing `${`. Compare against relying on `CLAUDE.md` to say the same thing.
- **Custom slugs** — `POST /api/links { url, slug }`. New validation surface: reserved words (`api`, `admin`), length bounds, collisions with generated slugs.
- **Expiring links** — a `expires_at` column. Note how a new column ripples through every layer, and how well the tests you already wrote protect you.

---

## Assessment

| Skill | Evidence of mastery |
|---|---|
| Plan Mode | Got a plan and *corrected it* before approving |
| `CLAUDE.md` | Every line is something unreadable from the code |
| TDD | Tests failed for the right reason before implementation |
| Subagent | `description` written so it actually triggers |
| MCP | Used it where model memory was genuinely stale |
| Security | `.env` denied by rule; diff read before commit |
| Context | Knows when to `/clear` and why cost changed |

---

## Further reading

- Common workflows — https://code.claude.com/docs/en/common-workflows
- `node:sqlite` — https://nodejs.org/api/sqlite.html
- `node:test` — https://nodejs.org/api/test.html
- OWASP unvalidated redirects — https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html
