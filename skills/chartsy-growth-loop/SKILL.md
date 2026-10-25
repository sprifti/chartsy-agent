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
1. Ask for a one- or two-line product pitch before anything else: what the
   product does, who it is for, and the main reason people buy. Chartsy only
   knows numbers and UTM tags, never what the product is or what a post
   said, so drafts written without a pitch come out generic. Skip the
   question only if the pitch is already in this conversation or the
   project's own docs. If the user only wants the report, no drafts, the
   pitch isn't needed.
2. If the user tracks more than one site, call list_tracked_sites and
   confirm which one.
3. Call get_revenue_by_source for the last 30 days with by=source, then
   by=campaign. Use by=content only when posts aren't flagged low_sample.
4. Check coverage. unmatched_signups counts everyone not linked to billing,
   which includes every signup who simply never paid, so a high number is
   normal. The real gap is unattributed_paying_customers: new paying
   customers that match no signup. If it is more than about 20% of the
   paying customers in the results, say so first: conclusions may be
   incomplete.
5. Judge carefully:
   - Treat low_sample groups as inconclusive, never as winners or losers.
   - Rank by current_mrr and signup_to_paid_rate_percent. Only weigh
     retention_rate_percent for signups older than about 90 days.
   - Call get_traffic_sources for the same window to spot channels that
     bring traffic but no paying customers.
6. Recommend 1-2 channels or themes to double down on and 1 to reduce,
   each with the numbers behind it.
7. Find what the winning posts actually said. Chartsy only has their
   utm_content slugs. If a publishing tool is connected, look the posts up
   there by their tagged links; otherwise ask the user to paste or link one
   or two of them. If neither is possible, say the drafts are based on the
   pitch and the slug names only.
8. Draft next week's posts for the recommended channels from the pitch and
   the winning posts' angle, with every link UTM-tagged. Create them as
   drafts in the publishing tool. Never publish or schedule live without
   the user's explicit approval in this conversation.
   - Publer: create every post as a draft (state "draft"). Never use
     state "scheduled": with no date, Publer publishes it immediately.
     Leave the date off the draft unless the user asks for one (a dated
     draft shows on the Publer calendar, an undated one waits in Drafts);
     give the planned date in your reply instead. Drafts are visible to
     everyone in the Publer workspace.
   - Postiz: create every post with type "draft" (schedulePostTool).
     Never use "schedule" or "now"; "now" publishes immediately. Postiz
     needs a date on every post, drafts included: use the planned date in
     UTC and repeat it in your reply. Set shortLink to false so the UTM
     tags stay readable. Write content as HTML. Put the tagged link in the
     first postsAndComments item: later items become a thread on X,
     Threads and Bluesky, and comments on LinkedIn and Facebook.

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
