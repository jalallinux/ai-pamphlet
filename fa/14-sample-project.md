> [برگشت به فهرست](README.md) · قبلی: [۱۳ — تمرین‌های کلاس](13-lab-exercises.md)

# فصل ۱۴ — پروژهٔ نمونه: کوتاه‌کنندهٔ لینک

یک پروژه، ده مرحله، و هر فصل این جزوه یک بار استفاده می‌شود. زمان: ۴۵ تا ۹۰ دقیقه.

هدف خودِ کوتاه‌کنندهٔ لینک نیست. هدف این است که هر مرحله یک **شکست عمدی** دارد که قرار است یک قابلیت متفاوت Claude Code آن را بگیرد. اگر شاگرد مرحله‌ای را رد کند، باگ متناظرش زنده می‌ماند — و همین خودش درس است.

---

## چرا این پروژه

| ویژگی | چه چیزی را تمرین می‌دهد |
|---|---|
| سطح امنیتی واقعی (open redirect، injection) | مرور با Subagent — فصل ۰۵ |
| منطق خالصِ ارزشِ تست داشتن (slug، اعتبارسنجی URL) | TDD — فصل ۱۱ |
| یک صفحهٔ دیدنی | حلقهٔ Playwright — فصل ۰۴ |
| یک API نسبتاً تازه در stdlib | MCP برای مستندات — فصل ۰۴ |
| وجود `.env` | `permissions.deny` — فصل ۱۰ |
| یک قانون همزمانی که از کد پیدا نیست | Gotchas در `CLAUDE.md` — فصل ۰۷ |

## استک: صفر وابستگی

`node:http` + `node:sqlite` + `node:test`. بدون `npm install`، بدون چیزی که سر کلاس خراب شود، بدون فریم‌ورکی که باید یاد گرفته شود. هر سه با خودِ Node می‌آیند.

```bash
node --version
```

- **Node 24 و بالاتر** — `node:sqlite` پایدار است، ساده اجرا کنید: `node src/server.js`
- **Node 22** — کار می‌کند ولی آزمایشی: `node --experimental-sqlite src/server.js`
- **Node 20 و پایین‌تر** — به جایش `express` + `better-sqlite3`؛ مراحل تغییری نمی‌کند

سطح API تأییدشده (روی Node 22.23):

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

## مرحله ۰ — راه‌اندازی و خط پایه

```bash
mkdir ~/lab-shortener && cd ~/lab-shortener && git init && claude
```

دو دستور اول داخل جلسه، قبل از هر چیز دیگر:

```
/context
/cost
```

هر دو عدد را یادداشت کنید. آخر پروژه مقایسه می‌کنید.

---

## مرحله ۱ — قفل‌ها، قبل از کد

فایل `.claude/settings.json` را **دستی** بنویسید، قبل از اینکه مدل حتی یک خط نوشته باشد:

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

**بحث:** این با نوشتن «هرگز `.env` را دست نزن» در `CLAUDE.md` فرق دارد. آن یک درخواست است که مدل می‌تواند با استدلال دورش بزند. این یک قانون است که هارنس قبل از اجرای ابزار اعمال می‌کند. قانون برای چیزهایی که **نباید** اتفاق بیفتد؛ `CLAUDE.md` برای چیزهایی که **باید**. فصل ۱۰.

---

## مرحله ۲ — `CLAUDE.md`

```
/init
```

سپس فایل تولیدشده را ویرایش کنید و چیزهایی را اضافه کنید که **از روی کد قابل کشف نیستند** — چون هنوز کدی وجود ندارد:

