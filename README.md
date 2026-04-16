# AgentDo

> **AI-native service marketplace.** Let your AI agent search and book real-world services — restaurants, salons, appointments — through a single MCP endpoint.

Point Claude (or any MCP-compatible agent) at `https://mcp.agentdo.io`, add your API key, and the agent can call 9 tools to discover providers, check availability, request quotes, book appointments, and more. No per-call fees. Providers pay only for delivered bookings.

## Why AgentDo

The AI-agent ecosystem treats booking and ordering as solved problems — until the agent actually has to pick up the phone. Today, "book me a haircut for Thursday" routes through a web form, a phone number, or a dead end. We're building the missing marketplace layer so that any AI assistant can transact with any local service provider, safely, in one JSON-RPC call.

Built by a small team in Budapest; initial vertical: beauty, wellness, dining, and on-demand professional services in Central Europe. Commercial launch H2 2026.

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
| `price_check` | Get the **listed price** of a standard service (e.g. "haircut 30 min") |
| `quote` | Request a **custom quote** for a non-standard job (e.g. "fix a leaking pipe in the basement") — provider replies async |
| `book` | Reserve an appointment slot |
| `order` | Place a food or product order |
| `cancel` | Cancel a booking, order, or quote request |
| `status` | Check the status of a booking, order, or quote |

**`price_check` vs `quote`:** use `price_check` when the service has a published catalog price (haircut, pizza delivery, massage); use `quote` when the price depends on the job (plumbing, renovation, catering).

Full schemas in [`docs/api-reference.md`](./docs/api-reference.md).

## Stability & versioning

We commit to the following within the v1 line of the MCP surface:

- **Tool names** (`search`, `book`, …) are stable — no renames, no removals.
- **Required input fields** on each tool are stable — no type or name changes.
- **New optional fields** on inputs and outputs may be added without a major bump. Non-breaking by definition.
- **New tools** may be added without a major bump.

Breaking changes trigger a major version bump (v2) and a 6-month overlap period where both versions are served. Subscribe to [releases](https://github.com/AgentDo-io/agentdo-public/releases) or [`CHANGELOG.md`](./CHANGELOG.md) to track changes.

## Endpoints

| | URL |
|---|---|
| **MCP** (primary) | `https://mcp.agentdo.io` |
| **REST API** (alternative) | `https://api.agentdo.io` |
| **Dashboard** (keys, billing) | `https://agentdo.io` |
| **Info & sign-up** | `https://agentdo.me` |

## Documentation

- [MCP connection guide](./docs/mcp-connection.md) — Claude Desktop, Cursor, n8n
- [API reference](./docs/api-reference.md) — REST endpoints, request/response schemas
- [Security & data handling](./docs/security.md) — GDPR, PII, retention, HMAC signatures
- [MCP manifest](./mcp-manifest.json) — one-click add for supported clients
- [Changelog](./CHANGELOG.md) — MCP / API surface changes

## Pricing

| Who | Pay for |
|---|---|
| **AI agents / developers** | Free up to 1000 API calls/hour. No per-call charge. |
| **Service providers** | A small fee per **delivered** booking — never per click, never per query. Currency and amount configurable in the provider dashboard (HUF, EUR, USD). |

## Status

Pre-launch. Our Phase 1 inventory focuses on Central European beauty, wellness, and dining verticals; we're onboarding early partners through a CRM integration spec (see [`docs/mcp-connection.md`](./docs/mcp-connection.md) and the [Partner Integration section of our developer docs](https://agentdo.io/docs#provider-integration)). Public Early Access sign-ups at https://agentdo.me.

If you're building an agent that books or orders services, talk to us before you roll your own integration — we'll fast-track API keys and supply you with a real (non-sandbox) pool of providers in your region.

## License

Artifacts in this repository (docs, manifest, API specs) are released for integration and documentation purposes. The underlying AgentDo platform is proprietary; source lives in a separate, private repository.
