Start a weekly planning session.

First, check the work-planning memory context for any existing plan state using aim_search_nodes.

Then gather context:
1. Use Gmail MCP to search for emails from "notifications@meetambient.com" from the last 7 days - extract meeting summaries and action items
2. Use Google Calendar MCP to read events from the "Working Time" calendar for the past week (to see what got done and learn from actual time spent)
3. Use Google Calendar MCP to read ALL calendars for the upcoming week to see existing meetings and commitments

Report what you found:
- List the Ambient action items from this week's meetings
- Show the upcoming week's meeting schedule
- Note any patterns from last week (tasks that took longer/shorter than expected)

Then ask the user for:
1. Their Slack saved messages (tasks from messages)
2. Anything else on their mind that needs to get done
3. Any deadlines or priorities to be aware of

After gathering all tasks:
- Help estimate time for each task (check work-planning memory for patterns on similar task types)
- Prioritize with the user (urgent vs. can wait)
- Find available slots around their meetings
- Create time blocks on the "Working Time" calendar
- Save the plan to work-planning memory context (create entities for each task with estimates)

Remember: The periwinkle "Working Time" calendar is for work blocks. Use clear names like "Havas prep", "Follow up: [meeting name]", etc.
