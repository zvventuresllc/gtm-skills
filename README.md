# gtm-skills

GTM system skills for Claude, packaged as a plugin marketplace.

## Plugins

| Plugin | Skills | Requires |
|---|---|---|
| `hivekind` | `meeting-sweep-to-hivekind`: sweeps today's Granola or Gemini (Google Meet) notes into Hivekind contacts, accounts, debrief notes and tasks | Hivekind connector, plus Granola or Google Drive and Google Calendar |

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
