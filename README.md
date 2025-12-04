# AI Work Planning Assistant

Use Claude Code to plan your work week through conversation. Integrates with Google Calendar and Gmail to pull context, schedule work blocks, and learn your patterns over time.

## What It Does

- Reads your meeting summaries from Gmail (works with Ambient, Otter, Fireflies, etc.)
- Checks your calendar for existing meetings
- Helps you estimate time for tasks
- Creates work blocks directly on your calendar
- Remembers your patterns across sessions

## Commands

**Weekly Planning** - Run Friday or Sunday (~15 min)
```
/weekly-planning
```
Reviews the past week, pulls action items, and plans the week ahead.

**Daily Update** - Run each morning (~5 min)
```
/daily-update
```
Quick check-in to adjust for what changed.

## Setup

### 1. Google Cloud Credentials

1. Go to [console.cloud.google.com](https://console.cloud.google.com/)
2. Create a project
3. Enable **Google Calendar API** and **Gmail API**
4. Create OAuth credentials (Desktop app)
5. Download the JSON file

### 2. Install MCP Servers

```bash
# Calendar
claude mcp add-json "google-calendar" '{"command":"npx","args":["-y","@cocal/google-calendar-mcp"],"env":{"GOOGLE_OAUTH_CREDENTIALS":"/path/to/your/credentials.json"}}'

# Gmail
claude mcp add-json "gmail" '{"command":"npx","args":["-y","@gongrzhe/server-gmail-autoauth-mcp"],"env":{"GMAIL_OAUTH_PATH":"/path/to/your/credentials.json"}}'
```

### 3. Authenticate

```bash
GOOGLE_OAUTH_CREDENTIALS="/path/to/your/credentials.json" npx -y @cocal/google-calendar-mcp auth

GMAIL_OAUTH_PATH="/path/to/your/credentials.json" npx -y @gongrzhe/server-gmail-autoauth-mcp auth
```

### 4. Add Slash Commands

Copy `commands/weekly-planning.md` and `commands/daily-update.md` to `~/.claude/commands/`

### 5. Initialize Memory

Ask Claude:
```
Create a work-planning memory context with my planning preferences
```

## Customize

- **Calendar name**: Update the slash commands with your work calendar name
- **Email source**: Change the email search to match your meeting notes tool
- **Task categories**: Modify to match your work types

## Files

- `commands/` - Slash commands to copy to `~/.claude/commands/`
- `docs/plans/` - Design docs and implementation details
- `work-planning-instructions.pdf` - Quick reference guide

---

*Built with [Claude Code](https://claude.com/code)*
