# Claude Code & AI Tooling — Course Notes

Classroom handbook. Each chapter is standalone, short, with runnable examples and links to official sources.

## Suggested schedule (3 sessions)

| Session | Chapters | Hands-on outcome |
|---|---|---|
| 1 — Fundamentals | 01, 02, 03, 12 | Install, run, work in Plan Mode |
| 2 — Customization | 07, 08, 10, 06 | Write a `CLAUDE.md` + one Skill |
| 3 — Extending | 04, 05, 09, 11, 13 | Install an MCP + build a subagent + labs |

## Table of contents

1. [What is a harness? (Claude Code, Codex, ...)](01-harness.md)
2. [OpenAI-Compatible API and connecting other models](02-openai-compatible-api.md)
3. [Claude Code permission modes (Plan, Auto-Accept, ...)](03-modes.md)
4. [MCP — what it is, installing, scopes](04-mcp.md)
5. [Subagents — building and using them](05-subagents.md)
6. [Skills — creating and adding them](06-skills.md)
7. [`CLAUDE.md` and `AGENTS.md`](07-claude-md-agents-md.md)
8. [Prompt engineering](08-prompt-engineering.md)
9. [6 essential MCPs and agents](09-essential-tools.md)
10. [Hooks, Slash Commands, and Settings](10-hooks-and-commands.md)
11. [Professional workflow: context, cost, security, Git](11-workflow-and-best-practices.md)
12. [Cheatsheet](12-cheatsheet.md)
13. [Lab exercises](13-lab-exercises.md)

## Mental model (one glance)

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

## Primary sources

- Claude Code docs — https://code.claude.com/docs/en/overview
- MCP specification — https://modelcontextprotocol.io
- Anthropic prompt engineering — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
- AGENTS.md standard — https://agents.md
