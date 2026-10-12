# Chartsy for AI agents

Ask your AI agent about your SaaS business: MRR, churn, revenue, refunds, at-risk
customers, goals, site traffic and signups. The numbers are live from
[Chartsy](https://chartsy.app), which reads Stripe, Paddle, Polar, Lemon Squeezy, Creem,
Freemius or a custom source.

This repo bundles two things every agent install points at:

- **The hosted Chartsy MCP server**: `https://mcp.chartsy.app/mcp`. It uses streamable
  HTTP and OAuth sign-in, and it's **read-only**. Nothing to install or run locally.
- **The `chartsy` skill** ([`skills/chartsy/SKILL.md`](skills/chartsy/SKILL.md)). It
  teaches the agent which tool answers which question, how to keep tool calls to a minimum,
  and how to read the numbers correctly.

> **Usage:** every tool call counts as one ChartsyAI question against your plan's monthly
> allowance, the same budget as questions asked inside Chartsy. The skill steers agents to
> one-call answers (`get_business_snapshot`, `get_daily_snapshot`) wherever possible.

**You need:** a Chartsy account (any plan, including the free trial) with a payment
provider connected and synced.

---

## Install

### Claude (claude.ai, desktop, mobile)
**Settings → Connectors → Add custom connector**, then paste `https://mcp.chartsy.app/mcp`
and sign in to Chartsy.

### Claude Code
Plugin (MCP server and skill):
```bash
/plugin marketplace add sprifti/chartsy-agent
/plugin install chartsy@chartsy
```
Or just the server:
```bash
claude mcp add --transport http -s user chartsy https://mcp.chartsy.app/mcp
```
Then run `/mcp` to sign in.

### Cursor
[Add to Cursor](cursor://anysphere.cursor-deeplink/mcp/install?name=chartsy&config=eyJ1cmwiOiJodHRwczovL21jcC5jaGFydHN5LmFwcC9tY3AifQ==).
Or go to **Settings → MCP → Add server**, choose type **HTTP**, and use URL
`https://mcp.chartsy.app/mcp`.

### ChatGPT
**Settings → Connectors** (Developer mode) → add `https://mcp.chartsy.app/mcp` and sign
in to Chartsy.

### Gemini CLI
```bash
gemini extensions install https://github.com/sprifti/chartsy-agent
```

### Codex, OpenClaw and other skill-aware agents
```bash
npx skills add sprifti/chartsy-agent
```
The skill needs the MCP server connected in the same agent. Add
`https://mcp.chartsy.app/mcp` in that agent's MCP settings.

### Any other MCP client
Use [`mcp.json`](mcp.json), or point the client at `https://mcp.chartsy.app/mcp`
(streamable HTTP, OAuth sign-in).

### Clients that only run local (stdio) servers
Older clients can't connect to a hosted server directly. Bridge to it with
[`mcp-remote`](https://www.npmjs.com/package/mcp-remote), which handles the OAuth
sign-in in your browser:
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

### Agents that run without a browser (servers, cron jobs, CI)
OAuth needs someone to click **Approve** in a browser. For an agent running unattended,
such as OpenClaw on a VPS, a scheduled job or a CI pipeline, use a personal token instead:

1. In Chartsy, go to **Settings → Claude & MCP** → **Generate token**. It's shown once,
   so copy it then.
2. Store it as a secret (e.g. `CHARTSY_MCP_TOKEN`). Never commit it.
3. Send it on every request:
   ```
   Authorization: Bearer <your-token>
   ```
   For example with Claude Code:
   ```bash
   claude mcp add --transport http -s user chartsy https://mcp.chartsy.app/mcp \
     --header "Authorization: Bearer $CHARTSY_MCP_TOKEN"
   ```
   Or through `mcp-remote`: `npx -y mcp-remote https://mcp.chartsy.app/mcp --header "Authorization: Bearer ${CHARTSY_MCP_TOKEN}"`

Tokens last a year and can be revoked on the same tab. An unattended agent uses up ChartsyAI
questions like any other client, one per tool call, so schedule it sensibly. A daily
`get_daily_snapshot` is one question a day.

---

## Things to ask

- *"Give me a full business review. Where's the risk?"*
- *"What happened in my business yesterday?"*
- *"How has MRR moved by plan over the last 6 months?"*
- *"Which customers are at risk of churning?"*
- *"Which traffic source converts best to signups?"*
- *"What did we refund in Q1, and which plan has the worst refund ratio?"*

## Tools

All 18 tools are read-only and scoped to the accounts and sites your login owns.

| Area | Tools |
| --- | --- |
| Everything at once | `get_business_snapshot`, `get_daily_snapshot` |
| Revenue | `get_mrr`, `get_mrr_by_plan`, `get_mrr_by_country`, `get_revenue_breakdown`, `get_refund_summary`, `get_goal_progress` |
| Customers and churn | `get_churn_rate`, `get_new_subscribers`, `list_at_risk_customers`, `list_top_customers`, `get_customer_concentration` |
| Site traffic | `list_tracked_sites`, `get_site_traffic`, `get_traffic_sources`, `get_monthly_events_usage` |
| Accounts | `list_accounts` |

## Privacy

Customers are identified by payment-processor ID only. Names and emails never leave
Chartsy. Access is checked on every call, so revoking a token or ending a subscription
takes effect immediately. Policy: https://chartsy.app/privacy-policy

## Repo layout

| Path | Used by |
| --- | --- |
| `skills/chartsy/SKILL.md` | All skill-aware agents (single source of truth) |
| `.claude-plugin/`, `.mcp.json` | Claude Code plugin and marketplace |
| `.cursor-plugin/` | Cursor plugin and marketplace |
| `gemini-extension.json` | Gemini CLI extension |
| `mcp.json` | Generic `mcpServers` config for other clients |

Keep `version` in step across `.claude-plugin/`, `.cursor-plugin/` and
`gemini-extension.json` when you release.

Support: support@chartsy.app

## License

Copyright © 2026 Chartsy. All rights reserved. See [LICENSE](LICENSE). You may install these
files to connect your own agent to Chartsy; no other rights are granted.
