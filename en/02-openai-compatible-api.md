> [Back to index](README.md) · Previous: [01 — Harness](01-harness.md) · Next: [03 — Modes](03-modes.md)

# Chapter 02 — OpenAI-Compatible API

## The problem

Every AI company has its own API format. If every tool had to write separate code for each one, that's a combinatorial mess.

## The solution

The industry converged on OpenAI's format as the *de facto* standard. Any service that honors these three things is "OpenAI-compatible":

1. `POST /v1/chat/completions`
2. `Authorization: Bearer <KEY>` header
3. JSON body with `model` and `messages`

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "hello"}]
  }'
```

Result: the same code works against OpenRouter, Groq, Together, Ollama, vLLM, DeepSeek, and dozens of others by swapping `base_url`.

## Using the official OpenAI SDK

Change only `base_url`:

```python
from openai import OpenAI

# Local via Ollama
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

# OpenRouter (hundreds of models behind one key)
client = OpenAI(base_url="https://openrouter.ai/api/v1", api_key="sk-or-...")

# Groq (very fast inference)
client = OpenAI(base_url="https://api.groq.com/openai/v1", api_key="gsk_...")

resp = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[{"role": "user", "content": "hello"}],
)
print(resp.choices[0].message.content)
```

## Anthropic has its own format

Claude uses `/v1/messages`, not `/v1/chat/completions`. The differences that matter:

| Aspect | OpenAI | Anthropic |
|---|---|---|
| Path | `/v1/chat/completions` | `/v1/messages` |
| Auth header | `Authorization: Bearer` | `x-api-key` |
| Version header | — | `anthropic-version: 2023-06-01` |
| System prompt | message with `role: "system"` | separate `system` parameter |
| `max_tokens` | optional | **required** |

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-5",
    "max_tokens": 1024,
    "system": "You are a programming teacher.",
    "messages": [{"role": "user", "content": "hello"}]
  }'
```

Anthropic does ship an OpenAI compatibility layer, intended for migration rather than production:

```python
client = OpenAI(
    base_url="https://api.anthropic.com/v1/",
    api_key=os.environ["ANTHROPIC_API_KEY"],
)
```

## Pointing Claude Code itself at a gateway

Claude Code connects to any Anthropic-format gateway through two environment variables:

```bash
export ANTHROPIC_BASE_URL="https://your-gateway.company.com"
export ANTHROPIC_AUTH_TOKEN="your-gateway-token"   # sent as Authorization: Bearer
claude
```

| Variable | Header sent | Use for |
|---|---|---|
| `ANTHROPIC_BASE_URL` | — | gateway address |
| `ANTHROPIC_AUTH_TOKEN` | `Authorization: Bearer` | gateway/proxy token |
| `ANTHROPIC_API_KEY` | `X-Api-Key` | direct Anthropic key |

Auth precedence: cloud provider → `ANTHROPIC_AUTH_TOKEN` → `ANTHROPIC_API_KEY` → `apiKeyHelper` → `CLAUDE_CODE_OAUTH_TOKEN` → `/login` credential.

Check which one is active with `/status`.

> **Security warning:** `ANTHROPIC_BASE_URL` routes all traffic — including your source code — to that server. Only point it at your own organization's gateway or a service you genuinely trust. An unknown proxy means your entire codebase is in a third party's hands.

> **Note:** Anthropic does not officially support routing Claude Code to non-Claude models. Community tools (such as `claude-code-router`) do this, but they're brittle.

## LiteLLM — one proxy for everything

If you want a single endpoint in front of all providers:

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

Now `http://localhost:4000/v1` serves both the OpenAI format and `/v1/messages`.

## Where this is useful

- **Cost** — cheap model for easy work, expensive model for hard work
- **Privacy** — Ollama or vLLM on your own hardware
- **Fallback** — one provider going down doesn't stop your work
- **Evaluation** — run the same code against several models and compare

## Sources

- OpenAI Chat Completions reference — https://platform.openai.com/docs/api-reference/chat
- Anthropic Messages API reference — https://docs.claude.com/en/api/messages
- Anthropic OpenAI compatibility layer — https://docs.claude.com/en/api/openai-sdk
- Connect Claude Code to a gateway — https://code.claude.com/docs/en/llm-gateway-connect
- Claude Code environment variables — https://code.claude.com/docs/en/env-vars
- LiteLLM — https://docs.litellm.ai
- OpenRouter — https://openrouter.ai/docs
- Ollama OpenAI compatibility — https://ollama.com/blog/openai-compatibility
