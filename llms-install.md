# Installing Chartsy for an AI agent

Chartsy is a hosted, read-only MCP server. You don't clone, build or run anything locally.

## 1. Add the server

Add this entry to the client's MCP settings (for Cline: `cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "chartsy": {
      "type": "streamableHttp",
      "url": "https://mcp.chartsy.app/mcp"
    }
  }
}
```

If the client can only start local (stdio) servers, bridge to the hosted server with `mcp-remote`. It handles the sign-in in the user's browser:

```json
{
  "mcpServers": {
    "chartsy": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.chartsy.app/mcp"]
    }
  }
}
```

## 2. Sign in

The server uses OAuth. On first use, the client opens a browser window. The user signs in with their Chartsy account and clicks Approve. There's no API key to ask for.

For an agent with no browser (a server, a scheduled job, CI), the user generates a token in Chartsy under **Settings → Claude & MCP → Generate token**. Send it on every request:

```json
{
  "mcpServers": {
    "chartsy": {
      "type": "streamableHttp",
      "url": "https://mcp.chartsy.app/mcp",
      "headers": { "Authorization": "Bearer <token>" }
    }
  }
}
```

Never write a real token into a file that gets committed.

## 3. Check it works

Call `list_accounts`. It should return the user's connected billing accounts. If it fails with an access error, the user needs a Chartsy account with a payment provider connected: https://chartsy.app

## Notes

- All 19 tools are read-only.
- Each tool call uses one ChartsyAI question from the user's monthly allowance.
- The skill in `skills/chartsy/SKILL.md` explains which tool answers which question.
