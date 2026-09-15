# Claude Code & AI Tooling — Course Notes

Teaching material for a hands-on course on **Claude Code**, agentic harnesses, MCP, subagents, skills, and prompt engineering. Written as classroom notes: short chapters, runnable examples, links to official sources.

Available in two languages, with identical structure and content.

| Language | Index | فهرست |
|---|---|---|
| 🇬🇧 **English** | **[en/README.md](en/README.md)** | — |
| 🇮🇷 **فارسی (Persian)** | — | **[fa/README.md](fa/README.md)** |

---

## What's covered

| # | Chapter | English | فارسی |
|---|---|---|---|
| 01 | What is a harness? (Claude Code, Codex, Gemini CLI, Cursor, Aider) | [en](en/01-harness.md) | [fa](fa/01-harness.md) |
| 02 | OpenAI-compatible API, gateways, `ANTHROPIC_BASE_URL`, LiteLLM | [en](en/02-openai-compatible-api.md) | [fa](fa/02-openai-compatible-api.md) |
| 03 | Permission modes — plan, acceptEdits, auto, dontAsk, bypass | [en](en/03-modes.md) | [fa](fa/03-modes.md) |
| 04 | MCP — transports, scopes, building a server, security | [en](en/04-mcp.md) | [fa](fa/04-mcp.md) |
| 05 | Subagents — frontmatter, context isolation, 3 ready agents | [en](en/05-subagents.md) | [fa](fa/05-subagents.md) |
| 06 | Skills — progressive disclosure, shell injection, 3 ready skills | [en](en/06-skills.md) | [fa](fa/06-skills.md) |
| 07 | `CLAUDE.md` and `AGENTS.md` — hierarchy, imports, `.claude/rules/` | [en](en/07-claude-md-agents-md.md) | [fa](fa/07-claude-md-agents-md.md) |
| 08 | Prompt engineering — 10 parts, ordered by impact | [en](en/08-prompt-engineering.md) | [fa](fa/08-prompt-engineering.md) |
| 09 | 6 essential MCPs + spec-driven development | [en](en/09-essential-tools.md) | [fa](fa/09-essential-tools.md) |
| 10 | Hooks, settings, slash commands, plugins | [en](en/10-hooks-and-commands.md) | [fa](fa/10-hooks-and-commands.md) |
| 11 | Workflow — context, cost, Git, security, 4 patterns | [en](en/11-workflow-and-best-practices.md) | [fa](fa/11-workflow-and-best-practices.md) |
| 12 | Cheatsheet (printable, one page) | [en](en/12-cheatsheet.md) | [fa](fa/12-cheatsheet.md) |
| 13 | 10 lab exercises + assessment rubric | [en](en/13-lab-exercises.md) | [fa](fa/13-lab-exercises.md) |
| 14 | Sample project — URL shortener, 10 steps, zero dependencies | [en](en/14-sample-project.md) | [fa](fa/14-sample-project.md) |

---

## Suggested schedule

Four sessions, roughly two hours each.

| Session | Chapters | Hands-on outcome |
|---|---|---|
| **1 — Fundamentals** | 01, 02, 03, 12 | Install, run a first session, work in Plan Mode |
| **2 — Customization** | 07, 08, 10, 06 | Write a `CLAUDE.md` and one Skill |
| **3 — Extending** | 04, 05, 09, 11, 13 | Install an MCP, build a subagent, run the labs |
| **4 — Capstone** | 14 | Build the URL shortener end to end |

The labs in chapter 13 are ordered to build on each other. Labs 1–4 fit session 1–2; labs 5–10 fit session 3 and homework.

Chapter 14 is one continuous project that uses every chapter once. It fits a fourth session, or a take-home assignment after session 3.

---

## Mental model

```
Harness (Claude Code / Codex)
├── Model       → where does it come from? (Anthropic | OpenAI-compatible proxy)  → ch. 02
├── Memory      → CLAUDE.md / AGENTS.md                                           → ch. 07
├── Control     → modes, permissions, hooks                                       → ch. 03, 10
├── Capabilities
│   ├── MCP       → external tools (browser, database, docs)                       → ch. 04
│   ├── Subagent  → separate context window for specialized work                   → ch. 05
│   └── Skill     → instructions loaded on demand                                  → ch. 06
└── Output quality → prompt engineering                                            → ch. 08
```

---

## Repository layout

```
.
├── README.md      ← you are here
├── en/            English notes (15 files)
│   ├── README.md  English index
│   └── 01..14     chapters
└── fa/            Persian notes (15 files)
    ├── README.md  فهرست فارسی
    └── 01..14     فصل‌ها
```

Every chapter links to the previous and next one, and back to its language index.

---

## Notes for the instructor

- **Chapters 04, 05, 06, 07, and 10** were written against the live docs at `code.claude.com`, so the CLI syntax and frontmatter fields are exact. Everything else is conceptual and ages more slowly.
- **Labs 5 and 7** require `jq` installed and a session restart. Run them yourself once before class.
- **Lab 2** clones a public repo; make sure the room has network access, or pre-clone it.
- **Chapter 14** needs Node 22+ for `node:sqlite` (`--experimental-sqlite` on 22, stable on 24). Check `node --version` in the room before class; on Node 20 or older, substitute `express` + `better-sqlite3` — the steps are unchanged. The `node:sqlite` calls in that chapter were run and verified on Node 22.23.
- Anthropic moved its Claude Code docs from `docs.claude.com/en/docs/claude-code/*` to `code.claude.com/docs/en/*`. The new URLs are used throughout.
- Chapter 12 is designed to be printed and handed out.

---

## Primary sources

- Claude Code documentation — https://code.claude.com/docs/en/overview
- Claude Code best practices — https://www.anthropic.com/engineering/claude-code-best-practices
- Model Context Protocol — https://modelcontextprotocol.io
- Anthropic prompt engineering — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
- AGENTS.md standard — https://agents.md

---

## License

Course material. Reuse and adapt freely for teaching.
