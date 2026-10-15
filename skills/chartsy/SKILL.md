---
name: chartsy
description: Read live SaaS revenue and website-traffic analytics from Chartsy - MRR, MRR by plan or country, revenue, churn, new subscribers, refunds, at-risk customers, top customers, revenue concentration, goals, site visits, signups and traffic sources. Use when the user asks how their business, subscriptions, revenue, churn or marketing channels are doing, wants a daily or weekly business review, or asks "what happened yesterday". Works through the hosted Chartsy MCP server (Stripe, Paddle, Polar, Lemon Squeezy, Creem, Freemius and custom sources).
homepage: https://chartsy.app
---

# Chartsy

Chartsy turns a SaaS company's payment-provider data (Stripe, Paddle, Polar, Lemon Squeezy,
Creem, Freemius or a custom source) and its website tracking into revenue and growth
analytics. This skill tells you how to answer business questions with the Chartsy MCP
tools: which tool to call, how few calls you can get away with, and how to read the
numbers without misreporting them.

The tools are **read-only**. Nothing you call can change the user's Chartsy account,
their payment-provider data or their customers. Nothing here can write to it either.
Don't offer to set goals, edit data or contact customers through Chartsy.

---

## Before anything: is Chartsy connected?

Look for tools named `get_business_snapshot`, `get_mrr`, `list_accounts` and so on (often
prefixed, e.g. `mcp__chartsy__get_mrr`). If they're missing, Chartsy isn't connected yet.
Tell the user how to connect it rather than guessing at numbers:

- **Server URL:** `https://mcp.chartsy.app/mcp` (streamable HTTP, OAuth sign-in)
- **Claude Code:** `claude mcp add --transport http -s user chartsy https://mcp.chartsy.app/mcp`, then `/mcp` to sign in
- **Claude / ChatGPT / Cursor / other clients:** add a custom connector or MCP server with that URL
- **Clients without OAuth:** generate a token in Chartsy under **Settings → Claude & MCP** and send it as `Authorization: Bearer <token>`

