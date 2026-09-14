> [برگشت به فهرست](README.md) · قبلی: [۰۹ — ابزارهای ضروری](09-essential-tools.md) · بعدی: [۱۱ — ورک‌فلو](11-workflow-and-best-practices.md)

# فصل ۱۰ — Hooks، Settings و Plugins

## بخش الف — Hooks

### تعریف

Hook یک **دستور شل** است که در نقطهٔ مشخصی از چرخهٔ عمر Claude Code اجرا می‌شود — **مستقل از تصمیم مدل**.

تفاوت با `CLAUDE.md`:

| | `CLAUDE.md` | Hook |
|---|---|---|
| ماهیت | درخواست | اجبار |
| تضمین | ندارد | دارد |
| مناسب برای | سبک و قرارداد | lint، فرمت، مسدودسازی |

اگر جمله‌ای در `CLAUDE.md` نوشتید که شروع می‌شود با «همیشه...» یا «هرگز...» و مدل گاهی نادیده می‌گیرد — آن را Hook کنید.

### رویدادهای پرکاربرد

| رویداد | چه زمانی |
|---|---|
| `SessionStart` | شروع جلسه |
| `UserPromptSubmit` | بعد از ارسال پیام شما، قبل از رسیدن به مدل |
| `PreToolUse` | قبل از اجرای ابزار — **می‌تواند جلوی آن را بگیرد** |
| `PostToolUse` | بعد از اجرای موفق ابزار |
| `Stop` | وقتی Claude پاسخ را تمام می‌کند |
| `SubagentStop` | پایان کار subagent |
| `PreCompact` | قبل از خلاصه‌سازی context |
| `SessionEnd` | پایان جلسه |

(در مجموع ۳۳+ رویداد وجود دارد.)

### ساختار

سه سطح تودرتو: رویداد → گروه matcher → آرایهٔ handler.

`.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/lint-check.sh"
          }
        ]
      }
    ]
  }
}
```

`matcher` ننویسید = برای همه اجرا می‌شود.

### سه Hook کاربردی

**۱. فرمت خودکار بعد از هر ویرایش**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**۲. مسدود کردن ویرایش فایل‌های حساس**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/block-sensitive.sh"
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/block-sensitive.sh`:

```bash
#!/usr/bin/env bash
# ورودی hook به‌صورت JSON روی stdin می‌آید
input=$(cat)
path=$(echo "$input" | jq -r '.tool_input.file_path // empty')

case "$path" in
  *.env|*/secrets/*|*/.git/*)
    echo "Refusing to modify protected file: $path" >&2
    exit 2   # exit code 2 = مسدود کن و به مدل بگو چرا
    ;;
esac
exit 0
```

قرارداد exit code:
- `0` → ادامه بده
- `2` → مسدود کن، `stderr` را به مدل نشان بده
- بقیه → خطای غیرمسدودکننده

**۳. اعلان صوتی وقتی کار تمام شد**

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'کار تمام شد'"
          }
        ]
      }
    ]
  }
}
```

> ⚠️ **هشدار امنیتی:** Hook ها با دسترسی کامل کاربر شما و بدون تأیید اجرا می‌شوند. هرگز Hook ای را از منبع نامطمئن copy-paste نکنید. قبل از commit کردن `.claude/settings.json` در repo مشترک، محتوای هر hook را بخوانید.

---

## بخش ب — Slash Commands

از نسخه‌های اخیر، **Skill جایگزین Slash Command** شده. فرمت قدیمی همچنان کار می‌کند:

```
.claude/commands/deploy.md        →  /deploy
.claude/commands/frontend/test.md →  /frontend:test
```

برای کار جدید از [Skill](06-skills.md) استفاده کنید.

### دستورات داخلی مهم

| دستور | کار |
|---|---|
| `/help` | لیست کامل |
| `/clear` | پاک کردن context — بین کارهای نامرتبط حتماً بزنید |
| `/compact` | خلاصه‌سازی context با حفظ نکات مهم |
| `/context` | چه چیزی و چقدر در context است |
| `/cost` | هزینهٔ جلسه |
| `/status` | لاگین، مدل، تنظیمات فعال |
| `/memory` | ویرایش فایل‌های حافظه |
| `/mcp` | وضعیت سرورهای MCP |
| `/tasks` | subagent های در حال اجرا |
| `/model` | تغییر مدل |
| `/init` | ساخت `CLAUDE.md` اولیه |
| `/doctor` | تشخیص مشکلات کانفیگ |
| `/review` | مرور کد |
| `/resume` | ادامهٔ جلسهٔ قبلی |
| `/rewind` | برگشت به نقطهٔ قبلی گفتگو |

### میان‌برهای صفحه‌کلید

| کلید | کار |
|---|---|
| `Shift+Tab` | چرخش بین مودها |
| `Esc` | توقف Claude |
| `Esc Esc` | ویرایش پیام قبلی |
| `Ctrl+R` | جستجو در تاریخچه |
| `#` در ابتدای خط | افزودن سریع به حافظه |
| `!` در ابتدای خط | اجرای مستقیم دستور شل |
| `@` | اشاره به فایل |

---

## بخش ج — Settings

### سلسله‌مراتب فایل‌ها

| فایل | اسکوپ |
|---|---|
| `~/.claude/settings.json` | همهٔ پروژه‌های شما |
| `.claude/settings.json` | پروژه — commit در git |
| `.claude/settings.local.json` | پروژه، شخصی — در `.gitignore` |
| managed settings | سیاست سازمان — غیرقابل override |

اولویت: managed > local > project > user.

### نمونهٔ کامل

```json
{
  "permissions": {
    "defaultMode": "default",
    "allow": [
      "Bash(npm run test:*)",
      "Bash(npm run lint:*)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ],
    "ask": [
      "Bash(git push:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Bash(rm -rf:*)",
      "Bash(curl:*)"
    ]
  },
  "env": {
    "NODE_ENV": "development"
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\" || true" }
        ]
      }
    ]
  }
}
```

---

## بخش د — Plugins

پلاگین، بستهٔ آماده‌ای از Skill + Agent + MCP + Hook است.

```
/plugin marketplace add anthropics/claude-code
/plugin install <plugin-name>
/plugin list
```

مارکت‌پلیس‌های جامعه:
- https://github.com/anthropics/claude-code
- https://github.com/davila7/claude-code-templates

---

## منابع

- Hooks — https://code.claude.com/docs/en/hooks
- راهنمای Hooks — https://code.claude.com/docs/en/hooks-guide
- Settings — https://code.claude.com/docs/en/settings
- مرجع Settings — https://code.claude.com/docs/en/settings-reference
- دستورات — https://code.claude.com/docs/en/commands
- Plugins — https://code.claude.com/docs/en/plugins
