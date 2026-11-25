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

In the Claude app, add this repository as a plugin marketplace from the plugin settings, then install `hivekind`.

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
