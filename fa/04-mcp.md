> [برگشت به فهرست](README.md) · قبلی: [۰۳ — مودها](03-modes.md) · بعدی: [۰۵ — Subagents](05-subagents.md)

# فصل ۰۴ — MCP (Model Context Protocol)

## تعریف

MCP یک **پروتکل باز** است که ابزارهای خارجی را به مدل وصل می‌کند. استعارهٔ رسمی: «USB-C برای AI».

بدون MCP: برای هر ترکیب (هارنس × ابزار) باید کد جدا نوشت → M×N.
با MCP: هر ابزار یک سرور MCP می‌نویسد، هر هارنس یک کلاینت → M+N.

معرفی‌شده توسط Anthropic در نوامبر ۲۰۲۴، حالا استاندارد صنعتی (OpenAI, Google, Microsoft همه پشتیبانی می‌کنند).

## سرور MCP سه چیز می‌دهد

| نوع | چیست | مثال |
|---|---|---|
| **Tools** | تابعی که مدل صدا می‌زند | `query_database`, `browser_click` |
| **Resources** | داده‌ای که مدل می‌خواند | فایل، جدول، داکیومنت |
| **Prompts** | قالب آمادهٔ قابل فراخوانی توسط کاربر | `/review-pr` |

## انواع Transport

| نوع | کجا اجرا می‌شود | مناسب برای |
|---|---|---|
| `stdio` | پروسهٔ محلی روی دستگاه شما | ابزار محلی، دسترسی فایل، دیتابیس لوکال |
| `http` | سرور راه دور (توصیه‌شده) | SaaS، سرویس ابری، پشتیبانی OAuth |
| `sse` | راه دور (منسوخ) | قدیمی — از http استفاده کنید |

## افزودن سرور

### stdio (محلی)

`--` اجباری است: فلگ‌های Claude را از دستور سرور جدا می‌کند.

```bash
claude mcp add <name> -- <command> [args...]
```

مثال واقعی:

```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp
claude mcp add playwright -- npx -y @playwright/mcp@latest
claude mcp add --env AIRTABLE_API_KEY=YOUR_KEY airtable -- npx -y airtable-mcp-server
```

### http (راه دور)

```bash
claude mcp add --transport http <name> <url>
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### با JSON

```bash
claude mcp add-json weather-api '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'
```

## اسکوپ‌ها (نکتهٔ حیاتی کلاس)

```bash
claude mcp add --scope <scope> <name> -- <command>
```

| اسکوپ | فایل ذخیره | کجا بارگذاری می‌شود | با تیم share می‌شود؟ |
|---|---|---|---|
| `local` (پیش‌فرض) | `~/.claude.json` | فقط پروژهٔ فعلی | ❌ |
| `project` | `.mcp.json` در ریشهٔ پروژه | پروژهٔ فعلی | ✅ commit در git |
| `user` | `~/.claude.json` | همهٔ پروژه‌ها | ❌ |

اولویت: `local` > `project` > `user`.

**قانون ساده:**
- ابزار شخصی خودتان (یادداشت، ترمینال) → `user`
- ابزار مخصوص این پروژه که تیم هم لازم دارد → `project`
- تست و آزمایش → `local`

نمونهٔ `.mcp.json` برای commit:

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    },
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp"
    }
  }
}
```

پشتیبانی از متغیر محیطی: `${VAR}` و `${VAR:-default}` — کلید را مستقیم داخل `.mcp.json` commit نکنید:

```json
{
  "mcpServers": {
    "db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub", "--dsn", "${DATABASE_URL}"]
    }
  }
}
```

## مدیریت

```bash
claude mcp list              # لیست سرورها + وضعیت اتصال
claude mcp get <name>        # جزئیات یک سرور
claude mcp remove <name>     # حذف
claude mcp login <name>      # احراز هویت OAuth
claude mcp logout <name>
```

داخل جلسه:

```
/mcp        وضعیت، ابزارها، لاگین OAuth
```

## استفاده

کاری نمی‌کنید — فقط به زبان طبیعی بخواهید. Claude خودش ابزار درست را انتخاب می‌کند:

```
Review PR #456 and suggest improvements
صفحهٔ لاگین را در مرورگر باز کن و اسکرین‌شات بگیر
مستندات Next.js App Router را بگیر و بر اساسش این route را بنویس
```

## نام ابزارها (برای قواعد دسترسی)

```
mcp__<server-name>__<tool-name>
```

مثال در `settings.json`:

```json
{
  "permissions": {
    "allow": ["mcp__playwright__browser_navigate"],
    "deny":  ["mcp__github__delete_repository"]
  }
}
```

## هشدارهای امنیتی — حتماً در کلاس بگویید

1. **سرور MCP کد اجرا می‌کند.** قبل از نصب، منبع را ببینید. سرور ناشناس = دسترسی کامل به دستگاه شما.
2. **Prompt Injection.** سرور MCP می‌تواند متنی برگرداند که شبیه دستور به مدل باشد. خروجی MCP را *داده* بدانید، نه دستور.
3. **کلید API را داخل `.mcp.json` commit نکنید.** از `${VAR}` استفاده کنید.
4. **سرور غیرلازم را حذف کنید.** هر سرور ابزارهایش را به context اضافه می‌کند و توکن می‌سوزاند و انتخاب مدل را سخت‌تر می‌کند.

## ساخت سرور MCP خودتان

```bash
npx -y @modelcontextprotocol/create-server my-server
```

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool(
  "get_weather",
  { city: z.string().describe("نام شهر") },
  async ({ city }) => ({
    content: [{ type: "text", text: `هوای ${city}: ۲۵ درجه` }],
  })
);

await server.connect(new StdioServerTransport());
```

اتصال:

```bash
claude mcp add my-server -- node /path/to/server.js
```

## منابع

- MCP در Claude Code — https://code.claude.com/docs/en/mcp
- مشخصات رسمی MCP — https://modelcontextprotocol.io
- SDK ها — https://modelcontextprotocol.io/docs/sdk
- فهرست سرورهای رسمی — https://github.com/modelcontextprotocol/servers
- رجیستری MCP — https://registry.modelcontextprotocol.io
