---
name: "event-follow-up"
description: "After a conference or event, import or find it in Hivekind, grade the roster against the ICP, pull attendees at a minimum grade, discover more contacts at top accounts, tag the accounts, and create follow-up tasks. Use when asked to follow up on an event, conference, trade show or workshop."
---

# Event follow-up

Turn an event roster into graded accounts, tagged and with follow-up tasks, in the user's Hivekind workspace. Works for any Hivekind user; nothing about the user is hardcoded.

## Setup

1. Find the Hivekind tools with ToolSearch (query "Hivekind domain"). Names vary by install (mcp__Hivekind__domain_* or mcp__plugin_hivekind_hivekind__domain_*). If no Hivekind domain tools are found, tell the user to add the Hivekind connector at https://api-v2.hive-loop.com/mcp and stop.
2. Identify the user. Call workspace.status.get.v2 (workspace domain) for company_domain; linkedin.identities.list.v2 (network domain) is_self for the user's name. INTERNAL_DOMAIN = company_domain, or the user's email domain. Attendees from INTERNAL_DOMAIN are staff, not prospects.
3. Hivekind is V2 domain discovery: discover -> describe -> invoke. Read each operation's schema before invoking. Writes need a new operation_id UUID; reuse it only to retry the same write. Never guess IDs or pass workspace_id/user_id. Poll queued jobs with jobs.status.get.v2 (workspace domain) until terminal; partial is not complete.
4. Domains used: events, accounts, tasks, workspace.

## Step 1 - Find or import the event

- events.search.v2 with the event name. If it exists, use its event_id.
- If not and the user attached a roster (CSV or sheet): events.import.preview.v2 with the whole file. Show the column mapping, row counts and how many companies and people already exist. Then events.import.start.v2 with the exact import_token (nothing else alongside it) and source.kind event, workshop or webinar. Poll until done.
- A list of people with no event goes through prospects.import.* in the accounts domain instead.
- Ask for a minimum grade if the user did not give one (default B).

## Step 2 - Grade the roster

- Check events.scoring_status.get.v2. If the roster is not graded, run events.score.v2 and poll events.scoring_status.get.v2 until done. Report the grade spread.

## Step 3 - Pull the roster at the minimum grade

- events.roster.get.v2 with min_grade = the chosen grade. Drop staff from INTERNAL_DOMAIN.
- Group rows by account. Rank accounts by grade, then by number of attendees.

## Step 4 - Contacts at top accounts

- For attendees worth pursuing who are not yet real contacts: events.participant.promote.v2 per row. If contact details are missing, events.enrich.v2 on those rows first and poll events.enrichment_status.get.v2.
- For the top accounts (default top 10, or what the user says) where one attendee is not enough: events.participant.discover_contacts.v2 to find other people at the company. Poll the job.
- Before promoting, search contacts by email and full name so existing contacts are reused.

## Step 5 - Tag the accounts

- accounts.tags.list.v2 to reuse an existing tag name for the event (for example "<Event name> <year>"). Do not create a near-duplicate.
- accounts.tags.apply.v2 with event_id or the chosen account_ids, dry_run true first. Show how many accounts would change, then apply.

## Step 6 - Follow-up tasks

- Check tasks.list.v2 for open tasks on each account so a rerun does not duplicate.
- tasks.create.batch.v2 (up to 20 per call). Owner: the account owner if set, otherwise the user. One task per account: title "Follow up from <event> with <attendee>"; objective names who attended and their grade; due 1 to 3 business days after the event, never a weekend; decision rationale citing the event and grade.
- Ask before creating more than 20 tasks.

## Rules

- No sends and no campaign launches. If the user wants outreach, hand off to the launch-outbound-campaign skill.
- Never delete, merge or overwrite records. Search before creating.
- Event assignment previews write suggestions; do not run events.assignments.preview.v2 or confirm unless the user asks.
- Credit-billed operations (grading, enrichment, promote, discover) are fine; do not warn about cost.

## Style

Plain and direct. No filler, puffery, "moreover/furthermore/additionally/notably", "not X but Y", strained metaphors, or words like "drives"/"carries"/"load bearing". Minimal bold, sentence-case headers, no emoji.

## Report (under 10 lines)

Event used or imported; rows and grade spread; accounts at or above the minimum grade; contacts promoted and discovered; tag applied and account count; tasks created (count, due dates); anything that failed or needs review.
