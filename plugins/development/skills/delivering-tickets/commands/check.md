---
description: Check for replies on communication channels (Slack, Teams) from people the agent previously contacted during a task
---

# Check Replies

The user wants to check if anyone has replied to questions you asked earlier.

## What to do

1. **Load the project file** from `projects/` to know which MCP tools are available for communication
2. **Identify pending questions** — recall what you asked, to whom, and on which channel
3. **Read the relevant threads/channels** using the appropriate MCP tool (Slack, Teams, etc.)
4. **Report back**:
   - If there's a reply → summarize it and ask if you should proceed based on it
   - If no reply yet → say so, suggest the user can check again later or provide the answer directly
   - If multiple pending questions → check all of them and report each result

## Example output

> **Alice (@alice.rossi on Slack)** — asked about error handling pattern
> Reply found: "Use the Result type pattern, we're migrating everything to that."
>
> **Marco (@marco.bianchi on Teams)** — asked about billing scope
> No reply yet.
>
> Want me to proceed with Alice's input? For Marco's question I'll wait — let me know when to check again.