```markdown
## Gotchas

- شمارندهٔ کلیک باید در یک statement اتمیک زیاد شود:
  `UPDATE links SET clicks = clicks + 1 WHERE slug = ?`
  هرگز SELECT سپس UPDATE — این یک lost-update race است.
- redirect فقط http و https. هر scheme دیگر (javascript:, data:, file:, vbscript:)
  باید رد شود، هم موقع نوشتن هم موقع خواندن.
- slug دقیقاً ۷ کاراکتر از [a-zA-Z0-9]. روی برخورد retry شود — هرگز overwrite نشود.
- مسیرهای `/api/*` باید قبل از catch-all یعنی `/:slug` تطبیق داده شوند.

## Commands

- test: `node --test`
- run:  `node src/server.js`
```

**بحث:** هر خط بالا یک واقعیت است که با خواندن مخزن به دست نمی‌آید. همین معیار تشخیص است که چه چیزی به `CLAUDE.md` تعلق دارد — فصل ۰۷.

---

## مرحله ۳ — Plan Mode

`Shift+Tab` بزنید تا در پایین بنویسد `plan mode`. سپس:

```
یک کوتاه‌کنندهٔ لینک طراحی کن که فقط از کتابخانهٔ استاندارد Node استفاده کند:
node:http برای سرور، node:sqlite برای ذخیره‌سازی، بدون هیچ وابستگی npm.

مسیرها:
  POST /api/links        { url } -> { slug, shortUrl }
  GET  /:slug            -> 302 redirect و افزایش شمارندهٔ کلیک
  GET  /api/stats/:slug  -> { url, clicks, createdAt }
  GET  /                 -> داشبورد HTML

schema جدول، ساختار فایل‌ها، و حالت‌های شکستی که انتظار داری را بگو.
کد ننویس.
```

**بحث:** نقشه چه چیزی را جا انداخت؟ مواردی که معمولاً سر کلاس ارزش مطرح کردن دارند:

- چیزی دربارهٔ index روی `slug` نگفت (اینجا primary key است و مشکلی نیست — ولی گفت یا فرض کرد؟)
- ترتیب مسیرها: `/:slug` که `/api/links` را می‌بلعد
- اگر همان URL دوباره ثبت شود چه؟ slug جدید یا استفادهٔ مجدد از قبلی؟ در مشخصات بالا نیامده. نقشه پرسید یا بی‌صدا یکی را انتخاب کرد؟

مورد آخر درس اصلی است: نقشهٔ خوب ابهام را رو می‌آورد، نه اینکه ساکت حلش کند.

---

## مرحله ۴ — MCP برای مستندات

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

داخل جلسه:

```
/mcp
```

سپس:

```
use context7: API دقیق node:sqlite در Node 22 و 24 چیست؟
DatabaseSync، prepare، run، get، all — و روی ۲۲ چه flag ای لازم است.
```

**بحث:** `node:sqlite` تازه اضافه شده و بین نسخه‌ها شکلش عوض شده. بعد از `/clear` همان سؤال را **بدون** context7 بپرسید و مقایسه کنید. این واضح‌ترین نمایش این است که چرا مستندات MCP از حافظهٔ مدل بهتر است — فصل ۰۴.

---

## مرحله ۵ — TDD روی منطق خالص

اول `/clear`. بعد **سه پیام جدا** بفرستید. یکی نکنید.

پیام ۱:

```
فایل src/lib.js سه تابع export خواهد کرد: generateSlug()، validateUrl(input)
و normalizeUrl(input).

فقط تست بنویس، در test/lib.test.js، با node:test و node:assert.

پوشش:
- generateSlug: طول ۷، فقط charset [a-zA-Z0-9]، دو فراخوانی پشت هم متفاوت باشند
- validateUrl: http و https را قبول کند؛ javascript:، data:، file:،
  رشتهٔ خالی، فاصله، و ورودی بدشکل را رد کند
- normalizeUrl: fragment را حذف کند، query string را نگه دارد

پیاده‌سازی ننویس.
```

پیام ۲:

```
`node --test` را اجرا کن. تأیید کن که تست‌ها به دلیل درست fail می‌شوند —
نبودِ تابع، نه خطای syntax در خود فایل تست.
```

پیام ۳:

```
حالا src/lib.js را بنویس تا همهٔ تست‌ها pass شوند. تست‌ها را تغییر نده.
```

**چرا سه پیام:** در یک پیام، مدل تست و پیاده‌سازی را با هم می‌نویسد و تست‌ها در نهایت کدی را توصیف می‌کنند که اتفاقی نوشته، نه رفتاری که شما خواسته‌اید. جدا کردن، تست‌ها را مجبور می‌کند مقابل مشخصات نوشته شوند. فصل ۱۱.

**بحث:** مرحلهٔ ۳ چند دور طول کشید؟ مقایسه کنید با وقتی فقط می‌گفتید «این سه تابع را بنویس».

---

## مرحله ۶ — دیتابیس و سرور

```
فایل src/db.js را با node:sqlite بساز. جدول `links`:
  slug TEXT PRIMARY KEY, url TEXT NOT NULL,
  clicks INTEGER NOT NULL DEFAULT 0, created_at TEXT NOT NULL

export کن: createLink(url)، getLink(slug)، recordClick(slug)، getStats(slug).

createLink روی برخورد slug تا ۵ بار retry کند.
recordClick باید اتمیک باشد — به بخش Gotchas در CLAUDE.md نگاه کن.
```

سپس:

```
فایل src/server.js را با node:http بساز. مسیرها طبق نقشه.

مسیر GET /:slug باید قبل از فرستادن هدر Location، روی URL ذخیره‌شده
validateUrl را اجرا کند — حتی با اینکه URL موقع نوشتن اعتبارسنجی شده بود.
```

**بحث:** چرا دو بار اعتبارسنجی؟ چون دیتابیس یک **مرز اعتماد متفاوت** با درخواست HTTP است. سطرها ممکن است از راه migration، از fixture، از یک نویسندهٔ دوم، یا از یک endpoint مدیریتی در آینده آمده باشند. اعتبارسنجی به مرزی تعلق دارد که همین الان از آن عبور می‌کنید، نه یک بار دمِ درِ ساختمان.

---

## مرحله ۷ — Subagent امنیتی

فایل `.claude/agents/security-reviewer.md`:

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

سپس:

```
@agent-security-reviewer بررسی کن src/
```

**بحث — دو سؤال:**

۱. فهرست `tools` فقط خواندنی است. `Edit` را اضافه کنید و دوباره اجرا کنید. چه فرقی می‌کند؟ (به جای گزارش دادن، وصله می‌زند — و شما دیگر یافته‌ها را نمی‌خوانید. جدا کردن **پیدا کردن** از **درست کردن** عمدی است.)

۲. آیا نبودِ rate limiting را گزارش کرد؟ آن یک **غیاب** است، نه یک خط کد. غیاب‌ها سخت‌ترین دستهٔ یافته برای مروری هستند که به آن گفته‌اید فقط چیزی بگو که بتوانی خط نشانش دهی. پرامپت ایجنت را چطور بازنویسی می‌کنید که کنترل‌های غایب را هم بگیرد؟

---

## مرحله ۸ — داشبورد و حلقهٔ بصری

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

```
یک داشبورد HTML در GET / اضافه کن. یک فرم برای کوتاه کردن URL، و یک جدول
از لینک‌های اخیر با تعداد کلیکشان. CSS به صورت inline، بدون فریم‌ورک، بدون build.
```

حالا حلقه را ببندید — مدل باید خروجی خودش را ببیند:

```
سرور را اجرا کن، localhost:3000 را باز کن و اسکرین‌شات بگیر.
سپس عرض را به ۳۷۵ پیکسل تغییر بده و دوباره اسکرین‌شات بگیر.
هر چه شکسته است درست کن و دوباره اسکرین‌شات بگیر تا واقعاً درست شود.
```

**بحث:** مقایسه کنید با حالتی که CSS را می‌نویسد و شاگرد دستی چک می‌کند. حلقه تمام ماجراست — ایجنتی که بتواند خروجی‌اش را ببیند همگرا می‌شود؛ آنکه نتواند، حدس می‌زند. فصل ۱۱.

---

## مرحله ۹ — یک Skill برای commit

فایل `.claude/skills/commit/SKILL.md`:

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

**بحث:** چرا این یک Skill است و نه یک خط در `CLAUDE.md`؟ چون برای جمع کردن context تازه، لحظهٔ فراخوانی دستور shell اجرا می‌کند، و فقط موقع commit مربوط است. بارگذاری آن در هر نوبت اتلاف است. فصل ۰۶.

---

## مرحله ۱۰ — خودتان diff را بخوانید

```bash
git log --oneline
git show --stat HEAD
git show HEAD
```

**قانون کلاس:** هیچ commit ای قبول نیست مگر اینکه شاگرد خودش diff را خوانده باشد. تمام این جزوه دربارهٔ سریع‌تر حرکت کردن است؛ هیچ بخشی از آن دربارهٔ فرستادن کدی نیست که نگاهش نکرده‌اید.

در آخر، با اعداد مرحلهٔ ۰ مقایسه کنید:

```
/cost
/context
```

---

## شکست‌های عمدی

هر سطر یک باگ است که پروژه عمداً آن را در خود دارد. باید دقیقاً در یک جا گرفته شود — و اگر آن مرحله رد شده باشد، باگ منتشر می‌شود.

| شکست | کجا گرفته می‌شود | فصل | درس |
|---|---|---|---|
| open redirect به `javascript:` | subagent امنیتی | ۰۵ | مرور، کارِ جدایی از نوشتن است |
| برخورد slug باعث overwrite | تست TDD | ۱۱ | تست قبل از کد، از روی مشخصات |
| lost-update روی شمارندهٔ کلیک | Gotcha در `CLAUDE.md` | ۰۷ | دانشی که در کد نیست |
| بلعیده شدن `/api/links` توسط `/:slug` | Plan Mode | ۰۳ | مرور طراحی قبل از وجود ۳۰۰ خط |
| شکستن layout موبایل در ۳۷۵ پیکسل | حلقهٔ Playwright | ۰۴ | ایجنتی که خروجی‌اش را ببیند همگرا می‌شود |
| تغییر خوردن `.env` | `permissions.deny` | ۱۰ | قانون، نه درخواست |
| API غلط `node:sqlite` از حافظه | context7 MCP | ۰۴ | مستندات تازه بهتر از حافظهٔ مدل |

---

## توسعه‌ها، اگر وقت بود

هر کدام دقیقاً یک مفهوم جدید اضافه می‌کند و نه بیشتر:

- **Rate limiting** روی `POST /api/links` — یک token bucket در حافظه. حالا subagent امنیتی را دوباره اجرا کنید: هنوز نبودِ این کنترل را گزارش می‌دهد؟
- **یک hook از نوع `PreToolUse`** که هر `Edit` نویسندهٔ رشتهٔ SQL خامِ حاویِ `${` را رد کند. مقایسه کنید با تکیه بر `CLAUDE.md` برای گفتن همان حرف.
- **slug سفارشی** — `POST /api/links { url, slug }`. سطح اعتبارسنجی جدید: کلمات رزرو (`api`، `admin`)، کران طول، برخورد با slug های تولیدشده.
- **لینک منقضی‌شونده** — ستون `expires_at`. ببینید یک ستون جدید چطور در همهٔ لایه‌ها موج می‌اندازد، و تست‌هایی که قبلاً نوشتید چقدر از شما محافظت می‌کنند.

---

## ارزیابی

| مهارت | نشانهٔ تسلط |
|---|---|
| Plan Mode | نقشه گرفت و **اصلاحش کرد** قبل از تأیید |
| `CLAUDE.md` | هر خطش چیزی است که از کد خوانده نمی‌شود |
| TDD | تست‌ها قبل از پیاده‌سازی، به دلیل درست fail شدند |
| Subagent | `description` طوری نوشته که واقعاً trigger شود |
| MCP | جایی استفاده کرد که حافظهٔ مدل واقعاً کهنه بود |
| امنیت | `.env` با قانون deny شده؛ diff قبل از commit خوانده شده |
| context | می‌داند کی `/clear` بزند و چرا هزینه تغییر کرد |

---

## منابع بیشتر

- ورک‌فلوهای رایج — https://code.claude.com/docs/en/common-workflows
- `node:sqlite` — https://nodejs.org/api/sqlite.html
- `node:test` — https://nodejs.org/api/test.html
- OWASP unvalidated redirects — https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html
