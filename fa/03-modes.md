> [برگشت به فهرست](README.md) · قبلی: [۰۲ — OpenAI-Compatible API](02-openai-compatible-api.md) · بعدی: [۰۴ — MCP](04-mcp.md)

# فصل ۰۳ — مودهای Claude Code (Permission Modes)

## مفهوم

مود تعیین می‌کند Claude **قبل از چه کاری از شما اجازه بگیرد**. این تنها دکمهٔ تعادل بین سرعت و کنترل است.

## جدول مودها

| مود (مقدار config) | نام در UI | بدون پرسیدن انجام می‌دهد | مناسب برای |
|---|---|---|---|
| `default` | Manual | تقریباً هیچ؛ برای هر ویرایش/دستور می‌پرسد | کد حساس، کد ناآشنا |
| `plan` | Plan | فقط خواندن؛ **ویرایش ممنوع** تا تأیید نقشه | بررسی قبل از تغییر |
| `acceptEdits` | Accept Edits | خواندن، ویرایش فایل، دستورات فایل‌سیستمی معمول (`mkdir`, `mv`, ...) | تکرار سریع روی کدی که خودتان مرور می‌کنید |
| `auto` | Auto | یک مدل دوم (classifier) به‌جای شما بررسی می‌کند | کار hands-off (پیش‌فرض Pro/Max/Team) |
| `dontAsk` | Don't Ask | فقط ابزارهای از پیش تأییدشده؛ بقیه **رد** می‌شود | CI و اسکریپت‌های قفل‌شده |
| `bypassPermissions` | Bypass | همه چیز، بدون هیچ سؤالی | فقط داخل کانتینر/VM جداشده |

## Plan Mode — مهم‌ترین عادت خوب

چرخهٔ کار:

```
۱. وارد Plan Mode شوید
۲. بخواهید: «کدبیس را بررسی کن و نقشهٔ اضافه کردن X را بده»
۳. Claude فقط می‌خواند و می‌گردد — هیچ فایلی تغییر نمی‌کند
۴. نقشه را می‌بینید، اصلاح می‌کنید
۵. تأیید → Claude شروع به پیاده‌سازی می‌کند
```

چرا مهم است: ارزان‌ترین جا برای اصلاح اشتباه، *قبل* از نوشتن کد است. خواندن یک نقشهٔ ۱۰ خطی از review کردن ۴۰۰ خط diff اشتباه خیلی سریع‌تر است.

## عوض کردن مود

**داخل جلسه:** کلید `Shift+Tab` را بزنید تا بین مودها بچرخد.

چرخه: `default` → `acceptEdits` → `plan` → برگشت به `default`
(از `auto` اولین زدن به `default` می‌رود.)

نوار وضعیت مود فعال را نشان می‌دهد:
```
⏸ manual mode on
⏵⏵ accept edits on
⏸ plan mode on
⏵⏵ auto mode on
```

**هنگام اجرا:**

```bash
claude --permission-mode plan
claude --permission-mode acceptEdits
claude -p "run the test suite" --permission-mode dontAsk --allowedTools "Bash(npm test)" "Read"
```

**به‌صورت دائمی** در `.claude/settings.json`:

```json
{
  "permissions": {
    "defaultMode": "default"
  }
}
```

> نکته: مقادیر `auto` و `bypassPermissions` از فایل settings پروژه اعمال **نمی‌شوند** (دلیل امنیتی). فقط از settings کاربر یا managed.

## قواعد دسترسی (Permission Rules)

مستقل از مود، می‌توانید قواعد دقیق تعریف کنید:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:*)",
      "Bash(git status)",
      "Read(src/**)"
    ],
    "ask": [
      "Bash(git push:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./secrets/**)",
      "Bash(rm -rf *)"
    ]
  }
}
```

اولویت: `deny` > `ask` > `allow`.

`deny` همیشه برنده است — حتی در `bypassPermissions`. برای فایل‌های حساس (`.env`، کلیدها) همیشه `deny` بنویسید.

## توصیهٔ عملی برای کلاس

| موقعیت | مود |
|---|---|
| اولین بار روی پروژه | `plan` |
| refactor بزرگ | `plan` → تأیید → `acceptEdits` |
| نوشتن تست و اجرای مکرر | `acceptEdits` |
| کد production حساس | `default` |
| CI | `dontAsk` + allowlist دقیق |
| `bypassPermissions` روی لپ‌تاپ شخصی | ❌ نه |

## منابع

- مودهای دسترسی — https://code.claude.com/docs/en/permission-modes
- تنظیم دسترسی‌ها — https://code.claude.com/docs/en/permissions
- مرجع CLI — https://code.claude.com/docs/en/cli-reference
- Sandboxing — https://code.claude.com/docs/en/sandboxing
