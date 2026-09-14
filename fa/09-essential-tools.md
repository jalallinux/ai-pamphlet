> [برگشت به فهرست](README.md) · قبلی: [۰۸ — مهندسی پرامپت](08-prompt-engineering.md) · بعدی: [۱۰ — Hooks و Commands](10-hooks-and-commands.md)

# فصل ۰۹ — ۶ MCP و Agent ضروری

## بخش الف — شش سرور MCP

### ۱. Context7 — مستندات به‌روز

**مشکلی که حل می‌کند:** دانش مدل تاریخ انقضا دارد. کدی می‌نویسد که برای نسخهٔ پارسال درست بوده.

```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

یا remote:

```bash
claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp
```

استفاده:

```
use context7: چطور در Next.js 15 با App Router یک server action بنویسم؟
```

دو ابزار می‌دهد: `resolve-library-id` و `query-docs`.

> **نکتهٔ حرفه‌ای:** در `~/.claude/CLAUDE.md` بنویسید «برای هر سؤال دربارهٔ کتابخانه، اول از Context7 مستندات بگیر» تا خودکار شود.

منبع: https://github.com/upstash/context7

---

### ۲. Playwright — کنترل مرورگر

**مشکلی که حل می‌کند:** Claude نمی‌تواند ببیند رابط کاربری‌ای که ساخته چه شکلی است. با Playwright می‌بیند، کلیک می‌کند، تست می‌نویسد.

```bash
claude mcp add --scope user playwright -- npx -y @playwright/mcp@latest
```

استفاده:

```
localhost:3000 را باز کن، فرم لاگین را با test@example.com پر کن،
ثبت کن، و اسکرین‌شات نتیجه را بگیر
```

```
صفحه را در ۳۷۵px عرض ببین و مشکلات موبایل را گزارش بده
```

از accessibility tree استفاده می‌کند نه تصویر — سریع‌تر و دقیق‌تر از screenshot برای تعامل.

منبع: https://github.com/microsoft/playwright-mcp

---

### ۳. GitHub — مدیریت issue، PR، CI

```bash
claude mcp add --scope user --transport http github https://api.githubcopilot.com/mcp/
```

بعد داخل جلسه `/mcp` بزنید و OAuth را کامل کنید.

استفاده:

```
issue های باز با برچسب bug را لیست کن
PR #123 را بررسی کن و کامنت بگذار
چرا CI روی این شاخه fail شد؟
```

> جایگزین ساده‌تر: `gh` CLI. اگر نصب است، Claude بدون هیچ MCP می‌تواند با `Bash(gh ...)` کار کند — سبک‌تر و کم‌هزینه‌تر از نظر context.

منبع: https://github.com/github/github-mcp-server

---

### ۴. Filesystem — دسترسی به پوشه‌های خارج از پروژه

```bash
claude mcp add --scope user filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/Documents ~/Projects
```

فقط مسیرهایی که صریح می‌دهید قابل دسترسی‌اند. برای خواندن مرجع، دیتاست، یا مستندات بیرون از repo.

> در Claude Code معمولاً `--add-dir` کافی است. این MCP وقتی مفید است که کنترل دقیق‌تری بخواهید.

منبع: https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem

---

### ۵. Sequential Thinking — تفکر ساختاریافته

**مشکلی که حل می‌کند:** برای مسائل پیچیده، مدل زود نتیجه می‌گیرد. این سرور او را وادار به تجزیهٔ مرحله‌به‌مرحله با امکان بازبینی می‌کند.

```bash
claude mcp add --scope user sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

استفاده:

```
با sequential thinking: چرا این memory leak فقط در production رخ می‌دهد؟
```

منبع: https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking

---

### ۶. Serena — ناوبری معنایی کد

**مشکلی که حل می‌کند:** `grep` متن پیدا می‌کند؛ Serena *نماد* (symbol) پیدا می‌کند. یک Language Server اجرا می‌کند تا Claude بتواند به تعریف یک تابع بپرد، متدهای یک کلاس را فهرست کند، یا همهٔ فراخوان‌کننده‌ها را بیابد — و فقط بدنهٔ همان نماد را بخواند، نه کل فایل را.

