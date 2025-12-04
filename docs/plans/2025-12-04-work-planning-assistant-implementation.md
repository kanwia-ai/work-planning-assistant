# Work Planning Assistant - Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Set up MCP integrations (Google Calendar + Gmail) and initialize the work-planning memory context so Claude can help Kyra plan her work week.

**Architecture:** Gmail MCP reads Ambient email summaries, Google Calendar MCP reads/writes to "Working Time" calendar, Memory MCP stores task history and patterns in `work-planning` context.

**Tech Stack:** MCP servers (npx), Google Cloud OAuth, Claude Code CLI

---

## Task 1: Set Up Google Cloud Project

**Purpose:** Create OAuth credentials that both Gmail and Calendar MCP servers will use.

**Step 1: Create Google Cloud Project**

1. Open browser to https://console.cloud.google.com/
2. Click project dropdown (top left) → "New Project"
3. Name: `claude-work-planning`
4. Click "Create"
5. Wait for project creation, then select it

**Step 2: Enable APIs**

1. Go to "APIs & Services" → "Library"
2. Search "Google Calendar API" → Click it → Click "Enable"
3. Search "Gmail API" → Click it → Click "Enable"

**Step 3: Configure OAuth Consent Screen**

1. Go to "APIs & Services" → "OAuth consent screen"
2. Select "External" user type → Click "Create"
3. Fill in:
   - App name: `Claude Work Planning`
   - User support email: Your email
   - Developer contact: Your email
4. Click "Save and Continue"
5. On Scopes page, click "Add or Remove Scopes"
6. Add these scopes:
   - `https://www.googleapis.com/auth/calendar`
   - `https://www.googleapis.com/auth/calendar.events`
   - `https://www.googleapis.com/auth/gmail.readonly`
7. Click "Update" → "Save and Continue"
8. On Test Users page, click "Add Users"
9. Add your Google Workspace email
10. Click "Save and Continue" → "Back to Dashboard"

**Step 4: Create OAuth Credentials**

1. Go to "APIs & Services" → "Credentials"
2. Click "Create Credentials" → "OAuth client ID"
3. Application type: "Desktop app"
4. Name: `claude-mcp-client`
5. Click "Create"
6. Click "Download JSON"
7. Save file to `~/.claude/gcp-oauth.keys.json`

**Verification:**

Run:
```bash
ls -la ~/.claude/gcp-oauth.keys.json
```

Expected: File exists with ~400-500 bytes

---

## Task 2: Install Google Calendar MCP Server

**Files:**
- Modify: Claude Code MCP configuration

**Step 1: Add Calendar MCP to Claude Code**

Run:
```bash
claude mcp add google-calendar --command "npx" -- -y @cocal/google-calendar-mcp
```

**Step 2: Configure OAuth credentials path**

Run:
```bash
claude mcp add-json "google-calendar" '{"command":"npx","args":["-y","@cocal/google-calendar-mcp"],"env":{"GOOGLE_OAUTH_CREDENTIALS":"/path/to/your"}}'
```

**Step 3: Authenticate with Google**

Run:
```bash
npx -y @cocal/google-calendar-mcp auth
```

A browser window will open. Sign in with your Google Workspace account and grant calendar permissions.

**Step 4: Verify installation**

Run:
```bash
claude mcp list
```

Expected: `google-calendar` appears in the list with status "Connected"

---

## Task 3: Install Gmail MCP Server

**Files:**
- Modify: Claude Code MCP configuration

**Step 1: Create Gmail MCP config directory**

Run:
```bash
mkdir -p ~/.gmail-mcp
cp ~/.claude/gcp-oauth.keys.json ~/.gmail-mcp/gcp-oauth.keys.json
```

**Step 2: Add Gmail MCP to Claude Code**

Run:
```bash
claude mcp add-json "gmail" '{"command":"npx","args":["-y","@gongrzhe/server-gmail-autoauth-mcp"],"env":{"GMAIL_OAUTH_PATH":"/path/to/your"}}'
```

**Step 3: Authenticate with Google**

Run:
```bash
npx -y @gongrzhe/server-gmail-autoauth-mcp auth
```

Browser opens for Google sign-in. Grant Gmail read permissions.

**Step 4: Verify installation**

Run:
```bash
claude mcp list
```

Expected: `gmail` appears in the list with status "Connected"

---

## Task 4: Initialize Work-Planning Memory Context

**Purpose:** Create the `work-planning` context in Memory MCP and seed it with initial structure.

**Step 1: Create initial entities**

In a new Claude Code session, run this command to Claude:

```
Create these entities in the work-planning memory context:

1. Entity: "Planning_System" (type: system)
   Observations:
   - "Weekly planning happens Friday afternoon or Sunday evening"
   - "Daily updates happen each morning or when things change"
   - "Working Time calendar (periwinkle) is for work blocks"
   - "Ambient email summaries contain meeting action items"

2. Entity: "Task_Categories" (type: reference)
   Observations:
   - "client_prep: Client preparation and delivery work (e.g., Havas)"
   - "meeting_followup: Action items from meetings"
   - "one_on_one_prep: Preparation for 1:1 meetings"
   - "deep_focus: Concentrated work requiring long blocks"
   - "admin: Quick administrative tasks"
   - "interviews: Interview-related activities"

3. Entity: "Time_Patterns" (type: learning)
   Observations:
   - "Initial estimates are user-provided, will learn over time"
   - "Track: estimated_time, actual_time, task_type for each task"
```

