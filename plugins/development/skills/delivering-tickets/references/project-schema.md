# Project Schema

This is the full schema for `~/.config/delivering-tickets/projects/{project-name}.yml`. All sections are optional except `project` and `repositories`.

```yaml
# ============================================================
# Project Identity
# ============================================================
project: "project-alpha"                    # Unique project name

# ============================================================
# Repositories
# ============================================================
repositories:
  - name: alpha-frontend                    # Used to resolve local path: {workspace}/{name}
    repo: git@github.com:company/alpha-frontend.git
    base_branch: develop                    # Default branch for PRs
  - name: alpha-api
    repo: git@github.com:company/alpha-api.git
    base_branch: main

# ============================================================
# Board / Task Management
# ============================================================
board:
  tool: jira                                # jira | clickup | github
  project_key: "ALPHA"                      # Project identifier on the board
  statuses:                                 # Optional: map workflow statuses
    todo: "To Do"
    in_progress: "In Progress"
    in_review: "In Review"
    done: "Done"

# ============================================================
# Team Contacts
# ============================================================
# Roles determine communication style:
#   Technical (code, architecture, trade-offs): tech_lead, developer, devops
#   Non-technical (impact, scope, decisions):   pm_business, stakeholder, product_owner
#   Hybrid (both styles OK):                    pm_technical
contacts:
  - name: "Alice Rossi"
    role: tech_lead                         # tech_lead | developer | devops | pm_technical | pm_business | stakeholder | product_owner
    channel: slack                          # slack | teams | prompt
    handle: "@alice.rossi"
    ask_about:                              # Topics this person can help with
      - architecture decisions
      - code patterns
      - performance concerns

  - name: "Marco Bianchi"
    role: pm_business
    channel: teams
    handle: "@marco.bianchi"
    ask_about:
      - requirements clarification
      - priority and scope
      - stakeholder impact

# ============================================================
# Notifications
# ============================================================
notifications:
  tool: slack                               # slack | teams
  channel: "#alpha-dev"                     # Where to post updates (PR opened, task done, etc.)

# ============================================================
# Documentation
# ============================================================
# List anything useful: files, folders, URLs. No need for topic tagging.
# The agent explores what it needs based on the task.
# If there's no documentation, that's fine — the codebase and contacts are the docs.
documentation:
  sources:
    - ./docs/
    - ./README.md
    - https://confluence.company.com/wiki/alpha

# ============================================================
# Conventions
# ============================================================
conventions:
  branching: "feat/{ticket-id}-{short-desc}" # Branch naming pattern
  commit_style: conventional                 # conventional | freeform
  pr_reviewers:                              # Default PR reviewers
    - alice.rossi
    - luca.verdi

# ============================================================
# Testing
# ============================================================
testing:
  commands:
    - npm test
    - npm run lint
    - npm run typecheck
  integration:
    enabled: true                             # Enable integration/smoke testing on running environment
    # No commands or ports needed — the agent figures out what to test
    # based on the changes made, and asks the dev to start services
    # from the worktree path

# ============================================================
# Tribal Knowledge
# ============================================================
# Hard-won lessons. Every item here exists because someone got burned.
tribal_knowledge:
  - "Never modify the legacy auth module directly — use the adapter in src/adapters/auth.ts"
  - "The payment service has a 30s timeout in staging — mock it in tests"
  - "Always run migrations before starting the API locally: npm run db:migrate"

# ============================================================
# MCP Tools
# ============================================================
# Which MCP tools the team has configured. The agent uses these to interact
# with external services. If a tool is missing, the agent will flag it.
mcp_tools:
  board: jira                               # MCP tool for task management
  notifications: slack                      # MCP tool for team notifications
  code: github                              # MCP tool for PRs (github | gitlab)
  docs: confluence                          # MCP tool for external docs (optional)

# ============================================================
# Setup Requirements
# ============================================================
# What a new developer needs to install to use delivering-tickets on this project.
setup:
  required_plugins:
    - name: superpowers
      install: "/plugin install superpowers@claude-plugins-official"
  required_mcp:
    - name: jira
      purpose: "Board management — fetch and update tasks"
    - name: slack
      purpose: "Team notifications and async communication"
    - name: github
      purpose: "PR creation and code review"
```

## Developer-Local Config

Each developer has their own `~/.config/delivering-tickets/config.yml` (NOT versioned, NOT shared):

```yaml
# Where to find project files (.yml files)
# Can be a personal directory or a cloned shared repo
projects: ~/.config/delivering-tickets/projects

# Mapping: repository name → local path on this machine
# Each key must match a repository `name` from a project file
repositories:
  alpha-frontend: ~/workspace/alpha-frontend
  alpha-api: ~/workspace/alpha-api
```

- `projects` — directory containing `.yml` project files. Can point to a personal directory or a cloned shared Git repo for team-wide sharing.
- `repositories` — maps each repository `name` (as defined in project files) to its local path on this machine. The agent uses this to find where repos are cloned.
