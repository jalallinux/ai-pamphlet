> [برگشت به فهرست](README.md) · قبلی: [۱۱ — ورک‌فلو](11-workflow-and-best-practices.md) · بعدی: [۱۳ — تمرین‌ها](13-lab-exercises.md)

# فصل ۱۲ — برگهٔ تقلب

> این صفحه را پرینت بگیرید و به شاگردان بدهید.

## نصب و شروع

```bash
curl -fsSL https://claude.ai/install.sh | bash
cd your-project
claude
```

## دستورات داخل جلسه

```
/help          لیست دستورات
/init          ساخت CLAUDE.md
/context       چه چیزی در context است
/clear         پاک کردن context
/compact       خلاصه‌سازی context
/cost          هزینه
/status        لاگین، مدل
/model         تغییر مدل
/memory        ویرایش حافظه
/mcp           وضعیت MCP
/tasks         subagent های فعال
/doctor        عیب‌یابی
/resume        ادامهٔ جلسهٔ قبل
/rewind        برگشت به نقطهٔ قبل
```

## صفحه‌کلید

```
Shift+Tab    چرخش مود (default → acceptEdits → plan)
Esc          توقف
Esc Esc      ویرایش پیام قبلی
Ctrl+R       جستجوی تاریخچه
#            افزودن به حافظه
!            اجرای مستقیم شل
@            اشاره به فایل
```

## مودها

| مود | بدون پرسیدن انجام می‌دهد |
|---|---|
| `default` | تقریباً هیچ |
| `plan` | فقط خواندن، ویرایش ممنوع |
| `acceptEdits` | خواندن + ویرایش فایل |
| `auto` | classifier به‌جای شما تأیید می‌کند |
| `dontAsk` | فقط ابزارهای allowlist، بقیه رد |
| `bypassPermissions` | همه چیز ⚠️ |

```bash
claude --permission-mode plan
```

## MCP

```bash
# محلی (stdio)
claude mcp add <name> -- npx -y <package>

# راه دور (http)
claude mcp add --transport http <name> <url>

# اسکوپ
claude mcp add --scope user    <name> -- ...   # همهٔ پروژه‌ها
claude mcp add --scope project <name> -- ...   # .mcp.json، قابل commit
claude mcp add --scope local   <name> -- ...   # فقط این پروژه، خصوصی

# مدیریت
claude mcp list
claude mcp get <name>
claude mcp remove <name>
claude mcp login <name>
```

نام ابزار برای permission: `mcp__<server>__<tool>`

## Subagent

`.claude/agents/<name>.md`:

```markdown
---
name: my-agent
description: What it does and WHEN to use it. Use PROACTIVELY when ...
tools: Read, Grep, Glob
model: sonnet
---

System prompt here.
```

فراخوانی: `@agent-my-agent` یا زبان طبیعی.

## Skill

`.claude/skills/<name>/SKILL.md`:

```markdown
---
description: What it does and when to use it.
argument-hint: [arg]
allowed-tools: Bash(git diff *)
---

Context: !`git diff HEAD`

Instructions: ...  استفاده از $1 برای آرگومان
```

فراخوانی: `/<folder-name>`

## CLAUDE.md

```
~/.claude/CLAUDE.md      شخصی، همهٔ پروژه‌ها
./CLAUDE.md              پروژه، commit در git
./CLAUDE.local.md        شخصی، gitignore
.claude/rules/*.md       قوانین تفکیک‌شده (با paths:)
```

قوانین: زیر ۲۰۰ خط · تیتر و بولت · دقیق و قابل تأیید · فقط چیزی که از کد پیدا نیست.

برای `AGENTS.md`:

```markdown
@AGENTS.md
```

## Hooks

`.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\" || true" }] }
    ]
  }
}
```

رویدادها: `SessionStart` `UserPromptSubmit` `PreToolUse` `PostToolUse` `Stop` `SubagentStop` `PreCompact` `SessionEnd`

exit code: `0` ادامه · `2` مسدود + پیام به مدل

## Permissions

```json
{
  "permissions": {
    "defaultMode": "default",
    "allow": ["Bash(npm test:*)", "Read(src/**)"],
    "ask":   ["Bash(git push:*)"],
    "deny":  ["Read(./.env)", "Bash(rm -rf:*)"]
  }
}
```

اولویت: `deny` > `ask` > `allow`

## API

```bash
# Anthropic
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-5","max_tokens":1024,"messages":[{"role":"user","content":"hi"}]}'

# OpenAI-compatible
curl $BASE_URL/v1/chat/completions \
  -H "Authorization: Bearer $KEY" \
  -d '{"model":"...","messages":[{"role":"user","content":"hi"}]}'
```

Gateway برای Claude Code:

```bash
export ANTHROPIC_BASE_URL="https://gateway.example.com"
export ANTHROPIC_AUTH_TOKEN="token"
```

## نصب سریع ۵ MCP

```bash
claude mcp add --scope user context7            -- npx -y @upstash/context7-mcp
claude mcp add --scope user playwright          -- npx -y @playwright/mcp@latest
claude mcp add --scope user sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
claude mcp add --scope user filesystem          -- npx -y @modelcontextprotocol/server-filesystem ~/Documents
claude mcp add --scope user --transport http github https://api.githubcopilot.com/mcp/
```

## کلمات کلیدی فکر کردن

```
think  <  think hard  <  think harder  <  ultrathink
```

## الگوی طلایی

```
plan mode → نقشه بگیر → اصلاح کن → تأیید →
تست بنویس → پیاده‌سازی → تست اجرا → خودت review کن → commit
```
