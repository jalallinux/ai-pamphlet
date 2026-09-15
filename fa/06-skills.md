> [برگشت به فهرست](README.md) · قبلی: [۰۵ — Subagents](05-subagents.md) · بعدی: [۰۷ — CLAUDE.md و AGENTS.md](07-claude-md-agents-md.md)

# فصل ۰۶ — Skills (اسکیل‌ها)

## تعریف

Skill یک **پوشه حاوی دستورالعمل** است که Claude فقط **وقتی لازم شد** بارگذاری می‌کند.

تفاوت بنیادی با `CLAUDE.md`:

| | `CLAUDE.md` | Skill |
|---|---|---|
| چه زمانی بارگذاری می‌شود | **همیشه**، هر جلسه | فقط هنگام نیاز |
| هزینهٔ context | دائمی | صفر تا وقتی فعال نشده |
| مناسب برای | قوانین کوتاه همیشگی | رویهٔ چندمرحله‌ای |

Anthropic این را **Progressive Disclosure** می‌نامد: Claude همیشه فقط نام + `description` را می‌بیند؛ محتوای کامل وقتی بارگذاری می‌شود که مرتبط باشد.

## محل فایل‌ها

| اسکوپ | مسیر |
|---|---|
| شخصی | `~/.claude/skills/<skill-name>/SKILL.md` |
| پروژه | `.claude/skills/<skill-name>/SKILL.md` |
| پلاگین | `<plugin>/skills/<skill-name>/SKILL.md` |

**نام دستور از نام پوشه می‌آید**، نه از فیلد `name`.
`deploy-staging/SKILL.md` → `/deploy-staging`

## کمینه‌ترین Skill ممکن

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

فراخوانی: `/summarize-changes` — یا فقط بپرسید «چی تغییر کرده؟»

## تزریق خروجی دستور شل

`` !`command` `` **قبل** از رسیدن محتوا به مدل اجرا می‌شود و خروجی جایگزین می‌شود:

```markdown
وضعیت فعلی: !`git status --short`
شاخهٔ فعلی: !`git branch --show-current`
```

⚠️ اگر دستور exit code غیرصفر بدهد، کل اسکیل abort می‌شود. برای دستورهایی مثل `grep` که طبیعتاً ممکن است ۱ برگردانند، `|| true` بگذارید.

## فیلدهای frontmatter

همه اختیاری‌اند؛ `description` عملاً لازم است.

| فیلد | کار |
|---|---|
| `description` | **مهم‌ترین** — چه کاری و چه زمانی |
| `when_to_use` | عبارت‌های trigger اضافه |
| `argument-hint` | راهنمای آرگومان، مثلاً `[issue-number]` |
| `allowed-tools` | مجوز ابزار فقط برای همین نوبت |
| `disable-model-invocation: true` | فقط شما صدا بزنید، Claude خودکار نه |
| `user-invocable: false` | فقط Claude صدا بزند |
| `context: fork` | داخل subagent اجرا شود، بدون تاریخچهٔ گفتگو |
| `model` | مدل خاص |
| `paths` | فقط برای فایل‌های منطبق فعال شود |

## آرگومان‌ها

```markdown
---
argument-hint: [issue-number]
---

Fix GitHub issue #$1. Read it with `gh issue view $1`, then implement the fix.
```

```
/fix-issue 234
```

متغیرهای در دسترس: `$ARGUMENTS`, `$1`, `$2`, `${CLAUDE_SKILL_DIR}`, `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_SESSION_ID}`

## فایل‌های کمکی

```
my-skill/
├── SKILL.md          # اجباری — زیر ۵۰۰ خط نگه دارید
├── reference.md      # وقتی لازم شد خوانده می‌شود
├── examples.md
└── scripts/
    └── helper.py     # اجرا می‌شود، بارگذاری نمی‌شود
```

در `SKILL.md` به آن‌ها ارجاع دهید تا Claude بداند کی بخواند:

```markdown
For the full API reference, read `reference.md` in this skill's directory.
```

