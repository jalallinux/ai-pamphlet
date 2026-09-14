> [Back to index](README.md) · Previous: [03 — Modes](03-modes.md) · Next: [05 — Subagents](05-subagents.md)

# Chapter 04 — MCP (Model Context Protocol)

## Definition

MCP is an **open protocol** that connects external tools to models. The official metaphor: "USB-C for AI".

Without MCP: every (harness × tool) pair needs its own code → M×N.
With MCP: each tool writes one server, each harness writes one client → M+N.

Introduced by Anthropic in November 2024, now an industry standard (OpenAI, Google, and Microsoft all support it).

## An MCP server provides three things

| Kind | What it is | Example |
|---|---|---|
| **Tools** | Functions the model calls | `query_database`, `browser_click` |
| **Resources** | Data the model reads | files, tables, docs |
| **Prompts** | Ready-made templates you invoke | `/review-pr` |

## Transport types

| Type | Where it runs | Best for |
|---|---|---|
| `stdio` | Local process on your machine | Local tools, filesystem access, local databases |
| `http` | Remote server (recommended) | SaaS, cloud services, OAuth support |
| `sse` | Remote (deprecated) | Legacy — use http instead |

## Adding a server

### stdio (local)

The `--` is mandatory: it separates Claude's flags from the server command.

```bash
claude mcp add <name> -- <command> [args...]
```

Real examples:

```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp
claude mcp add playwright -- npx -y @playwright/mcp@latest
claude mcp add --env AIRTABLE_API_KEY=YOUR_KEY airtable -- npx -y airtable-mcp-server
```

### http (remote)

```bash
claude mcp add --transport http <name> <url>
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### From JSON

```bash
claude mcp add-json weather-api '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'
```

## Scopes (the critical part)

```bash
claude mcp add --scope <scope> <name> -- <command>
```

| Scope | Stored in | Loads in | Shared with team? |
|---|---|---|---|
| `local` (default) | `~/.claude.json` | Current project only | ❌ |
| `project` | `.mcp.json` at project root | Current project | ✅ commit to git |
| `user` | `~/.claude.json` | All your projects | ❌ |

Precedence: `local` > `project` > `user`.

**Simple rule:**
- Your own personal tooling (notes, terminal) → `user`
- Project-specific tooling the team also needs → `project`
- Testing and experiments → `local`

A `.mcp.json` worth committing:

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    },
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp"
    }
  }
}
```

Environment variable expansion is supported — `${VAR}` and `${VAR:-default}` — so you never commit a key:

```json
{
  "mcpServers": {
    "db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub", "--dsn", "${DATABASE_URL}"]
    }
  }
}
```

## Managing servers

```bash
claude mcp list              # servers + connection status
claude mcp get <name>        # details for one server
claude mcp remove <name>     # remove
claude mcp login <name>      # OAuth
claude mcp logout <name>
```

In session:

```
/mcp        status, tools, OAuth login
```

## Using it

You do nothing special — just ask in natural language. Claude picks the right tool:

```
Review PR #456 and suggest improvements
Open the login page in the browser and take a screenshot
Fetch the Next.js App Router docs and write this route accordingly
```

## Tool names (for permission rules)

```
mcp__<server-name>__<tool-name>
```

In `settings.json`:

```json
{
  "permissions": {
    "allow": ["mcp__playwright__browser_navigate"],
    "deny":  ["mcp__github__delete_repository"]
  }
}
```

## Security warnings — say these in class

1. **An MCP server runs code.** Read the source before installing. An unknown server means full access to your machine.
2. **Prompt injection.** An MCP server can return text that looks like an instruction to the model. Treat MCP output as *data*, never as instructions.
3. **Never commit API keys into `.mcp.json`.** Use `${VAR}`.
4. **Remove servers you don't need.** Each server adds its tool definitions to context, burning tokens and making the model's tool choice harder.

## Building your own MCP server

```bash
npx -y @modelcontextprotocol/create-server my-server
```

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool(
  "get_weather",
  { city: z.string().describe("City name") },
  async ({ city }) => ({
    content: [{ type: "text", text: `Weather in ${city}: 25°C` }],
  })
);

await server.connect(new StdioServerTransport());
```

Connect it:

```bash
claude mcp add my-server -- node /path/to/server.js
```

## Sources

- MCP in Claude Code — https://code.claude.com/docs/en/mcp
- Official MCP specification — https://modelcontextprotocol.io
- SDKs — https://modelcontextprotocol.io/docs/sdk
- Official server list — https://github.com/modelcontextprotocol/servers
- MCP registry — https://registry.modelcontextprotocol.io
