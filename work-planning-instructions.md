# Work Planning Assistant

## Quick Start

### Weekly Planning (Friday or Sunday)
Open Claude Code and type:
```
/weekly-planning
```

**What happens:**
1. I pull your Ambient meeting summaries from the past week (via Gmail)
2. I check your calendar for next week's meetings
3. I look at last week's work blocks to learn your patterns
4. You paste your Slack saved messages
5. We estimate time and schedule work blocks together
6. I create blocks on your "Working Time" calendar

**Time:** ~15-20 minutes

---

### Daily Update (Each Morning)
Open Claude Code and type:
```
/daily-update
```

**What happens:**
1. I check today's calendar
2. I look for new Ambient action items
3. You tell me what changed
4. I adjust your calendar blocks

**Time:** 2-5 minutes

---

## New Habits to Build

### 1. Save Slack Messages
When someone sends you a task in Slack:
- Click the bookmark icon (or right-click → Save)
- At planning time, open Saved items and dump them to me

### 2. Weekly Planning Ritual
Pick a consistent time:
- Friday afternoon (plan before the weekend), OR
- Sunday evening (plan for the week ahead)

### 3. Morning Check-in
Before diving into work:
- Run `/daily-update`
- Tell me what shifted
- Start your day with a clear plan

---

## How the Learning Works

**Weeks 1-2:** You tell me estimates, I track actual time

**Weeks 3-4:** I start suggesting estimates based on task type
- "This looks like client prep - those typically take you 2-3 hours"

**Month 2+:** I propose schedules proactively
- "Here's how I'd slot this week - want to review?"

---

## Task Categories

| Category | Description | Examples |
|----------|-------------|----------|
| client_prep | Client work | Havas prep, deliverables |
| meeting_followup | Action items from meetings | Follow-ups, sends |
| one_on_one_prep | 1:1 preparation | Notes, agenda prep |
| deep_focus | Long concentrated work | Strategy, writing |
| admin | Quick tasks | Emails, scheduling |
| interviews | Interview activities | Prep, debrief |

---

## Your Calendar Colors

- **Periwinkle (Working Time):** Work blocks I create for you
- **Magenta/Purple:** Your meetings

---

## Quick Commands

| What you want | What to say |
|---------------|-------------|
| Weekly planning | `/weekly-planning` |
| Daily update | `/daily-update` |
| Add a task | "Add task: [description] - estimate [X hours]" |
| Check the plan | "What's on my plan for this week?" |

---

## Troubleshooting

**MCP servers not working?**
- Restart Claude Code (exit and reopen)
- Run `claude mcp list` to check status

**Calendar events not showing?**
- Make sure you're looking at the right calendar ("Working Time")
- Re-authenticate: `GOOGLE_OAUTH_CREDENTIALS=~/.claude/gcp-oauth.keys.json npx -y @cocal/google-calendar-mcp auth`

**Gmail not finding Ambient emails?**
- Check that Ambient sends to your Google Workspace email
- Emails come from: notifications@meetambient.com

---

## Technical Details

**MCP Servers Installed:**
- `google-calendar` - Reads/writes your calendars
- `gmail` - Reads your Ambient email summaries
- `memory` - Stores your task history and patterns

**Memory Context:** `work-planning`

**Files Created:**
- `~/.claude/commands/weekly-planning.md`
- `~/.claude/commands/daily-update.md`
- `~/.claude/prompts/work-planning-quickref.md`

---

*System designed December 2025*