## اسکیل با اسکریپت

```markdown
---
name: render-chart
description: Render a chart from a CSV file
allowed-tools: Bash(${CLAUDE_SKILL_DIR}/scripts/render.sh *)
---

Run `${CLAUDE_SKILL_DIR}/scripts/render.sh <csv-file>` to render the chart.
```

استفاده از همان متغیر در هر دو جا → اسکریپت بدون prompt اجازه اجرا می‌شود.

## سه اسکیل عملی برای کلاس

### ۱. کامیت استاندارد

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

### ۲. بررسی PR

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

### ۳. graphify — گراف دانش کدبیس

نمونه‌ای از یک **اسکیل بزرگِ نصب‌شده**، نه اسکیلی که خودتان می‌نویسید. دقیقاً نشان می‌دهد progressive disclosure چه چیزی به شما می‌دهد: `SKILL.md` یک رویهٔ طولانی به‌همراه پوشهٔ `references/` است، و هیچ‌کدام وارد context نمی‌شوند مگر وقتی دربارهٔ کدبیس سؤال کنید.

```
~/.claude/skills/graphify/
├── SKILL.md
└── references/
```

چه می‌کند: یک پوشه از فایل‌ها (کد، مستندات، مقاله، تصویر، ویدیو) را به یک گراف دانش ماندگار تبدیل می‌کند — گره، یال، و تشخیص جامعه (community detection) — و آن را در `graphify-out/` کنار پروژه می‌نویسد.

```bash
/graphify                      # ساخت گراف از دایرکتوری جاری
/graphify <path> --update      # فقط فایل‌های تغییرکرده را دوباره استخراج کن
/graphify query "احراز هویت چطور کار می‌کند؟"
/graphify path "AuthModule" "Database"
/graphify explain "SessionStore"
```

وقتی `graphify-out/graph.json` وجود داشته باشد، یک سؤال ساده مثل «فلوی لاگین چطور کار می‌کند؟» از روی گراف پاسخ داده می‌شود، نه با خواندن دوبارهٔ فایل‌ها.

هر یال یک برچسب حسابرسی دارد — `EXTRACTED` (در منبع پیدا شده)، `INFERRED` (حدس مدل)، یا `AMBIGUOUS` — تا بتوانید واقعیت را از استنتاج تشخیص دهید.

مکمل [Serena](09-essential-tools.md) است: Serena به سؤال‌های *دقیق* دربارهٔ نمادها پاسخ می‌دهد (چه کسی این تابع را صدا می‌زند)؛ graphify به سؤال‌های *ساختاری* (کدام بخش‌های سیستم با هم خوشه می‌شوند).

> **نکتهٔ هزینه:** ساخت اولیه کل corpus را یک‌بار می‌خواند — گران است. کوئری‌های بعدی ارزان‌اند. یک‌بار بساز، بارها کوئری بزن.

اسکیل در اولین اجرا CLI خودش را نصب می‌کند:

```bash
uv tool install graphifyy
```

منبع: https://github.com/Graphify-Labs/graphify

## نصب اسکیل‌های آماده

```bash
# پلاگین مارکت‌پلیس Anthropic
/plugin marketplace add anthropics/claude-code
/plugin install <name>
```

## چه زمانی Skill، چه زمانی Subagent، چه زمانی CLAUDE.md؟

| نیاز | ابزار |
|---|---|
| «همیشه ۲ فاصله تورفتگی» | `CLAUDE.md` |
| «مراحل دیپلوی به staging» | Skill |
| «یک نفر متخصص امنیت که جدا بررسی کند» | Subagent |
| «قبل از هر commit lint بزن» (اجباری) | Hook → [فصل ۱۰](10-hooks-and-commands.md) |

## منابع

- Skills در Claude Code — https://code.claude.com/docs/en/skills
- بهترین شیوه‌های نوشتن Skill — https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices
- اسکیل‌های رسمی Anthropic — https://github.com/anthropics/skills
- مقالهٔ مهندسی Anthropic — https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
