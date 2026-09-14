> [Back to index](README.md) · Next: [02 — OpenAI-Compatible API](02-openai-compatible-api.md)

# Chapter 01 — What is a Harness?

## One-line definition

A **model** only produces text. A **harness** is the program that gives it hands and feet: read files, run a terminal, browse the web, and loop until the job is done.

```
You → harness → [model] → tool decision → harness runs the tool → result back to model → repeat
```

Without a harness: ChatGPT writes code in a chat and you copy-paste it.
With a harness: Claude Code creates files, runs tests, sees the error, and fixes it.

## Parts of every harness

| Part | Job |
|---|---|
| **Agent loop** | think → tool → observe → think... until the goal is met |
| **Tools** | Read, Write, Edit, Bash, Grep, Glob, WebFetch |
| **Context management** | what goes in the model's window, when to summarize (compaction) |
| **Permissions** | what runs without asking you |
| **Extensibility** | MCP, subagents, skills, hooks |

## The main harnesses compared

| Harness | Maker | Default model | Instruction file | Note |
|---|---|---|---|---|
| **Claude Code** | Anthropic | Claude (Opus/Sonnet/Haiku) | `CLAUDE.md` | Richest extension ecosystem (MCP/Skill/Agent/Hook) |
| **Codex CLI** | OpenAI | GPT | `AGENTS.md` | Open source, Rust, strong sandbox |
| **Gemini CLI** | Google | Gemini | `GEMINI.md` | Open source, very large context window |
| **Cursor** | Anysphere | Multiple | `.cursorrules` | Full IDE, not just a CLI |
| **Aider** | Open source | Any model | `CONVENTIONS.md` | Git-native, lightweight |
| **OpenCode** | Open source | Any model | `AGENTS.md` | TUI, provider-agnostic |

## The key point for class

> The harness and the model are two separate things.
> Output quality = model quality × harness quality × **your context quality**.

Most frustration with AI traces back to the third factor: bad context. Chapters 07 and 08 are exactly that factor.

## Quick install

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Then inside your project:

```bash
claude
```

First run opens a browser to log in (Pro/Max subscription or a Console API key).

Basic in-session commands:

```
/help      list commands
/status    login, model, cost
/context   what is loaded into context
/doctor    diagnose config problems
/clear     wipe context
```

## Sources

- Install Claude Code — https://code.claude.com/docs/en/setup
- Claude Code overview — https://code.claude.com/docs/en/overview
- OpenAI Codex CLI — https://github.com/openai/codex
- Gemini CLI — https://github.com/google-gemini/gemini-cli
- Aider — https://aider.chat
- OpenCode — https://opencode.ai
