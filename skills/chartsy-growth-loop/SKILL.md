---
name: chartsy-growth-loop
description: Use when the user asks which marketing channels, campaigns or posts are working, or wants to plan social posts or content based on what brought paying customers. Pulls revenue by source from Chartsy and drafts next posts in the user's publishing tool for approval.
---

# Chartsy growth loop

Turn Chartsy revenue data into next week's marketing plan. Chartsy is
read-only. You recommend and draft; the user approves.

## Requirements
- Chartsy MCP connected (get_revenue_by_source, get_traffic_sources,
  list_tracked_sites).
- Optional: a publishing tool MCP (e.g. Postiz or Publer) for creating
  drafts. If none is connected, write the drafts in your reply instead.

## UTM convention
Tag every link you publish or draft:
- utm_source: platform (x, linkedin, reddit, threads, newsletter)
- utm_medium: social, email or community
- utm_campaign: theme or week, lowercase with hyphens (2026-w43-pricing)
- utm_content: a unique post slug (x-2026-10-15-pricing-thread)
Never reuse a utm_content value.

## Weekly loop
1. If the user tracks more than one site, call list_tracked_sites and
   confirm which one.
2. Call get_revenue_by_source for the last 30 days with by=source, then
   by=campaign. Use by=content only when posts aren't flagged low_sample.
3. Check coverage. unmatched_signups counts everyone not linked to billing,
   which includes every signup who simply never paid, so a high number is
   normal. The real gap is unattributed_paying_customers: new paying
   customers that match no signup. If it is more than about 20% of the
   paying customers in the results, say so first: conclusions may be
   incomplete.
4. Judge carefully:
   - Treat low_sample groups as inconclusive, never as winners or losers.
   - Rank by current_mrr and signup_to_paid_rate_percent. Only weigh
     retention_rate_percent for signups older than about 90 days.
   - Call get_traffic_sources for the same window to spot channels that
     bring traffic but no paying customers.
5. Recommend 1-2 channels or themes to double down on and 1 to reduce,
   each with the numbers behind it.
6. Draft next week's posts for the recommended channels, modeled on what
   worked, with every link UTM-tagged. Create them as drafts in the
   publishing tool. Never publish or schedule live without the user's
   explicit approval in this conversation.
   - Publer: use a private draft (state draft_private) when the tool
     offers it, so only the user sees it until they approve. Leave the
     date off the draft unless the user asks for one (a dated draft shows
     on the Publer calendar, an undated one waits in Drafts); give the
     planned date in your reply instead.

## Output
1. Results table: group, signups, paying customers, signup-to-paid rate,
   current MRR, churned MRR, low sample flag.
2. Recommendation in 3-5 sentences.
3. Drafts created: platform, planned date, link.
4. Coverage note, if relevant.

## Rules
- Don't call a channel or post a winner on fewer than 5 paying customers.
- Never estimate revenue. If revenue fields are null (e.g. Paddle Classic
  accounts), say revenue isn't available for that account.
- Never ask for or reveal customer names or emails.
