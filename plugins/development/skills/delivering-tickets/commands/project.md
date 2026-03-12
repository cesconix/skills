---
description: Create or edit a delivering-tickets project interactively
---

# Project Manager

Help the user create or edit a project. Walk them through it conversationally — don't dump the full schema at them.

## Creating a New Project

Ask these questions in order, adapting based on answers:

### 1. Project basics
- What's the project name?
- How many repos does it have? (names and git URLs)
- What's the base branch for each? (main, develop, etc.)

### 2. Board
- What board tool do you use? (Jira, ClickUp, GitHub Issues)
- What's the project key or identifier?

### 3. Team & contacts
For each person:
- Name and role: `tech_lead`, `developer`, `devops`, `pm_technical`, `pm_business`, `stakeholder`, `product_owner`
- How to reach them: which tool (slack, teams) and their handle
- What topics to ask them about

### 4. Communication
- What tool does the team use for notifications? (Slack, Teams)
- Which channel for project updates?

### 5. Conventions
- Branching pattern? (e.g., `feat/{ticket-id}-{short-desc}`)
- PR reviewers?
- Test commands?
- Linting/type-check commands?

### 6. Documentation
- Any docs the agent should know about? Files, folders, URLs — anything goes.
- Don't worry if there aren't many. The codebase and the team are valid sources too.

### 7. Tribal knowledge
- Any gotchas, pitfalls, or "things everyone on the team knows but nobody wrote down"?
- Patterns to follow or avoid?
- Services with quirks?

### 8. MCP tools
- Which MCP tools does the team have configured? (jira, clickup, slack, teams, github, gitlab, confluence, notion...)

### 9. Setup requirements
- Any plugins the team needs? (e.g., superpowers)
- Any MCP tools that must be configured?

## Output

Save the project to `~/.config/delivering-tickets/projects/{project-name}.yml`. Use the schema from `references/project-schema.md`.

After saving, suggest running `/delivering-tickets:setup` to verify the environment.

## Editing an Existing Project

If the user wants to modify an existing project:
1. Read the current project file
2. Ask what they want to change
3. Update only the relevant sections
4. Show a summary of changes before saving