```bash
claude mcp add --scope user serena -- uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant
```

استفاده:

```
کلاس AuthService را پیدا کن و فقط متد login آن را نشان بده
چه نمادهایی به createSession ارجاع می‌دهند؟
UserRepo را در کل کدبیس به AccountRepo تغییر نام بده
```

ابزارهای کلیدی: `find_symbol`، `find_referencing_symbols`، `get_symbols_overview`، `replace_symbol_body`، `rename_symbol`؛ به‌علاوهٔ یک سیستم **حافظهٔ پروژه** (`write_memory` / `read_memory`) که یادداشت‌ها را بین سشن‌ها در `.serena/memories/` نگه می‌دارد.

> **چرا برای هزینه مهم است:** در یک ریپوی بزرگ، «کل فایل را بخوان» هزاران توکن می‌سوزاند. Serena فقط همان تابعی را می‌خواند که پرسیده‌اید.

منبع: https://github.com/oraios/serena

---

### گزینه‌های افتخاری

| سرور | کار | نصب |
|---|---|---|
| **PostgreSQL/DBHub** | کوئری مستقیم دیتابیس | `npx -y @bytebase/dbhub` |
| **Figma** | خواندن دیزاین و تبدیل به کد | https://github.com/GLips/Figma-Context-MCP |
| **Sentry** | خواندن خطاهای production | https://mcp.sentry.dev/mcp |
| **Chrome DevTools** | دیباگ و performance profiling | https://github.com/ChromeDevTools/chrome-devtools-mcp |

---

## بخش ب — OpenSpec / Spec-Driven Development

این MCP نیست — **روش کار** است. ایدهٔ مرکزی:

> قبل از نوشتن کد، *مشخصات* را بنویس. مشخصات، منبع حقیقت است — نه کد.

جریان کار OpenSpec:

```
۱. proposal  → چه چیزی تغییر می‌کند و چرا
۲. spec      → رفتار دقیق، به زبان تأییدپذیر
۳. tasks     → تجزیه به قدم‌های کوچک
۴. implement → Claude بر اساس spec پیاده می‌کند
۵. archive   → spec تأییدشده به مخزن دانش پروژه می‌رود
```

نصب:

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

**چرا در کلاس بگویید:** بزرگ‌ترین شکست کدنویسی با AI، ابهام در نیازمندی است. spec آن را قبل از سوختن توکن حل می‌کند.

جایگزین‌ها:
- **Spec Kit** (GitHub) — https://github.com/github/spec-kit
- **Kiro** (AWS) — https://kiro.dev
- OpenSpec — https://github.com/Fission-AI/OpenSpec

---

## بخش ج — پنج Subagent ضروری

| ایجنت | کار | مدل پیشنهادی |
|---|---|---|
| `security-reviewer` | بررسی امنیتی بعد از تغییر auth/input/DB | `opus` |
| `test-runner` | اجرای تست و رفع fail | `sonnet` |
| `explorer` | جستجوی کدبیس، فقط خواندن | `haiku` |
| `code-reviewer` | مرور کیفیت قبل از commit | `sonnet` |
| `doc-writer` | نوشتن README و مستند API | `sonnet` |

نمونهٔ کامل سه تای اول در [فصل ۰۵](05-subagents.md) آمده.

مجموعه‌های آماده:
- https://github.com/wshobson/agents
- https://github.com/VoltAgent/awesome-claude-code-subagents

---

## پیکربندی پیشنهادی برای شروع

`.mcp.json` در ریشهٔ پروژه (قابل commit):

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

> **قانون:** با ۲ تا ۳ سرور شروع کنید. هر سرور، تعریف ابزارهایش را به context اضافه می‌کند. ده سرور نصب‌شده = context شلوغ + انتخاب ضعیف‌تر مدل + هزینهٔ بیشتر.

---

## منابع

- MCP در Claude Code — https://code.claude.com/docs/en/mcp
- فهرست رسمی سرورها — https://github.com/modelcontextprotocol/servers
- رجیستری MCP — https://registry.modelcontextprotocol.io
- Awesome MCP Servers — https://github.com/punkpeye/awesome-mcp-servers
