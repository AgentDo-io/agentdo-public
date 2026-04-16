# MCP Connection Guide

AgentDo exposes a hosted **Model Context Protocol** (MCP) endpoint at `https://mcp.agentdo.io`. This guide shows how to connect from Claude Desktop, Cursor, n8n, and any other MCP-compatible client.

Our transport is HTTPS with JSON-RPC 2.0 payloads and per-request HMAC-authenticated API keys. Clients that expect a stdio-based MCP server (Claude Desktop, Cursor) use the official **`mcp-remote`** bridge to talk to us over HTTPS.

## 1. Get an API key

Sign up at https://agentdo.io/dashboard/auth/register, create a new API key in the dashboard, and store it securely. Keys are shown only once.

Every outbound call sent through `mcp-remote` (or any HTTPS client) must include `X-API-Key: <your_key>` as a header.

## 2. Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

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

Restart Claude Desktop. The 9 AgentDo tools (`search`, `book`, `cancel`, …) appear in the tools list.

> Why `mcp-remote`? Claude Desktop's native MCP transport is stdio; we're an HTTPS server. `mcp-remote` is the standard MCP ecosystem bridge that maps stdio ⇄ HTTPS and forwards authentication headers. You could also write a custom MCP client, but `mcp-remote` is the zero-friction path.

## 3. Cursor

In Cursor Settings → Features → MCP Servers, add:

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

(Cursor uses the same stdio MCP runtime as Claude Desktop, so the config shape is identical.)

## 4. n8n

n8n can talk to the REST API (`https://api.agentdo.io`) directly — use the **HTTP Request** node with:

- **Method:** `POST`
- **URL:** `https://api.agentdo.io/v1/search` (or any other endpoint from the [API reference](./api-reference.md))
- **Headers:** `X-API-Key: ak_your_key_here`, `Content-Type: application/json`
- **Body:** Tool-specific JSON — see API reference.

For full MCP JSON-RPC semantics inside n8n, the HTTP Request node can POST to `https://mcp.agentdo.io` with a JSON-RPC 2.0 envelope: `{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"search","arguments":{...}}}`.

## 5. Any other MCP-aware client

If your client supports a generic HTTPS MCP transport (some VS Code extensions, custom agents), point it at `https://mcp.agentdo.io` and ensure the `X-API-Key` header is sent on every request. `mcp-remote` is only needed when the client is stdio-only.

## Available tools

| Tool | Purpose |
|---|---|
| `search` | Find providers by category + location (radius search). |
| `lookup` | Named-entity resolution — resolve a specific business by name. |
| `availability` | List open slots for a provider. |
| `price_check` | Get the **listed price** of a standard service (e.g. "haircut"). |
| `quote` | Request a **custom quote** for a non-standard job (e.g. "fix a leaky pipe in the basement") — the provider replies with a price estimate. |
| `book` | Reserve an appointment slot. |
| `order` | Place a food or product order. |
| `cancel` | Cancel a booking, order, or quote request. |
| `status` | Check the status of a booking, order, or quote request. |

**`price_check` vs `quote`:** `price_check` is synchronous and returns a catalog price the provider has already published for a standard service. `quote` is asynchronous — you send a description and the provider responds (via webhook or polling) with a bespoke price and lead time. Use `price_check` for "haircut, massage, pizza delivery"; use `quote` for "plumbing, renovation, event catering".

See [`api-reference.md`](./api-reference.md) for full tool schemas.

## Stability & versioning

We follow [SemVer](https://semver.org) on the `mcp-manifest.json` version field. Our stability contract on the MCP surface:

- **Tool names** (`search`, `book`, …) are **stable within the v1 line**. We will never rename or remove a tool in v1.
- **Required input fields** on each tool are **stable within v1**. We will never change the type or name of a required field in v1.
- **New optional fields** on inputs and outputs may be added without a major bump — they are non-breaking by definition.
- **New tools** may be added without a major bump.
- Any breaking change triggers a major bump and a 6-month overlap where both versions are served.

Subscribe to the [CHANGELOG.md](../CHANGELOG.md) or the [public repo's Releases](https://github.com/AgentDo-io/agentdo-public/releases) to track changes.
