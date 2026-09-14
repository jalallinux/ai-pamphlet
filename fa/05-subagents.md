> [برگشت به فهرست](README.md) · قبلی: [۰۴ — MCP](04-mcp.md) · بعدی: [۰۶ — Skills](06-skills.md)

# فصل ۰۵ — Subagents (ایجنت‌ها)

## تعریف

Subagent یک **نمونهٔ Claude با پنجرهٔ context مستقل، system prompt اختصاصی و مجموعهٔ ابزار محدود** است که Claude اصلی برای یک کار تخصصی صدا می‌زند.

## چرا لازم است؟

| مشکل | راه‌حل Subagent |
|---|---|
| گشتن در ۵۰ فایل، context اصلی را پر می‌کند | subagent می‌گردد، فقط **نتیجه** برمی‌گرداند |
| ایجنت عمومی در امنیت متوسط است | subagent با system prompt تخصصی امنیت |
| می‌خواهید ایجنت فقط بخواند، ننویسد | `tools: Read, Grep, Glob` |
| چند کار موازی | چند subagent هم‌زمان |

**نکتهٔ اصلی: context isolation.** کار کثیف در پنجرهٔ جدا، خلاصهٔ تمیز به پنجرهٔ اصلی.

## محل فایل‌ها

| محل | اسکوپ | اولویت |
|---|---|---|
| `.claude/agents/` | پروژه (commit در git) | بالاتر |
| `~/.claude/agents/` | همهٔ پروژه‌های شما | پایین‌تر |
| `<plugin>/agents/` | جایی که پلاگین فعال است | پایین‌ترین |

## ساختار فایل

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

بدنهٔ فایل = system prompt آن ایجنت. ایجنت **فقط** همین را می‌بیند، نه system prompt اصلی Claude Code.

## فیلدهای frontmatter

اجباری: `name`, `description`.

| فیلد | مقدار |
|---|---|
| `name` | حروف کوچک + خط تیره. بدون `:` و بدون `-` در ابتدا |
| `description` | **چه زمانی** Claude باید این ایجنت را صدا بزند — مهم‌ترین فیلد |
| `tools` | لیست مجاز. ننویسید = همه را به ارث می‌برد |
| `disallowedTools` | لیست ممنوع |
| `model` | `sonnet` / `opus` / `haiku` / `inherit` |
| `permissionMode` | `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions` |
| `maxTurns` | سقف تعداد نوبت |
| `skills` | اسکیل‌هایی که از ابتدا بارگذاری شوند |
| `mcpServers` | سرورهای MCP مخصوص این ایجنت |
| `memory` | `user` / `project` / `local` |
| `color` | `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |
| `isolation` | `worktree` — روی کپی جدای repo کار کند |

## نوشتن `description` خوب

این فیلد تعیین می‌کند Claude خودکار سراغ ایجنت برود یا نه.

❌ `description: Reviews code`
✅ `description: Reviews code for security vulnerabilities. Use PROACTIVELY after any change to auth, session handling, or database queries.`

کلمهٔ `PROACTIVELY` یا `MUST BE USED` باعث می‌شود Claude زودتر delegate کند.

## فراخوانی

```
# زبان طبیعی — Claude خودش تصمیم می‌گیرد
Use the test-runner subagent to fix failing tests

# @-mention — تضمینی
@agent-code-reviewer look at the auth changes

# کل جلسه به‌عنوان یک ایجنت
claude --agent code-reviewer
```

دیدن ایجنت‌های در حال اجرا:

```
/tasks
```

## سه ایجنت آماده برای کلاس

### ۱. مرور امنیتی

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

### ۲. اجراکنندهٔ تست

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

### ۳. کاوشگر کدبیس (فقط خواندن)

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

## ترتیب انتخاب Claude

اگر هم ایجنت مناسب هست و هم می‌تواند خودش انجام دهد، Claude بر اساس `description` تصمیم می‌گیرد. اگر delegate نکرد، `description` را دقیق‌تر بنویسید یا با `@agent-name` صریح صدا بزنید.

## منابع

- Subagents — https://code.claude.com/docs/en/sub-agents
- مجموعهٔ ایجنت‌های جامعه — https://github.com/wshobson/agents
- مجموعهٔ دیگر — https://github.com/VoltAgent/awesome-claude-code-subagents
