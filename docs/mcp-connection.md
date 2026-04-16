# MCP Connection Guide

AgentDo exposes a Model Context Protocol (MCP) endpoint at `https://mcp.agentdo.io`. This guide shows how to connect from Claude Desktop, n8n, and Cursor.

## 1. Get an API key

Sign up at https://agentdo.io/dashboard/auth/register, create a new API key in the dashboard, and store it securely. Keys are shown only once.

## 2. Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "agentdo": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-fetch", "https://mcp.agentdo.io"],
      "env": {
        "X-API-Key": "ak_your_key_here"
      }
    }
  }
}
```

Restart Claude Desktop. The 9 AgentDo tools (`search`, `book`, `cancel`, …) appear in the tools list.

## 3. n8n

Add an HTTP Request node:
- **Method:** `POST`
- **URL:** `https://mcp.agentdo.io`
- **Headers:** `X-API-Key: ak_your_key_here`, `Content-Type: application/json`
- **Body:** JSON-RPC 2.0 envelope (see API reference)

## 4. Cursor

In Cursor Settings → MCP Servers, add:

```json
{
  "agentdo": {
    "url": "https://mcp.agentdo.io",
    "headers": { "X-API-Key": "ak_your_key_here" }
  }
}
```

## Available tools

| Tool | Purpose |
|---|---|
| `search` | Find providers by category + location |
| `lookup` | Named-entity resolution (find a specific business) |
| `availability` | Check open slots for a provider |
| `book` | Reserve an appointment slot |
| `order` | Place a food/product order |
| `quote` | Request a price quote |
| `price_check` | Get current pricing |
| `cancel` | Cancel a booking or order |
| `status` | Check the status of a booking/order |

See [`api-reference.md`](./api-reference.md) for full tool schemas.
