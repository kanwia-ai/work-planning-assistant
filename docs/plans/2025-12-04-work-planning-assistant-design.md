# Work Planning Assistant - Design Document

## Overview

A system that helps Kyra plan her work week by integrating with her existing tools (Ambient, Google Calendar, Slack) and learning from her patterns over time.

## Problem Statement

**Pain points:**
1. **Task capture** - Tasks scattered across Ambient (meetings), Slack (messages), and her head
2. **Time estimation** - Hard to know how long tasks will take when blocking calendar time

**Current state:**
- Uses Google Calendar with time-blocking (periwinkle "Working Time" calendar for work blocks, magenta for meetings)
- Weekly planning + daily adjustments
- Manual copy/paste from Ambient
- No system for capturing Slack tasks

## System Architecture

### Integrations

| Component | Purpose | Access |
|-----------|---------|--------|
| Gmail MCP | Read Ambient email summaries (contain meeting context + action items) | Read |
| Google Calendar MCP | Read all calendars, create blocks on "Working Time" | Read/Write |
| Memory MCP | Store task history, patterns, estimates in `work-planning` context | Read/Write |

### Manual Inputs

- Slack saved messages (dumped at planning time)
- Tasks from Kyra's head
- Feedback on estimates (actual vs. planned time)

### Outputs

- Time blocks created directly on "Working Time" calendar
- Improving time estimates based on historical patterns
- Eventually: proactive schedule proposals

## Workflows

### Weekly Planning (End of Week)

**When:** Friday afternoon or Sunday evening

**Flow:**
1. Kyra starts conversation: "let's do weekly planning"
2. Claude pulls context automatically:
   - Ambient email summaries from past week (via Gmail)
   - Past week's "Working Time" blocks (to learn what got done)
   - Upcoming week's calendar (existing meetings)
3. Kyra provides:
   - Slack saved messages dump
   - Anything in her head
4. Together:
   - Review action items from Ambient
   - Estimate time for each task (Claude suggests based on patterns)
   - Identify urgent vs. can wait
   - Find available slots around meetings
5. Claude creates time blocks on "Working Time" calendar
6. Claude saves to memory: tasks planned, estimates, context

### Daily Update

**When:** Morning, or when things shift

**Flow:**
1. Kyra pings: "daily update" or "things changed"
2. Claude pulls:
   - Today's calendar
   - Any new Ambient emails since last session
   - What was planned from memory
3. Kyra shares what's new:
   - Tasks that took longer/shorter than expected
   - New urgent items
   - Things that need to move
4. Claude adjusts:
   - Moves/creates calendar blocks
   - Re-prioritizes if needed
   - Flags if something won't fit
5. Claude updates memory: actual time vs. estimated (feeds learning loop)

## Learning System

### Phase 1 (Weeks 1-2): Baseline Building
- Claude asks for estimates on each task
- Categorizes tasks by type
- Stores every estimate + actual time spent

### Phase 2 (Weeks 3-4): Pattern Recognition
- Claude suggests estimates based on task type
- "This looks like client prep - those typically take you 2-3 hours"
- User confirms or adjusts, Claude learns from corrections

### Phase 3 (Month 2+): Confident Defaults
- Claude proposes full schedules with estimates baked in
- Flags outliers: "This seems bigger than typical - want more time?"
- Knows buffer preferences

### Task Categories (Initial)
- Client prep/delivery work (Havas, etc.)
- Meeting follow-ups / action items
- 1:1 prep
- Deep focus work
- Admin / quick tasks
- Interview-related

### Proactive Mode (Goal State)
- Claude sees new Ambient action items via email
- Knows calendar and patterns
- Drafts proposed schedule: "Here's how I'd slot this week - want to review?"

## Continuity via Memory MCP

**Context:** `work-planning`

**What gets stored:**
- Entities: Tasks (with type, status, related project)
- Observations: Estimates, scheduled times, actual completion times, notes
- Relations: Task relationships, project groupings

**Example:**
```
Entity: Havas_Final_Prep (type: task)
Observations:
  - "Estimated 2.5 hours"
  - "Scheduled Tuesday 7:15-9:45pm"
  - "Completed - took 2.5 hours"
Relations:
  - related_to → Havas_Client_Work
```

**Session flow:**
- New session starts → Claude reads `work-planning` context
- Work happens → Claude updates entities/observations
- Session ends → State persists for next session

## New Habits Required

1. **Save Slack messages** when they contain tasks (using Slack's Save feature)
2. **Weekly planning session** - Friday or Sunday
3. **Daily check-in** - morning or as needed

## Setup Requirements

1. **Google Calendar MCP**
   - OAuth authentication with Google Workspace
   - Permissions: read events, create events
   - Target calendar: "Working Time"

2. **Gmail MCP**
   - Same Google account
   - Permissions: read emails
   - Filter to Ambient sender for efficiency

3. **Memory MCP** (already installed)
   - Initialize `work-planning` context
   - No additional setup needed

**Estimated setup time:** 30-45 minutes
