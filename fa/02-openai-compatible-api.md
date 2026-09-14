> [برگشت به فهرست](README.md) · قبلی: [۰۱ — هارنس](01-harness.md) · بعدی: [۰۳ — مودها](03-modes.md)

# فصل ۰۲ — OpenAI-Compatible API

## مسئله

هر شرکت AI فرمت API خودش را دارد. اگر هر ابزار بخواهد برای همه جدا کد بنویسد، فاجعه است.

## راه‌حل

صنعت روی فرمت OpenAI به‌عنوان *استاندارد عملی* توافق کرد. هر سرویسی که این سه چیز را رعایت کند، «OpenAI-compatible» است:

1. مسیر `POST /v1/chat/completions`
2. هدر `Authorization: Bearer <KEY>`
3. بدنهٔ JSON با `model` و `messages`

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "سلام"}]
  }'
```

نتیجه: همین کد با تعویض `base_url` روی OpenRouter, Groq, Together, Ollama, vLLM, DeepSeek و ده‌ها سرویس دیگر کار می‌کند.

## استفاده با SDK رسمی OpenAI

فقط `base_url` را عوض کنید:

```python
from openai import OpenAI

# محلی با Ollama
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

# OpenRouter (دسترسی به صدها مدل با یک کلید)
client = OpenAI(base_url="https://openrouter.ai/api/v1", api_key="sk-or-...")

# Groq (سرعت بسیار بالا)
client = OpenAI(base_url="https://api.groq.com/openai/v1", api_key="gsk_...")

resp = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[{"role": "user", "content": "سلام"}],
)
print(resp.choices[0].message.content)
```

## Anthropic فرمت خودش را دارد

Claude فرمت `/v1/messages` دارد، نه `/v1/chat/completions`. تفاوت‌های مهم:

| موضوع | OpenAI | Anthropic |
|---|---|---|
| مسیر | `/v1/chat/completions` | `/v1/messages` |
| هدر احراز هویت | `Authorization: Bearer` | `x-api-key` |
| نسخه | — | `anthropic-version: 2023-06-01` |
| System prompt | پیام با `role: "system"` | پارامتر جدای `system` |
| `max_tokens` | اختیاری | **اجباری** |

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-5",
    "max_tokens": 1024,
    "system": "تو یک معلم برنامه‌نویسی هستی.",
    "messages": [{"role": "user", "content": "سلام"}]
  }'
```

اما Anthropic یک لایهٔ سازگاری OpenAI هم دارد (برای مهاجرت، نه برای تولید):

```python
client = OpenAI(
    base_url="https://api.anthropic.com/v1/",
    api_key=os.environ["ANTHROPIC_API_KEY"],
)
```

## تغییر مسیر خود Claude Code به یک Gateway

Claude Code از طریق دو متغیر محیطی به هر gateway سازگار با فرمت Anthropic وصل می‌شود:

```bash
export ANTHROPIC_BASE_URL="https://your-gateway.company.com"
export ANTHROPIC_AUTH_TOKEN="your-gateway-token"   # هدر Authorization: Bearer
claude
```

| متغیر | هدر ارسالی | کاربرد |
|---|---|---|
| `ANTHROPIC_BASE_URL` | — | آدرس gateway |
| `ANTHROPIC_AUTH_TOKEN` | `Authorization: Bearer` | توکن gateway/پروکسی |
| `ANTHROPIC_API_KEY` | `X-Api-Key` | کلید مستقیم Anthropic |

ترتیب اولویت احراز هویت: cloud provider → `ANTHROPIC_AUTH_TOKEN` → `ANTHROPIC_API_KEY` → `apiKeyHelper` → `CLAUDE_CODE_OAUTH_TOKEN` → لاگین `/login`.

با `/status` چک کنید کدام فعال است.

> **هشدار امنیتی:** `ANTHROPIC_BASE_URL` تمام ترافیک — شامل محتوای کد شما — را به آن سرور می‌فرستد. فقط به gateway سازمان خودتان یا سرویسی که واقعاً به آن اعتماد دارید وصل شوید. یک proxy ناشناخته یعنی کل کدبیس شما نزد شخص ثالث.

> **نکته:** Anthropic رسمی پشتیبانی نمی‌کند که Claude Code را به مدل غیر-Claude وصل کنید. ابزارهای جامعه (مثل `claude-code-router`) این کار را می‌کنند اما ناپایدارند.

## LiteLLM — یک پروکسی برای همه

اگر می‌خواهید یک endpoint واحد جلوی همهٔ providerها بگذارید:

```yaml
# config.yaml
model_list:
  - model_name: claude-sonnet
    litellm_params:
      model: anthropic/claude-sonnet-5
      api_key: os.environ/ANTHROPIC_API_KEY
  - model_name: local-llama
    litellm_params:
      model: ollama/llama3.1:8b
      api_base: http://localhost:11434
```

```bash
litellm --config config.yaml --port 4000
```

حالا `http://localhost:4000/v1` هم فرمت OpenAI و هم `/v1/messages` را سرو می‌کند.

## کجا مفید است

- **هزینه** — مدل ارزان برای کارهای ساده، مدل گران برای کارهای سخت
- **حریم خصوصی** — Ollama یا vLLM روی سرور خودتان
- **پشتیبان (fallback)** — قطع شدن یک provider کار را متوقف نکند
- **تست** — همان کد را روی چند مدل مقایسه کنید

## منابع

- مرجع OpenAI Chat Completions — https://platform.openai.com/docs/api-reference/chat
- مرجع Anthropic Messages API — https://docs.claude.com/en/api/messages
- لایهٔ سازگاری OpenAI در Anthropic — https://docs.claude.com/en/api/openai-sdk
- اتصال Claude Code به gateway — https://code.claude.com/docs/en/llm-gateway-connect
- متغیرهای محیطی Claude Code — https://code.claude.com/docs/en/env-vars
- LiteLLM — https://docs.litellm.ai
- OpenRouter — https://openrouter.ai/docs
- Ollama OpenAI compatibility — https://ollama.com/blog/openai-compatibility
