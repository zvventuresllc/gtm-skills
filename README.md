# gtm-skills

GTM system skills for Claude, packaged as a plugin marketplace.

## Plugins

| Plugin | Skills | Requires |
|---|---|---|
| `hivekind` | `meeting-sweep-to-hivekind`: sweeps today's Granola or Gemini (Google Meet) notes into Hivekind contacts, accounts, debrief notes and tasks | Hivekind connector, plus Granola or Google Drive and Google Calendar |
| | `founder-daily-brief`: one morning list of replies waiting, unconverted action items, overdue tasks, stage changes and new signals; offers to turn action items into tasks | Hivekind connector |
| | `meeting-prep-brief`: brief before a call with account history, buying group, ICP score, signals and intro paths | Hivekind connector; Google Calendar or Granola optional |
| | `warm-intro-finder`: ranks a target list by who on the team can introduce you; optionally creates intro-request tasks | Hivekind connector with LinkedIn synced |
| | `event-follow-up`: imports or finds an event, grades the roster, promotes and discovers contacts, tags accounts and creates follow-up tasks | Hivekind connector |
| | `launch-outbound-campaign`: builds the audience, launches the campaign shell, generates sample drafts and checks sendability; asks before any send | Hivekind connector |

## Install

In Claude Code:

```
/plugin marketplace add zvventuresllc/gtm-skills
/plugin install hivekind@gtm-skills
```

The plugin connects the Hivekind MCP server (`https://api-v2.hive-loop.com/mcp`). Sign in to Hivekind and pick your workspace when prompted.

In the Claude app, add this repository as a plugin marketplace from the plugin settings, then install `hivekind`. If Hivekind is not already connected:

1. Open Customize → Connectors → Add custom connector.
2. Paste the MCP URL: `https://api-v2.hive-loop.com/mcp`
3. Click Connect, sign in to Hivekind, pick your workspace, and Allow.
4. Enable the Hivekind connector in the chat before asking it to use tools.

Connect Granola, or Google Drive and Google Calendar for Gemini notes, the same way.

## Use

Ask Claude to "sweep today's meetings into Hivekind". The skill reads your identity from Hivekind, treats your email domain as internal, and uses whichever notetaker is connected. Meetings marked "Private - Skip" are always skipped. It never deletes, merges, sends email or starts campaigns.

Other prompts: "give me my daily brief", "prep me for my call with <company>", "find warm intros to accounts tagged <tag>", "follow up on <event>", "set up an outbound campaign for A-grade accounts". No skill deletes or merges records, and nothing reaches a prospect (replies, sends, campaign starts) without your confirmation in the chat.

## Repository layout

```
.claude-plugin/marketplace.json       marketplace catalog
plugins/<plugin>/.claude-plugin/plugin.json
plugins/<plugin>/skills/<skill>/SKILL.md
```

## Adding a skill

1. Create `plugins/<plugin>/skills/<skill-name>/SKILL.md` with `name` and `description` frontmatter.
2. Bump `version` in that plugin's `plugin.json` and its entry in `marketplace.json`.
3. Open a pull request.

## License

Apache 2.0
