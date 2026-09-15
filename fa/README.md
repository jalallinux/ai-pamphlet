# جزوهٔ آموزش Claude Code و ابزارهای جانبی AI

جزوهٔ کلاسی. هر فصل مستقل، کوتاه، با مثال اجرایی و لینک منبع رسمی.

## مسیر پیشنهادی کلاس (۳ جلسه)

| جلسه | فصل‌ها | خروجی عملی |
|---|---|---|
| جلسه ۱ — مبانی | ۰۱، ۰۲، ۰۳، ۱۲ | نصب، اجرا، کار با Plan Mode |
| جلسه ۲ — شخصی‌سازی | ۰۷، ۰۸، ۱۰، ۰۶ | نوشتن `CLAUDE.md` + یک Skill |
| جلسه ۳ — توسعهٔ قابلیت | ۰۴، ۰۵، ۰۹، ۱۱، ۱۳ | نصب MCP + ساخت Subagent + تمرین‌ها |

## فهرست فصل‌ها

1. [هارنس چیست؟ (Claude Code، Codex، ...)](01-harness.md)
2. [OpenAI-Compatible API و اتصال مدل‌های دیگر](02-openai-compatible-api.md)
3. [مودهای Claude Code (Plan, Auto-Accept, ...)](03-modes.md)
4. [MCP — چیست، نصب، اسکوپ‌ها](04-mcp.md)
5. [Subagents — ساخت و استفاده](05-subagents.md)
6. [Skills — ساخت و افزودن](06-skills.md)
7. [`CLAUDE.md` و `AGENTS.md`](07-claude-md-agents-md.md)
8. [مهندسی پرامپت](08-prompt-engineering.md)
9. [۶ MCP و Agent ضروری](09-essential-tools.md)
10. [Hooks و Slash Commands و Settings](10-hooks-and-commands.md)
11. [ورک‌فلو حرفه‌ای: context، هزینه، امنیت، Git](11-workflow-and-best-practices.md)
12. [برگهٔ تقلب (Cheatsheet)](12-cheatsheet.md)
13. [تمرین‌های کلاس](13-lab-exercises.md)

## نقشهٔ ذهنی (یک نگاه)

```
هارنس (Claude Code / Codex)
├── مدل        → از کجا می‌آید؟ (Anthropic | OpenAI-compatible proxy)   → فصل ۰۲
├── حافظه      → CLAUDE.md / AGENTS.md                                  → فصل ۰۷
├── کنترل      → مودها، Permissions، Hooks                              → فصل ۰۳، ۱۰
├── قابلیت‌ها
│   ├── MCP       → ابزار خارجی (مرورگر، دیتابیس، داکیومنت)             → فصل ۰۴
│   ├── Subagent  → پنجرهٔ context جدا برای کار تخصصی                   → فصل ۰۵
│   └── Skill     → دستورالعمل قابل بارگذاری بر حسب نیاز                → فصل ۰۶
└── کیفیت خروجی → مهندسی پرامپت                                        → فصل ۰۸
```

## منابع اصلی

- مستندات رسمی Claude Code — https://code.claude.com/docs/en/overview
- مشخصات MCP — https://modelcontextprotocol.io
- مهندسی پرامپت Anthropic — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
- استاندارد AGENTS.md — https://agents.md
