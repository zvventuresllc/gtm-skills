---
name: "warm-intro-finder"
description: "For a target list in Hivekind (account names, a tag, or an ICP grade), find who in the user's or team's LinkedIn network can introduce them, ranked by relationship strength. Optionally creates intro-request tasks. Use when asked for warm intros, who knows someone at a company, or intro paths."
---

# Warm intro finder

Rank target accounts by who on the team can introduce the user, using consented LinkedIn and partner evidence in Hivekind. Works for any Hivekind user; nothing about the user is hardcoded.

## Setup

1. Find the Hivekind tools with ToolSearch (query "Hivekind domain"). Names vary by install (mcp__Hivekind__domain_* or mcp__plugin_hivekind_hivekind__domain_*). If no Hivekind domain tools are found, tell the user to add the Hivekind connector at https://api-v2.hive-loop.com/mcp and stop.
2. Identify the user. Call workspace.status.get.v2 (workspace domain) for company_domain, and linkedin.identities.list.v2 (network domain) for the user's own identity (is_self) and the teammates whose networks are shared. INTERNAL_DOMAIN = company_domain, or the user's email domain.
3. Hivekind is V2 domain discovery: discover -> describe -> invoke. Read each operation's schema before invoking. Writes need a new operation_id UUID. Never guess IDs or pass workspace_id/user_id.
4. Domains used: accounts, network, intelligence, tasks.

## Step 1 - Resolve the target list

The user gives one of:
- Account names or domains: accounts.search.v2 per name (match_mode "exact" on domain, then "prefix" on name). List any name that does not resolve.
- A tag: check the exact tag name with accounts.tags.list.v2, then accounts.search.v2 with tags [<tag>].
- An ICP grade or tier: accounts.search.v2 with icp_grades and/or account_tiers.

Cap at 100 accounts per run; if more match, ask whether to narrow or take the top 100 by ICP score. Keep account_id, name, domain, ICP grade, owner.

## Step 2 - Coverage

- analytics.relationships.coverage.v2 (intelligence domain) for the overall share of targets with a consented LinkedIn match. Report it once at the top. Unmatched is not proof an account is cold.
- accounts.search.v2 with the same account_ids and sort_by network_strength_then_icp gives each account's network_strength (0 to 3), linkedin_member_count, partner_count and partner_names.

## Step 3 - Paths per account

For each account with network_strength of 1 or more, best first:

- linkedin.connections.accounts.search.v2 with owner "me" and company_id = the account, then once per sharing teammate (owner = their user_id). Collect matching_contact_ids per owner.
- network.paths.get.v2 and relationships.batch.get.v2 with those contact_ids to confirm the state (connected, syncing, not shared, unknown) and who holds each connection.
- network.account.access.get.v2 for partner routes and excluded routes.
- Optional: accounts.buyer_groups.get.v2 to mark which connected contacts are on the buying committee.

## Step 4 - Rank

Score each path, highest first:
1. The user is connected directly to a buying-group member.
2. A teammate is connected to a buying-group member.
3. A teammate or the user is connected to anyone else at the account.
4. A Channel Partner route.

Within a tier, rank by network_strength, then number of distinct connectors, then ICP score. Paths in a syncing or unknown state go in a separate "unconfirmed" group.

## Step 5 - Output

A ranked table, one row per account:

| Account | ICP | Best connector | Connected to (title) | Other paths | Strength |

Then the unconfirmed group and accounts with no known path. Link records with web_url when returned.

## Step 6 - Optional intro-request tasks

- Ask whether to create intro-request tasks for the top rows, and for which ones.
- Before creating, check tasks.list.v2 for open tasks on the same account so a rerun does not duplicate.
- Create with tasks.create.batch.v2 (up to 20 per call). Owner: the user. Each task on the account: title "Ask <connector> for an intro to <contact>"; objective and done_when stated plainly; due 2 business days out, never a weekend; decision rationale citing the path evidence.
- Do not message the connector or the target. Do not schedule LinkedIn actions. If the user wants to message someone, preview it with linkedin.action.preview.v2 and confirm the exact text and recipient before any schedule call.

## Rules

- Read only, except tasks the user approves.
- Never delete, merge or overwrite records. Never send messages or invitations without explicit confirmation.
- Use only consented relationship evidence. Do not infer permission to message from a connection.
- Credit-billed operations are fine; do not warn about cost.

## Style

Plain and direct. No filler, puffery, "moreover/furthermore/additionally/notably", "not X but Y", strained metaphors, or words like "drives"/"carries"/"load bearing". Minimal bold, sentence-case headers, no emoji.

## Report (under 10 lines)

Targets resolved and unresolved; coverage share; accounts with confirmed paths, unconfirmed, and none; tasks created (title + due date); anything skipped.