**Step 2: Verify context was created**

Ask Claude:
```
Search the work-planning memory context for "Planning_System"
```

Expected: Returns the Planning_System entity with its observations

---

## Task 5: Create Planning Session Prompts

**Purpose:** Create reusable prompts for weekly and daily planning sessions.

**Step 1: Create weekly planning prompt file**

Create file: `~/.claude/prompts/weekly-planning.md`

```markdown
# Weekly Planning Session

## What I'll do:
1. Read your Ambient email summaries from this past week (via Gmail)
2. Look at your "Working Time" calendar from last week to see what got done
3. Check your calendar for next week's meetings
4. Help you plan work blocks around those meetings

## What I need from you:
1. Dump your Slack saved messages (tasks from messages)
2. Tell me anything else on your mind that needs to get done
3. Let me know any deadlines or priorities

## Let's start:
First, let me pull your Ambient emails and calendar...
```

**Step 2: Create daily update prompt file**

Create file: `~/.claude/prompts/daily-update.md`

```markdown
# Daily Update

## Quick check:
1. I'll look at today's calendar
2. Check for new Ambient emails since we last talked
3. Review what was planned from memory

## Tell me:
- What changed since yesterday?
- Anything take longer/shorter than expected?
- New urgent items?

## Then I'll:
- Adjust your calendar blocks
- Update the plan
- Flag anything that won't fit
```

**Step 3: Create slash command for weekly planning**

Create file: `~/.claude/commands/weekly-planning.md`

```markdown
Start a weekly planning session.

First, check the work-planning memory context for any existing plan state.

Then:
1. Use Gmail MCP to search for emails from "notifications@meetambient.com" from the last 7 days
2. Use Google Calendar MCP to read events from the past week on all calendars
3. Use Google Calendar MCP to read events for the upcoming week
4. Report what you found and ask the user for their Slack saved messages and any other tasks

After gathering all tasks:
- Help estimate time for each task (check work-planning memory for patterns)
- Prioritize with the user
- Create time blocks on the "Working Time" calendar
- Save the plan to work-planning memory context
```

**Step 4: Create slash command for daily update**

Create file: `~/.claude/commands/daily-update.md`

```markdown
Start a daily planning update.

1. Read current plan from work-planning memory context
2. Check Google Calendar for today's schedule
3. Check Gmail for any new Ambient emails since yesterday
4. Ask user what changed

Then:
- Adjust calendar blocks as needed
- Update work-planning memory with any changes
- Note actual vs. estimated time for completed tasks (for learning)
```

**Verification:**

Run:
```bash
ls -la ~/.claude/prompts/
ls -la ~/.claude/commands/
```

Expected: Both directories contain the new files

---

## Task 6: Test the Full System

**Step 1: Start new Claude Code session**

Run:
```bash
claude
```

**Step 2: Verify all MCP servers are connected**

Run:
```bash
/mcp
```

Expected: google-calendar, gmail, memory all show "Connected"

**Step 3: Test Gmail access**

Ask Claude:
```
Use Gmail MCP to search for emails from "notifications@meetambient.com" from the last 7 days. Just tell me how many you find and the subjects.
```

Expected: Returns list of Ambient email summaries

**Step 4: Test Calendar read access**

Ask Claude:
```
Use Google Calendar MCP to list all my calendars, then show me events from "Working Time" calendar for this week.
```

Expected: Shows your calendars including "Working Time" and lists events

**Step 5: Test Calendar write access**

Ask Claude:
```
Create a test event on my "Working Time" calendar for tomorrow at 9am-9:30am called "TEST - Delete Me"
```

Expected: Event is created. Go verify in Google Calendar, then delete it.

**Step 6: Test Memory context**

Ask Claude:
```
Search the work-planning memory context for "Planning_System" and show me what's stored.
```

Expected: Returns the Planning_System entity with observations

**Step 7: Run first weekly planning session**

Run the slash command:
```
/weekly-planning
```

Follow the prompts to complete your first weekly planning session.

---

## Task 7: Document the Workflow

**Step 1: Create quick reference**

Create file: `~/.claude/prompts/work-planning-quickref.md`

```markdown
# Work Planning Quick Reference

## Weekly Planning (Friday/Sunday)
```
/weekly-planning
```
- Have Slack saved messages ready to paste
- ~15-20 min session

## Daily Update (Morning)
```
/daily-update
```
- Quick 2-5 min check-in
- Tell me what changed

## Ad-hoc Task Capture
Just tell me:
"Add task: [description] - estimate [X hours] - due [date]"
I'll add it to memory and suggest when to schedule it.

## Check Current Plan
"What's on my plan for this week?"
I'll read from work-planning memory and show you.
```

---

## Summary: What You Now Have

| Component | Status | Purpose |
|-----------|--------|---------|
| Google Calendar MCP | Installed | Read/write "Working Time" calendar |
| Gmail MCP | Installed | Read Ambient email summaries |
| Memory MCP | Configured | `work-planning` context for continuity |
| `/weekly-planning` | Ready | Start weekly planning session |
| `/daily-update` | Ready | Quick daily adjustments |

**Your new workflow:**
1. Friday/Sunday: `/weekly-planning`
2. Each morning: `/daily-update`
3. System learns your patterns over time
4. Eventually becomes proactive with schedule suggestions
