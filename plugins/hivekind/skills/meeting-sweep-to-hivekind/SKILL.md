---
name: "meeting-sweep-to-hivekind"
description: "Sweep today's meeting notes from Granola or Google Meet's Gemini notes into Hivekind as contacts, accounts, debrief notes and follow-up tasks. Use when asked to sweep, log or sync meetings into Hivekind."
---

# Meeting sweep to Hivekind

Sweep TODAY's meetings from the user's notetaker and update their Hivekind workspace. Works for any Hivekind user; nothing about the user is hardcoded.

## Setup

1. Identify the user. Call the Hivekind workspace domain (discover "current user" / "whoami") to get the user's name and email. If that fails, use the email of the connected Google or Granola account. Set:
   - OWNER_EMAIL = the user's email
   - INTERNAL_DOMAIN = the domain of OWNER_EMAIL (treat free-mail domains as having no internal domain)
   - TZ = the user's time zone if known, otherwise ask once; in an unattended run, use America/Los_Angeles and say so in the report.
2. Get today's date in TZ (YYYY-MM-DD).
3. Pick the notes source. Load tools with ToolSearch and use whichever is connected:
   - Granola: list_meetings / search_meetings, get_meetings, get_meeting_transcript. SOURCE = "Granola".
   - Gemini (Google Meet): Google Drive (search_files, read_file_content, get_file_metadata) and Google Calendar (list_events, get_event). SOURCE = "Gemini".
   - If both are connected, sweep both and de-duplicate meetings by title and start time. If neither is connected, tell the user which connector to add and stop.
4. Load the Hivekind domain tools with ToolSearch (search "Hivekind domain"). Names depend on how Hivekind was connected, e.g. mcp__Hivekind__domain_accounts_v2 for a custom connector or mcp__plugin_hivekind_hivekind__domain_accounts_v2 via this plugin. You need the accounts, tasks and workspace domains. If none are found, tell the user to connect Hivekind (MCP URL https://api-v2.hive-loop.com/mcp) and stop.
5. Hivekind is V2 domain discovery. For each operation: call the domain with params={action:"discover",query:"<task>"}, then params={action:"describe",operation:"<exact .v2 ID>"} and read its schema, prerequisites and effects, then params={action:"invoke",operation:"<ID>",arguments:{...},operation_id:"<new UUID>"}. Reuse a UUID only to retry the same write. Never guess IDs or pass workspace_id/user_id. Poll queued jobs through the workspace domain until done.
6. Operations you will need: contact and account search, notes list/create (accounts domain), prospects.import.start.v2 (accounts domain), tasks.create.batch.v2 (tasks domain).

## Step 1 - Meetings

Granola:
- List today's meetings, then get_meetings for full notes (transcript only if notes are thin). MEETING_ID = Granola note ID.
- Skip meetings in a folder named Private.

Gemini:
- Search Drive for Google Docs created today whose title contains "Notes by Gemini" (usually in "Meet Recordings"). Open the matching "Transcript" doc only if notes are thin. MEETING_ID = Drive file ID.
- Match each doc to its Calendar event (title and start time) for attendee emails; prefer the calendar's emails.
- Skip meetings whose calendar event is marked Private or whose doc is in a folder named Private.
- Gemini only saves notes to the organizer's Drive; mention in the report that meetings organized by others are missed unless the notes were shared.

Both sources:
- If no meetings, report "No <SOURCE> meetings today" and stop.
- Skip any meeting whose title or text says "Private - Skip". THIS IS A MUST.
- Skip a meeting if Hivekind already has a note titled "<SOURCE> debrief <today>: <meeting title>" (or containing its MEETING_ID) on any attendee's contact or account. Check with notes list before writing.

## Step 2 - Contacts and accounts

- External attendees = any email not @INTERNAL_DOMAIN. Skip the user and internal staff.
- Duplicate check before creating: search contacts by email, full name, and last name (confirm first name); for corporate domains also search by email domain; search accounts by domain (contains) and by the first word of the company name. Company names in Hivekind often differ from the domain. Same email or same full name = existing contact; reuse it.
- Free-mail domains (gmail, outlook, yahoo, icloud, etc.): take the company from the meeting notes; if unknown, use "<Full Name> (independent)" with no domain, and flag it.
- Only for contacts that do not exist: create with prospects.import.start.v2, owner OWNER_EMAIL, source "<SOURCE>" (kind: other). Reuse an existing source of that name if describe/discover shows one. Do not import someone who exists under a different email or company; note them only and flag it.
- Resolve final contact_id and account_id after the import.

## Step 3 - Notes

- One note per meeting. Title: "<SOURCE> debrief <YYYY-MM-DD>: <meeting title>". Put it on the contact for person-level discussions, or on the account when the discussion was about the company (deal, pricing, partnership, multiple attendees from the same company). Do not duplicate across both.
- Body (markdown, short): MEETING_ID (and Drive link for Gemini); attendees; what was learned; decisions; next steps with owners and dates if stated.
- Business content only. Leave out personal, health, family, financial-personal or other sensitive details unrelated to the deal, since notes are visible to the whole workspace. State only what the notes show. Gemini's "Suggested next steps" can misattribute owners; confirm against the notes body or transcript.

## Step 4 - Tasks

- Use tasks.create.batch.v2, one batch per run where possible. Owner: OWNER_EMAIL.
- Create tasks only for explicit next steps the user or their company owns. Skip actions owned only by the other party, unless a check-in makes sense (e.g. "they'll send the contract by Wednesday" -> check in Thursday).
- Due date: the date stated in the meeting; otherwise a realistic business day (follow-up emails 1-2 business days, deliverables 3-5, check-ins the day after the other party's deadline). Never weekends.
- Link each task to the contact or account. Decision rationale: one line on why the task exists, citing MEETING_ID.
- Before creating, check existing open tasks for the same contact/account so a rerun does not duplicate.

## Rules

Never delete, merge or overwrite existing records. No sends, campaigns or paid operations.

## Style

Plain and direct. No filler, puffery, "moreover/furthermore/additionally/notably", "not X but Y", strained metaphors, or words like "drives"/"carries"/"load bearing". Minimal bold, sentence-case headers, no emoji.

## Report (under 10 lines)

Source(s) used; meetings processed and skipped; contacts/accounts created (names); notes added; tasks created (title + due date); anything ambiguous (company guesses, existing records not imported, unclear owners, failed writes, time zone assumed).