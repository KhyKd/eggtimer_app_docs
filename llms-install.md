# Installing the Eggtimer.app MCP server

Eggtimer.app is a hosted remote MCP server. There is nothing to clone, build or run locally.

1. Add this entry to the client's MCP settings (for Cline: `cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "eggtimer": {
      "type": "streamableHttp",
      "url": "https://eggtimer.app/api/mcp"
    }
  }
}
```

2. No API key or environment variable is needed.
3. Verify: call `get_capabilities`, then `create_timer` with `{"duration": "12m", "label": "Tea"}`. Expected URL: `https://eggtimer.app/Tea/for/720s`.

Optional authenticated server, only if the user already has an Eggtimer.app workspace API key (starts with `ew_`): URL `https://eggtimer.app/api/mcp/server`, header `Authorization: Bearer <key>`. Ask the user for the key; never invent one, and never use an `er_` recovery key.

Tool reference: [docs/tools.md](docs/tools.md).