Requirements: a Chartsy login with a real payment provider connected and synced (the demo
account seeded at signup doesn't count), on any plan or the free trial.

---

## ⚠️ Hard rules (read first)

**Rule 1: Every tool call uses up one of the user's ChartsyAI questions.** Each call,
including `list_accounts` and calls that fail, counts against the same monthly allowance as
questions asked inside Chartsy. When it runs out, every tool returns an error until the 1st
of next month. So:
- For a broad question ("how's the business", "give me a review", "where's the risk"), make
  **one** `get_business_snapshot` call, not a dozen small calls.
- For "what happened yesterday / today / on Monday", make **one** `get_daily_snapshot` call.
- Don't call the same tool again for data you already have in this conversation. Reuse it.
- Don't fan out across many accounts, date windows or granularities unless the user asked.
  If the plan needs more than ~5 calls, say so and ask first.

**Rule 2: Name the account and the window you read.** Numbers without context are the
most common source of "that doesn't match my dashboard". Always say which account (by name)
and which date range or period the figures cover.

**Rule 3: Dates are `YYYY-MM-DD` and both ends are inclusive.** Work out relative dates
("last month", "Q1", "last 7 days") from today's real date before calling. For a single day,
pass the same date as `start_date` and `end_date`. Today is always a partial day.

**Rule 4: Never invent customer identities.** Customers come back as payment-processor IDs
only (e.g. `cus_…`, `ctm_…`). Chartsy never sends names or emails over MCP. Report the ID
and tell the user to look it up in Chartsy. Don't guess who it is.

**Rule 5: Missing is not zero.** A `null` field, an empty list or a `traffic: null` block
means "no data / not tracked", not "0". Say that it's unavailable. Don't report it as zero
or as a drop.

---

## Which tool answers what

| The user asks about… | Call | Notes |
| --- | --- | --- |
| Overall health, a review, "where's the risk", a dashboard | `get_business_snapshot` | One call covers everything below. `months` 1-36 (default 12) |
| A single day ("yesterday", "today", "last Monday") | `get_daily_snapshot` | `date` defaults to yesterday. Comes with prior-day and 7-day-average baselines |
| MRR right now | `get_mrr` | Single current value plus `currency` |
| MRR trend, by plan, week over week | `get_mrr_by_plan` | `granularity` day/week/month/quarter/year. The only MRR *series* |
| MRR by country | `get_mrr_by_country` | Current value only, top K (max 200) |
| Money received in a window, by plan / country / payment method | `get_revenue_breakdown` | `by` = `plan`, `country` or `payment_method` |
| Churn rate over time | `get_churn_rate` | Per period, customers-at-period-start basis |
| New subscribers over time | `get_new_subscribers` | By first subscription start date |
| Refunds, refund ratio by plan | `get_refund_summary` | Dates optional (all-time if omitted) |
| Customers likely to churn | `list_at_risk_customers` | 2+ failed payments this quarter, plus paused subscriptions |
| Biggest customers | `list_top_customers` | By lifetime revenue |
| Dependence on a few customers | `get_customer_concentration` | % of MRR from the top K |
| Goals / targets | `get_goal_progress` | Progress is measured from each goal's baseline, not from zero |
| Which sites are tracked | `list_tracked_sites` | Gives `site_id`s for the traffic tools |
| Visits, visitors, signups over time | `get_site_traffic` | `granularity` day/week/month only |
| Where traffic / signups come from | `get_traffic_sources` | `by` = source, medium, campaign, content, country, referrer, landing_path |
| Which channels / campaigns / posts brought **paying** customers, their MRR and churn | `get_revenue_by_source` | `by` = source, medium, campaign, content; `touch` = last (default) or first. For a planning loop see the `chartsy-growth-loop` skill |
| Tracked-event usage vs plan limit | `get_monthly_events_usage` | This month only |
| Which accounts exist | `list_accounts` | Only needed when the login has several |

If the snapshot already holds the answer to a follow-up question, answer from it. Only
call a narrower tool when you need something the snapshot doesn't have: a custom date range,
a different granularity, a `by` grouping it lacks, or more than its top K.

---

## Accounts and sites

- An **account** is one connected payment provider. With exactly one, every tool uses it
  automatically. Don't pass `account_id` and don't call `list_accounts`.
- With several, the tool returns an error listing them (`"…has N accounts, so account_id is
  required…"`). That error already names the accounts and their IDs, so pick from it. You
  don't need a `list_accounts` call. If the user's wording doesn't make the choice obvious,
  ask them which one they mean, by name.
- Accounts can have **different currencies** (`currency` on `list_accounts`/`get_mrr`).
  Never add or compare money across accounts in different currencies as if they were one.
- **Traffic tools are scoped to tracked sites, not accounts.** With no `site_id` they cover
  every site the login tracks, combined. A wrong `site_id` returns an empty list, not an
  error. If you get `[]` back for a `site_id`, check it against `list_tracked_sites` before
  telling the user there was no traffic. (`get_revenue_by_source` is the exception: a
  wrong `site_id` returns an error naming the problem.)

---

## Reading the numbers correctly

- **MRR is a run rate, not money received.** In `get_mrr_by_plan` each bucket is the MRR
  *as of* that period. A monthly plan shows its full amount in every week it's active.
  **Compare buckets, never sum them.** For "how much did we make", use `get_revenue_breakdown`.
- **Revenue is net**: what reached the account after tax and processor fees. Refunds are
  **not** subtracted from it. They're reported separately by `get_refund_summary`, on the
  same basis.
- **Revenue by plan uses the subscription's current plan.** A customer who switched plans
  has their older payments attributed to today's plan. Mention this when someone asks about
  a plan's historical revenue.
- **Churn is counted when paid access ends**, not when someone clicks cancel. A cancellation
  this week can show up as churn at the end of their billing period.
- **Goal `progress_pct`** counts from the goal's baseline: 0% means "no movement since the
  goal was set", not "zero MRR". It can go above 100.
- **Snapshots carry `flags`**: `{code, severity, message}` with severity `critical`,
  `warning` or `info`. Codes are stable (e.g. `revenue_concentration`, `churn_spikes`,
  `refund_ratio`, `revenue_drop`, `traffic_spike`, `partial_day`, `quiet_day`). Lead with
  `critical` and `warning` flags. Use the message text, which is written for humans, rather
  than paraphrasing loosely.
- **Daily snapshots compare against the previous day and the trailing 7-day average.** Say
  whether a day was normal for this business, not just its number. A `quiet_day` flag means
  nothing happened at all, which for a business that bills on some days is the calendar,
  not a collapse.

---

## Common workflows

**Business review** ("how are we doing", "monthly review", "build me a dashboard")
1. `get_business_snapshot` (pass `months` if the user named a window).
2. Report: current MRR and trend, new subscribers vs churn, top flags, refunds, goals, and
   traffic → signup conversion if `traffic` isn't null.
3. Offer specific follow-ups, but don't make the calls until the user picks one.

**Daily check-in** ("what happened yesterday", a morning brief)
1. `get_daily_snapshot` (no `date` = yesterday).
2. Lead with anything flagged, then revenue / signups / cancellations against the 7-day
   average.

**Churn investigation**
1. `get_churn_rate` over the window, monthly (weekly for a recent spike).
2. `list_at_risk_customers` for who's likely next.
3. If useful, `get_mrr_by_plan` to see which plans are shrinking.

**Marketing / channel question** ("which channel converts best", "where do signups come from")
1. `get_traffic_sources` with `by="source"` (or `medium` / `campaign` / `landing_path`).
2. Rank by signup rate *and* volume. A 50% rate on 4 visits isn't a channel.
3. If the question is about *paying* customers or revenue ("which channel brings customers
   who pay / stay"), use `get_revenue_by_source` instead. Treat `low_sample` groups as
   inconclusive.

**Week over week**
1. `get_mrr_by_plan` with `granularity="week"` and `periods` around 8-12.
2. `get_site_traffic` with `granularity="week"` if the user also cares about traffic.

---

## Errors and what to do

| Error says… | Meaning | Do |
| --- | --- | --- |
| "…no connected payment provider yet…" | Nothing to read | Tell the user to connect a provider in Chartsy and let the first sync finish |
| "…has N accounts, so account_id is required…" | Ambiguous account | Pick from the IDs in the message, or ask the user which account |
| "No Chartsy account with id …" | Wrong `account_id` | Use one of the IDs the message lists |
| "You've used all N ChartsyAI questions…" | Monthly allowance spent | Stop calling tools. Tell the user it resets on the 1st or they can upgrade |
| "…must be a date in YYYY-MM-DD format…" / "…must be one of…" | Bad argument | Fix the argument once. Don't retry blindly |
| "Chartsy could not complete …" | Server-side failure | Retry once at most, then suggest support@chartsy.app |
| 401 / connector needs re-authorising | Token expired or revoked, or trial/subscription lapsed | Ask the user to reconnect, or check **Settings → Claude & MCP** |

Never retry in a loop. Every retry uses up another question (Rule 1).

---

## Privacy

Everything returned is the user's own business data, scoped to the accounts and sites
their login owns. Treat it as confidential. Don't paste it into public places or other tools
unless the user asks. Customer rows are IDs only by design. Keep it that way.
