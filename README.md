# AgentDo

> **AI-native service marketplace.** Let your AI agent search and book real-world services — restaurants, salons, appointments — through a single MCP endpoint.

Point Claude (or any MCP-compatible agent) at `https://mcp.agentdo.io`, add your API key, and the agent can call 9 tools to discover providers, check availability, request quotes, book appointments, and more. No per-call fees. Providers pay only for delivered bookings.

## Connect in 30 seconds

**Claude Desktop** — edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "agentdo": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.agentdo.io",
        "--header",
        "X-API-Key:ak_your_key_here"
      ]
    }
  }
}
```

Restart Claude Desktop. The AgentDo tools appear in the tools list.

> **Why `mcp-remote`?** AgentDo is a hosted HTTP MCP server; Claude Desktop's native MCP transport is stdio, so we use the `mcp-remote` bridge (part of the official MCP ecosystem) to forward calls over HTTPS with your API key attached as a header.

Get an API key at https://agentdo.io/dashboard. See [`docs/mcp-connection.md`](./docs/mcp-connection.md) for n8n, Cursor, and other clients.

## Tools

| Tool | What it does |
|---|---|
| `search` | Find providers by category + location (radius search) |
| `lookup` | Named-entity resolution — resolve a specific business by name |
| `availability` | List open time slots for a provider |
| `book` | Reserve an appointment slot |
| `order` | Place a food or product order |
| `quote` | Request a price quote for a custom service |
| `price_check` | Get current pricing for a known service |
| `cancel` | Cancel a booking or order |
| `status` | Check the status of a booking, order, or quote |

Full schemas in [`docs/api-reference.md`](./docs/api-reference.md).

## Endpoints

| | URL |
|---|---|
| **MCP** (primary) | `https://mcp.agentdo.io` |
| **REST API** (alternative) | `https://api.agentdo.io` |
| **Dashboard** (keys, billing) | `https://agentdo.io` |
| **Info & sign-up** | `https://agentdo.me` |

## Documentation

- [MCP connection guide](./docs/mcp-connection.md) — Claude Desktop, n8n, Cursor
- [API reference](./docs/api-reference.md) — REST endpoints, request/response schemas
- [MCP manifest](./mcp-manifest.json) — one-click add for supported clients
- [Changelog](./CHANGELOG.md) — MCP / API surface changes

## Pricing

| Who | Pay for |
|---|---|
| **AI agents / developers** | Free up to 1000 API calls/hour. No per-call charge. |
| **Service providers** | A small fee per **delivered** booking — never per click, never per query. Currency and amount configurable in the provider dashboard. |

## Status

Pre-launch. Public Early Access sign-ups at https://agentdo.me.

## License

Artifacts in this repository (docs, manifest, API specs) are released for integration and documentation purposes. The underlying AgentDo platform is proprietary; source lives in a separate, private repository.
