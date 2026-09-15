> [برگشت به فهرست](README.md) · قبلی: [۱۲ — Cheatsheet](12-cheatsheet.md) · بعدی: [۱۴ — پروژهٔ نمونه](14-sample-project.md)

# فصل ۱۳ — تمرین‌های کلاس

هر تمرین ۱۰ تا ۲۰ دقیقه. ترتیب مهم است.

---

## تمرین ۱ — اولین جلسه (۱۰ دقیقه)

**هدف:** آشنایی با حلقهٔ کار.

```bash
mkdir ~/lab-01 && cd ~/lab-01
git init
claude
```

داخل جلسه:

```
یک اسکریپت پایتون بساز که یک فایل CSV بگیرد و میانگین هر ستون عددی را چاپ کند.
یک فایل CSV نمونه هم برای تست بساز.
بعد اجرا کن و نتیجه را نشان بده.
```

بعد:

```
/cost
/context
```

**بحث کلاس:** چند بار از شما اجازه گرفت؟ چرا؟

---

## تمرین ۲ — Plan Mode (۱۵ دقیقه)

**هدف:** حس کردن تفاوت بررسی قبل از نوشتن.

روی یک پروژهٔ واقعی (یا clone یک repo):

```bash
git clone https://github.com/expressjs/express ~/lab-02
cd ~/lab-02
claude
```

`Shift+Tab` تا برسد به plan mode. سپس:

```
این کدبیس را بررسی کن و نقشهٔ اضافه کردن یک middleware
برای rate limiting بده. کد ننویس.
```

**بحث:** نقشه چقدر دقیق بود؟ چه چیزی در آن غلط بود که اگر مستقیم کد می‌زد، ۳۰۰ خط دور می‌ریختید؟

---

## تمرین ۳ — نوشتن `CLAUDE.md` (۱۵ دقیقه)

**هدف:** تفاوت context خوب و بد.

قسمت الف — بدون فایل:

```
یک endpoint جدید برای لیست کاربران اضافه کن.
```

نتیجه را ببینید. احتمالاً سبک کد با بقیهٔ پروژه نمی‌خواند.

قسمت ب:

```
/init
```

سپس فایل تولیدشده را ویرایش کنید — بخش **Gotchas** اضافه کنید:

```markdown
## Gotchas
- تمام endpoint ها باید از `asyncHandler` در `src/utils/` استفاده کنند
- خطا هرگز مستقیم throw نشود — از `AppError` استفاده شود
- هر endpoint جدید باید تست integration داشته باشد
```

`/clear` بزنید و همان درخواست را تکرار کنید.

**بحث:** تفاوت خروجی.

---

## تمرین ۴ — نصب MCP و استفاده (۱۵ دقیقه)

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
claude mcp list
```

داخل جلسه:

```
/mcp
```

سپس:

```
use context7: در Next.js 15 چطور یک Server Action با اعتبارسنجی Zod بنویسم؟
```

بعد بدون context7 همان سؤال را بپرسید (`/clear` اول).

**بحث:** کدام جواب به‌روزتر بود؟ چطور تشخیص می‌دهید مدل دارد از دانش قدیمی حدس می‌زند؟

---

## تمرین ۵ — ساخت Subagent (۲۰ دقیقه)

بسازید `.claude/agents/security-reviewer.md`:

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

یک فایل عمداً ناامن بسازید:

```javascript
// src/users.js
app.get('/user', (req, res) => {
  const q = `SELECT * FROM users WHERE id = ${req.query.id}`;
  db.query(q, (e, r) => res.json(r));
});
```

سپس:

```
@agent-security-reviewer بررسی کن src/users.js
```

**بحث:** ایجنت چه چیزی پیدا کرد؟ حالا `tools` را به `Read, Edit` تغییر دهید — چه فرقی می‌کند؟

---

## تمرین ۶ — ساخت Skill (۱۵ دقیقه)

بسازید `.claude/skills/standup/SKILL.md`:

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

**بحث:** چرا این Skill است و نه `CLAUDE.md`؟

---

## تمرین ۷ — Hook (۱۵ دقیقه)

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

جلسه را restart کنید، بعد:

```
یک متغیر جدید به فایل .env اضافه کن
```

**بحث:** تفاوت این با نوشتن «هرگز `.env` را تغییر نده» در `CLAUDE.md` چیست؟

---

## تمرین ۸ — TDD کامل (۲۰ دقیقه)

**هدف:** تجربهٔ قوی‌ترین الگو.

```
مرحلهٔ ۱:
برای یک تابع `parseDuration(input: string): number` تست بنویس که
رشته‌هایی مثل "1h30m"، "45s"، "2d" را به ثانیه تبدیل کند.
موارد نامعتبر باید خطا بدهند. پیاده‌سازی ننویس.

مرحلهٔ ۲:
تست‌ها را اجرا کن و تأیید کن که به دلیل درست fail می‌شوند.

مرحلهٔ ۳:
حالا پیاده‌سازی کن تا همه pass شوند. تست‌ها را تغییر نده.
```

**بحث:** چند بار نیاز به تکرار شد؟ مقایسه کنید با وقتی مستقیم می‌گفتید «این تابع را بنویس».

---

## تمرین ۹ — Playwright و حلقهٔ بصری (۲۰ دقیقه)

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

یک صفحهٔ HTML ساده بسازید و سرو کنید. سپس:

```
localhost:3000 را باز کن و اسکرین‌شات بگیر.
این صفحه در موبایل (375px) مشکل دارد. عرض را عوض کن، ببین،
و CSS را درست کن. دوباره اسکرین‌شات بگیر تا مطمئن شوی درست شد.
```

**بحث:** تفاوت با وقتی فقط CSS را بدون دیدن نتیجه می‌نویسد.

---

## تمرین ۱۰ — پروژهٔ پایانی (۴۵ دقیقه)

**هدف:** همه چیز با هم.

بسازید: یک CLI ساده برای مدیریت یادداشت (add / list / search / delete) با ذخیره در SQLite.

الزامات:
1. `CLAUDE.md` را **قبل** از شروع کد بنویسید
2. با Plan Mode شروع کنید
3. با TDD پیاده کنید
4. یک subagent برای review بسازید و اجرا کنید
5. یک Skill برای commit استاندارد بسازید
6. `permissions.deny` برای `.env` تنظیم کنید
7. خودتان diff را بخوانید، بعد commit کنید

**ارائه:** هر شاگرد `CLAUDE.md` خودش را نشان دهد و بگوید چرا این خطوط را نوشته.

---

## معیار ارزیابی

| مهارت | نشانهٔ تسلط |
|---|---|
| مدیریت context | می‌داند کی `/clear` بزند |
| Plan Mode | قبل از کد بزرگ، نقشه می‌گیرد |
| `CLAUDE.md` | چیزی می‌نویسد که از کد پیدا نیست |
| MCP | با ۲-۳ سرور کار می‌کند، نه ۱۵ تا |
| Subagent | `description` می‌نویسد که واقعاً trigger شود |
| امنیت | `.env` را deny کرده، diff را می‌خواند |
| پرامپت | محدودیت و معیار موفقیت می‌دهد |

---

## منابع بیشتر

- ورک‌فلوهای رایج — https://code.claude.com/docs/en/common-workflows
- آموزش تعاملی پرامپت — https://github.com/anthropics/prompt-eng-interactive-tutorial
- Anthropic Cookbook — https://github.com/anthropics/anthropic-cookbook
- Awesome Claude Code — https://github.com/hesreallyhim/awesome-claude-code
