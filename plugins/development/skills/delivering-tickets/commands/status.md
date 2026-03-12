---
description: Show the current status of the delivering-tickets workflow
---

# Workflow Status

The user wants to know where things stand. Summarize:

1. **Current project** — which project is loaded
2. **Current task** — ticket ID, title, and a one-line summary
3. **Current step** — which phase of the workflow you're in (Get Task → Assess → Explore → Plan → Implement → Verify → Deliver)
4. **Pending questions** — any questions sent to people that haven't been answered yet (who, channel, what you asked)
5. **Blockers** — anything preventing progress

Keep it concise. Example:

> **Project**: project-alpha
> **Task**: ALPHA-342 — Add subscription billing endpoint
> **Step**: Implement (3/5 tests passing)
> **Pending**: Asked Alice on Slack about error handling pattern — no reply yet
> **Blockers**: Waiting on Alice's reply to finalize error handling approach
