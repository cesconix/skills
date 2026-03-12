---
description: Verify the delivering-tickets environment is ready — check MCP tools, plugins, local config, and cloned repos
---

# Environment Setup

Verify everything is in place for the delivering-tickets to work on a project.

## Steps

1. **Ask which project** if not specified, then load its project from `~/.config/delivering-tickets/projects/`

2. **Check local config** — does `~/.config/delivering-tickets/config.yml` exist?
   - Yes → read `projects` path and `repositories` map
   - No → ask the user to configure it, then create the file

3. **Check repos** — for each repository in the project:
   - Look up its `name` in the `repositories` map from `config.yml`
   - If mapped and path exists → confirm with a checkmark
   - If mapped but path doesn't exist → offer to clone from `repo` URL
   - If not mapped → ask the user for the local path and add it to `config.yml`

4. **Check plugins** — for each entry in `setup.required_plugins`:
   - Verify it's installed
   - If missing → show the install command

5. **Check MCP tools** — for each entry in `setup.required_mcp`:
   - Try to use or list the MCP tool
   - If missing → explain what it's for and how to configure it

6. **Summary** — show a checklist:

```
Environment check for project-alpha:

[x] Local config (~/.config/delivering-tickets/config.yml)
[x] Repo: alpha-frontend (~/workspace/alpha-frontend)
[x] Repo: alpha-api (~/workspace/alpha-api)
[ ] Repo: alpha-libs — not cloned. Run: git clone git@github.com:company/alpha-libs.git ~/workspace/alpha-libs
[x] Plugin: superpowers
[x] MCP: jira
[ ] MCP: slack — not configured. This is needed for team notifications.

1 repo to clone, 1 MCP tool to configure. Want me to help set these up?
```
