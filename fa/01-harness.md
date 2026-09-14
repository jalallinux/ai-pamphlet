> [برگشت به فهرست](README.md) · بعدی: [۰۲ — OpenAI-Compatible API](02-openai-compatible-api.md)

# فصل ۰۱ — هارنس (Harness) چیست؟

## تعریف در یک خط

**مدل** فقط متن تولید می‌کند. **هارنس** برنامه‌ای است که به مدل دست و پا می‌دهد: فایل بخواند، ترمینال اجرا کند، وب بگردد، و حلقه بزند تا کار تمام شود.

```
شما → هارنس → [مدل] → تصمیم ابزار → هارنس ابزار را اجرا می‌کند → نتیجه به مدل → تکرار
```

بدون هارنس: ChatGPT که کد را در چت می‌نویسد و شما کپی می‌کنید.
با هارنس: Claude Code که خودش فایل می‌سازد، تست می‌زند، خطا را می‌بیند و درست می‌کند.

## اجزای هر هارنس

| جزء | کار |
|---|---|
| **Agent Loop** | حلقهٔ فکر → ابزار → مشاهده → فکر... تا رسیدن به هدف |
| **Tools** | Read, Write, Edit, Bash, Grep, Glob, WebFetch |
| **Context Management** | چه چیزی داخل پنجرهٔ مدل برود، چه زمانی خلاصه شود (compaction) |
| **Permissions** | چه کاری بدون اجازه اجرا شود |
| **Extensibility** | MCP, Subagent, Skill, Hook |

## مقایسهٔ هارنس‌های اصلی

| هارنس | سازنده | مدل پیش‌فرض | فایل دستورالعمل | نکته |
|---|---|---|---|---|
| **Claude Code** | Anthropic | Claude (Opus/Sonnet/Haiku) | `CLAUDE.md` | کامل‌ترین اکوسیستم افزونه (MCP/Skill/Agent/Hook) |
| **Codex CLI** | OpenAI | GPT | `AGENTS.md` | متن‌باز، Rust، سندباکس قوی |
| **Gemini CLI** | Google | Gemini | `GEMINI.md` | متن‌باز، context window خیلی بزرگ |
| **Cursor** | Anysphere | چند مدل | `.cursorrules` | IDE کامل، نه فقط CLI |
| **Aider** | متن‌باز | هر مدلی | `CONVENTIONS.md` | Git-native، سبک |
| **OpenCode** | متن‌باز | هر مدلی | `AGENTS.md` | TUI، provider-agnostic |

## نکتهٔ کلیدی برای کلاس

> هارنس و مدل دو چیز جدا هستند.
> کیفیت خروجی = کیفیت مدل × کیفیت هارنس × کیفیت context شما.

بیشتر نارضایتی از AI مربوط به ضلع سوم است: context بد. فصل‌های ۰۷ و ۰۸ دقیقاً همان ضلع.

## نصب سریع Claude Code

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

بعد در پوشهٔ پروژه:

```bash
claude
```

بار اول مرورگر برای لاگین باز می‌شود (اشتراک Pro/Max یا Console API key).

دستورات پایه داخل جلسه:

```
/help      لیست دستورات
/status    وضعیت لاگین، مدل، هزینه
/context   چه چیزی داخل context بارگذاری شده
/doctor    تشخیص مشکلات کانفیگ
/clear     پاک کردن context
```

## منابع

- نصب Claude Code — https://code.claude.com/docs/en/setup
- نمای کلی Claude Code — https://code.claude.com/docs/en/overview
- OpenAI Codex CLI — https://github.com/openai/codex
- Gemini CLI — https://github.com/google-gemini/gemini-cli
- Aider — https://aider.chat
- OpenCode — https://opencode.ai
