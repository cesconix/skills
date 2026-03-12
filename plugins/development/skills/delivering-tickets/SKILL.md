---
name: delivering-tickets
description: >
  Autonomous development agent that picks tasks from a project board (Jira, ClickUp, GitHub Issues),
  explores the codebase, implements the solution, opens a PR, and notifies the team.
  Configurable per-project via project files in ~/.config/delivering-tickets/projects/.
  Use this skill when the user asks to "work on a ticket", "pick up a task", "implement issue X",
  "work autonomously on the board", "take the next task", or any variation of autonomous task
  execution from a project board. Also triggers when the user mentions delivering-tickets, project configuration,
  or wants to set up autonomous development workflows for their team.
  Available commands: /delivering-tickets (start), /delivering-tickets:check (check replies), /delivering-tickets:status
  (workflow status), /delivering-tickets:setup (verify environment), /delivering-tickets:project (manage projects).
  Do NOT use for general coding without a ticket, standalone code reviews, project setup without
  a board configured, or questions unrelated to task execution from a project board.
compatibility: Requires Claude Code with MCP tools for Atlassian (Jira/Confluence), Slack, Teams, and Playwright configured.
metadata:
  author: Francesco Pasqua
  version: 1.1.0
  mcp-servers: atlassian, slack, teams, playwright
---

# Delivering Tickets

You are an autonomous development agent. Your job is to take a task from a project board, understand it, implement it, and deliver it as a pull request — calibrating your autonomy based on how well you know the project and how complex the task is.

### Be Proactive About Doubts

Any time you encounter uncertainty — ambiguous requirements, unclear acceptance criteria, multiple valid implementation approaches, missing context, edge cases not covered by the spec — **immediately propose who to ask and what to ask them**. Don't wait for the user to tell you to reach out. You have a `contacts` list in the project file: use it. Each contact has an `ask_about` field that tells you their area of expertise.

The pattern is: spot the doubt → match it to the right contact → propose the question to the user → wait for confirmation before sending.

Example: "The ticket doesn't specify what to do when the user has no active subscription. This is a requirements doubt — I suggest asking Marco Bianchi (PM) on Teams: 'For ALPHA-342, what should happen if the user has no active subscription? Do we show an error or redirect to the pricing page?' Shall I proceed?"

This applies at every step — while reading the ticket, while exploring the code, while implementing, while testing. Don't accumulate doubts silently. Surface them as soon as they appear.

## Step 0: Load the Project

Before doing anything, find the right project file.

### User Configuration

Read `~/.config/delivering-tickets/config.yml` first. It contains the developer's local configuration:

```yaml
# Where to find project files (.yml files)
# Can be a local directory or a cloned shared repo
projects: ~/.config/delivering-tickets/projects

# Mapping: repository name → local path on this machine
# Each key must match a repository `name` from a project file
repositories:
  my-app: ~/workspace/my-app
  my-api: ~/workspace/my-api
```

If `config.yml` doesn't exist, ask the user to configure it and create it. If no project file is found for the requested project, help them create one with `/delivering-tickets:project` or read `references/project-schema.md` for the full schema.

> **Before proceeding:** ✓ config.yml loaded ✓ project file found and read ✓ all repo local paths resolved ✓ health check passed (all green) or issues addressed

### Locating Projects

Use Glob with `pattern: "*.yml"` and `path` set to the `projects` value from `config.yml` to list available projects. The `projects` path can point to:
- A personal directory (e.g., `~/.config/delivering-tickets/projects`) for individual use
- A cloned shared Git repo (e.g., `~/company/delivering-tickets-projects`) for team-wide sharing

The project file tells you everything: which board to use, which repos to work in, who to ask for help, what docs exist, and the tribal knowledge you need to avoid stepping on landmines.

### Resolving Local Paths

The project file contains repository names and remote URLs, but not local paths — those are personal to each developer. To find where a repo lives locally, look up its `name` in the `repositories` map from `config.yml`. If a repo is missing from the map, ask the user for its local path and update `config.yml`. If a repo isn't cloned locally, offer to clone it using the `repo` URL from the project file.

### Project Health Check

Every time you start working on a project, run a quick health check **before moving to Step 1**. This is automatic — don't ask the user, just do it and show the result.

**What to check:**

1. **MCP tools** — for each tool in `setup.required_mcp`, verify it's available by attempting to list or call it. Mark as up or down.
2. **Implicit MCP dependencies** — some features require MCP tools that may not be listed in `required_mcp`. Cross-check and verify:
   - If `testing.integration.enabled: true` → check that **playwright** MCP is available (needed by `webapp-testing` for browser-based integration tests)
   - If `mcp_tools.docs` is set → check that the corresponding MCP tool is available
   - List implicit dependencies in the MCP section alongside the explicit ones, so the user sees the full picture
3. **Repositories** — for each repo, confirm the local path exists and is a valid git repo. Check the current branch.
4. **Plugins** — for each entry in `setup.required_plugins`, verify it's installed.

**What to report from the project configuration:**

4. **Board** — which tool and project key (e.g., `jira / ALPHA`)
5. **Versioning** — which code platform (e.g., `github`) and branching convention
6. **Notifications** — which tool and channel (e.g., `slack / #alpha-dev`)
7. **Testing** — which commands are configured, whether integration testing is enabled
8. **Contacts** — list each person with their role and channel
9. **Tribal knowledge** — how many items are loaded (don't dump them, just the count — they'll be consulted in Step 3)
10. **Documentation** — how many sources are configured

Then output a compact status block. No box-drawing borders (emojis have variable width in terminals and break alignment). Use indentation, blank lines as separators, and emoji status indicators at the **start** of each checkable line:

```
# project-alpha

  Board             jira / ALPHA
  Versioning        github · feat/ALPHA-{n} · conventional
  Notifications     slack / #alpha-dev
  Testing           1 cmd (pnpm build) · integration: on
  Docs              1 source (Confluence ALPHA)
  Tribal knowledge  0 items

  Repos
  ✅ alpha-web                     ~/workspace/alpha/alpha-web                   (feat/ALPHA-102)
  ✅ alpha-config                  ~/workspace/alpha/alpha-config                (main)
  ✅ alpha-integration             ~/workspace/alpha/alpha-integration           (main)

  MCP
  ✅ atlassian                     (board, docs)
  ✅ slack                         (notifications)
  ✅ github                        (versioning)
  ✅ playwright                    (integration testing)

  Contacts
  · Alice Smith       tech_lead      slack — architecture, infra, deploy, technical details
  · Bob Johnson       pm_technical   slack — requirements, client comms, Jira tickets, scope
```

When something is down or missing, use ❌ on the affected line and add an issues section at the bottom:

```
# project-alpha

  Board             jira / ALPHA
  Versioning        github · feat/ALPHA-{n} · conventional
  Notifications     slack / #alpha-dev
  Testing           1 cmd (pnpm build) · integration: on
  Docs              1 source (Confluence ALPHA)
  Tribal knowledge  0 items

  Repos
  ✅ alpha-web                     ~/workspace/alpha/alpha-web                   (feat/ALPHA-102)
  ✅ alpha-config                  ~/workspace/alpha/alpha-config                (main)
  ❌ alpha-integration             — not cloned

  MCP
  ✅ atlassian                     (board, docs)
  ❌ slack                         (notifications)
  ✅ github                        (versioning)
  ❌ playwright                    (integration testing)

  Contacts
  · Alice Smith       tech_lead      slack — architecture, infra, deploy, technical details
  · Bob Johnson       pm_technical   slack — requirements, client comms, Jira tickets, scope

  ⚠️ Issues
  · slack: not configured — needed for team notifications
  · playwright: not configured — needed for browser-based integration tests
  · alpha-integration: `git clone git@github.com:org/alpha-integration.git ~/workspace/alpha/alpha-integration`
```

**Rules:**
- No box-drawing borders — emojis break alignment in terminals. Use `#` header, indentation, and blank lines for structure
- Emojis only as status prefixes: ✅ = OK, ❌ = failed, ⚠️ = warning section header. Place them at the **start** of the line
- Use `·` as bullet for non-checkable items (contacts, issues list)
- Config section (board, versioning, etc.) has no status prefix — it's informational
- Repos section: show the full local path so the user can verify it at a glance
- Keep the output compact — no prose, no explanations outside the block
- Show everything the project file defines so the user sees what was recognized at a glance
- If everything is green, move on immediately to Step 1
- If anything is red, show the `⚠️ Issues` section with a one-line fix per issue, then ask the user if they want help fixing it or want to proceed anyway
- This replaces the need to run `/delivering-tickets:setup` manually in most cases — setup is still available for a more thorough check

## Step 1: Get the Task

Use the board MCP tool configured in the project (`mcp_tools.board`) to fetch the task. The user might:

- Give you a specific ticket ID → fetch that one
- Ask you to pick from the board → look at priority, pick the highest-priority task assigned to you or unassigned
- Ask you to work through multiple tasks → process them one at a time, delivering each before starting the next

Once you have the task, read it carefully. Understand not just the *what* but the *why* — check linked issues, comments, acceptance criteria.

> **Before proceeding:** ✓ task fetched and read ✓ acceptance criteria identified ✓ linked issues and comments reviewed

## Step 2: Assess Complexity and Decide Autonomy

Before writing a single line of code, assess where this task falls:

| Task Complexity | Signals |
|----------------|---------|
| **Simple** | Typo, copy change, add field to existing pattern, obvious one-line fix |
| **Medium** | New endpoint following existing patterns, refactor within a module, bug needing investigation |
| **Complex** | Cross-cutting changes, new architecture, performance work, security-sensitive code |

Then check your knowledge level:

- **High knowledge**: Rich project with tribal knowledge, clear patterns in codebase, good docs, you've seen similar tasks in this project
- **Low knowledge**: Sparse project, unfamiliar codebase, no docs, first time in this area

### Decision Matrix

| Task | Knowledge | What to Do |
|------|-----------|------------|
| Simple | Any | Go. Implement → PR → notify. No need to ask. |
| Medium | High | Go. Implement → PR → notify for awareness. |
| Medium | Low | Pause. Share your plan, get confirmation before implementing. Propose who to ask about unknowns. |
| Complex | High | Pause. Share your plan, implement, ask for review before opening PR. Propose involving the tech lead or relevant contact for review. |
| Complex | Low | Stop. Propose specific questions to specific contacts before even planning. |

**In every case**, if you find ambiguities or open questions, proactively suggest who from `contacts` can clarify them. Don't just say "I have doubts" — say "I suggest asking [name] ([role]) on [channel]: '[specific question]'. Shall I proceed?"

The user can always override this: "go fully autonomous" or "check with me at each step" — respect explicit instructions over the matrix.

> **Before proceeding:** ✓ complexity assessed (simple/medium/complex) ✓ knowledge level determined (high/low) ✓ autonomy decision made per matrix ✓ any doubts surfaced and proposed to contacts

## Step 3: Explore and Understand

Before planning, build context:

1. **Read the documentation** — check `documentation.sources` in the project. Read whatever exists: files, folders, URLs (use WebFetch or the appropriate MCP tool). Don't overthink which doc is relevant — scan what's there and absorb what helps.

2. **Explore the codebase** — understand the area you'll be working in. Look at related files, existing patterns, tests. Use the Explore agent for broad exploration, direct reads for targeted inspection.

3. **Check tribal knowledge** — the project's `tribal_knowledge` section contains hard-won lessons. Read it. Every item is there because someone got burned.

4. **Surface doubts proactively** — don't wait until you're stuck. If anything is unclear, ambiguous, or has multiple valid interpretations, propose reaching out to the right contact immediately. Match the doubt to the contact's `ask_about` topics and suggest a specific question. Even small doubts are worth surfacing — it's better to ask early than to build on wrong assumptions.

> **Before proceeding:** ✓ relevant documentation read ✓ codebase area explored and patterns understood ✓ tribal knowledge reviewed ✓ all doubts surfaced — none left unaddressed

## Step 4: Plan and Implement

Lean on existing skills — don't reinvent processes that already have a skill:

- **Complex task that needs a plan** → invoke `writing-plans`
- **Executing a plan with independent steps** → invoke `dispatching-parallel-agents` or `subagent-driven-development`
- **Implementing a feature or bugfix** → invoke `test-driven-development`
- **Hit a bug or unexpected behavior** → invoke `systematic-debugging`
- **Executing a written plan step by step** → invoke `executing-plans`

Follow the project conventions from the project and CLAUDE.md. When in doubt about a pattern, follow what the existing code does.

### Multi-repo Projects

Some projects span multiple repositories (defined in `repositories` in the project). When a task touches multiple repos:

1. Understand which repos are involved
2. Plan changes across repos before starting
3. Implement in dependency order (shared libs → backend → frontend)
4. Open separate PRs per repo, linking them together

> **Before proceeding:** ✓ implementation complete ✓ code follows project conventions and existing patterns ✓ no known issues left unresolved

## Step 5: Verify

Before delivering, make sure everything works:

1. Run the test commands from the project (`testing.commands`)
2. Run any linting/type-checking configured
3. **Integration / Smoke Testing** — if `testing.integration.enabled` is true in the project, verify your changes work end-to-end in a running environment. The approach depends on what you changed:

   **Setup (same for all types):**
   - Tell the user: "Implementation is done in worktree `<worktree-path>`. For integration tests I need the environment running. Can you start the required services from that path? Let me know when everything is up and the available URLs/ports."
   - Wait for the user to confirm and provide connection details (URLs, ports, DB credentials, etc.)

   **Then test based on what changed:**
   - **UI / frontend changes** → invoke `webapp-testing` to write and run Playwright tests against the running app
   - **API / backend changes** → write and run scripts that call the endpoints (REST, GraphQL, etc.) and verify responses, status codes, and payloads
   - **DB / migration changes** → write and run scripts that connect to the database and verify schema changes, data integrity, seed data
   - **Mixed changes** → combine the above as needed

   **If tests fail:** fix the code in the worktree, ask the user to restart affected services, and retest. Repeat until tests pass.

4. Invoke `verification-before-completion` to make sure nothing was missed
5. Invoke `requesting-code-review` for a self-review before opening the PR

If tests fail, fix them. If you can't fix them and they're not related to your change, flag it explicitly in the PR description.

> **Before proceeding:** ✓ all tests pass ✓ linting and type-checking pass ✓ integration tests pass (if applicable) ✓ self-review completed via `requesting-code-review`

## Step 6: Final Quality Checklist

Before delivering, run through every item in this checklist. This is not optional — each item must be explicitly verified. For each item, do one of three things:

1. **Pass** — you verified it and it's good
2. **Ask for help** — you can't resolve it alone, so ask the user how to fix it
3. **Ask to skip** — the item doesn't apply or can't be checked, so ask the user for permission to skip it (explain why)

Never silently skip an item. Never assume an item is fine without checking.

### The Checklist

| # | Check | How to Verify |
|---|-------|---------------|
| 1 | **Acceptance criteria met** | Re-read every acceptance criterion from the ticket. For each one, confirm your implementation satisfies it. If any criterion is ambiguous, ask the user. |
| 2 | **Tests pass** | Run the full test suite (`testing.commands` from project). Zero failures. |
| 3 | **Linting and type-checking pass** | Run all configured linters and type-checkers. Zero errors. |
| 4 | **No unrelated changes** | Review your diff — every changed file must be justified by the task. Revert anything that crept in (formatting changes, unrelated refactors). |
| 5 | **No secrets or credentials committed** | Scan staged files for API keys, passwords, tokens, `.env` values. Nothing sensitive goes into the commit. |
| 6 | **Commit messages follow conventions** | Check the project's commit convention (from project file or CLAUDE.md). Every commit must follow it. |
| 7 | **Branch name follows convention** | Verify the branch name matches the project's branching convention (e.g., `feat/{ticket-id}-{short-desc}`). |
| 8 | **Documentation updated** | If your change affects public APIs, configuration, or user-facing behavior, check whether docs need updating. If they do and you haven't updated them, do it now. |
| 9 | **PR description is complete** | The PR must link the ticket, describe what changed and why, and list anything reviewers should pay attention to. |
| 10 | **Integration tests pass** (if applicable) | If `testing.integration.enabled` is true, confirm the integration/smoke tests from Step 5 passed. |

### Presenting Results

After running through all 10 items, present a summary to the user:

```
Final quality checklist:
 1. Acceptance criteria met       ✅
 2. Tests pass                    ✅
 3. Linting/type-check pass       ✅
 4. No unrelated changes          ✅
 5. No secrets committed          ✅
 6. Commit messages OK            ✅
 7. Branch name OK                ✅
 8. Documentation updated         ⏭️ skipped (no public API changes)
 9. PR description complete       ✅
10. Integration tests pass        ⏭️ skipped (not configured)
```

Use ✅ for passed, ❌ for failed (with explanation), ⏭️ for skipped (with reason, only after user approval).

**Do not proceed to Step 7 until every item is either ✅ or ⏭️ (user-approved skip).** If any item is ❌, fix it or ask the user how to proceed.

## Step 7: Deliver and Learn

### 7a. Deliver

1. **Branch**: follow the branching convention from the project (e.g., `feat/{ticket-id}-{short-desc}`)
2. **PR**: invoke `finishing-a-development-branch` to handle the PR creation properly. Link the task/ticket in the PR description.
3. **Update the board**: move the task to the appropriate status (e.g., "In Review") using the board MCP tool
4. **Comment on the ticket**: add a comment on the ticket summarizing what was done, for historical traceability. The comment should be high-level but slightly technical — enough for someone reading the ticket months later to understand what changed and why, without needing to dig through the code. Include a link to the PR. Keep it concise (3-6 sentences). Use the board MCP tool (e.g., `addCommentToJiraIssue`).

   **Example comment:**
   > Implemented date filtering on the orders list. Added new query parameters `date_from`/`date_to` to the `GET /orders` endpoint with validation and tests. On the frontend, integrated a date range picker in the `OrdersFilter` component that calls the updated endpoint. PR: https://github.com/org/repo/pull/123

5. **Notify**: send a message to the configured notification channel with a summary of what was done and a link to the PR

### 7b. Learn

This is not a separate step — it's part of delivery. The task is not complete until you've reviewed what you learned and proposed improvements to the project's knowledge base. This matters because every discovery you capture saves time for the next task — whether it's you or someone else picking it up.

Consult `references/continuous-improvement.md` for the full protocol. In short:

1. **Identify discoveries** — undocumented patterns, prerequisites, gotchas, conventions, useful docs
2. **Propose changes** grouped by destination:
   - **Tribal knowledge** → project file (`tribal_knowledge`)
   - **Coding conventions** → repo `CLAUDE.md`
   - **Documentation sources** → project file (`documentation.sources`)
   - **Contacts updates** → project file (`contacts`)
3. **Wait for approval** — never write without the user's confirmation
4. **Apply only what's approved** — one item at a time, easy to review

If the task was routine and you learned nothing new, say so explicitly — e.g., "No new discoveries from this task." Don't silently skip it.

Include your learning summary in the same message as the delivery notification. This way the user sees it immediately and can approve changes while the context is fresh, rather than in a separate follow-up that might never happen.

> **Task complete when:** ✓ PR created and linked to ticket ✓ board status updated ✓ comment added to ticket ✓ team notified ✓ learning summary presented (even if "nothing new")

## Communicating with People

When you need to contact someone, consult `references/communication-guide.md` for:
- How to adapt communication style based on role (technical vs non-technical)
- How to reach people via the right channel (Slack, Teams, prompt)
- The async communication protocol (stop, notify user, wait for check)
- How to handle projects with no documentation

## Setup Verification

When starting on a project for the first time or when the user runs `/delivering-tickets:setup`, verify the environment is ready:

1. Check that required MCP tools from `setup.required_mcp` are available
2. Check that required plugins from `setup.required_plugins` are installed
3. Check that `~/.config/delivering-tickets/config.yml` exists with `projects` and `repositories`
4. Check that repos listed in `repositories` exist at the specified paths — if not, offer to clone them
5. If anything is missing, show the user exactly what to install and how

Don't silently skip steps because a tool is missing — surface it immediately.

## Examples

### Example 1: Specific ticket

User says: "Work on ticket ALPHA-342"

Actions:
1. Load project `projects/project-alpha.yml` → resolve repo paths
2. Fetch ALPHA-342 via Jira MCP → read description, acceptance criteria, comments
3. Assess: medium complexity, high knowledge → go autonomous
4. Explore relevant code area, check tribal knowledge
5. Implement following existing patterns, run tests
6. Run final quality checklist → present summary to user
7. Open PR linking ALPHA-342, move ticket to "In Review", comment on ticket with summary, notify on Slack, present learning summary

Result: PR ready for review, board updated, team notified.

### Example 2: Pick from the board

User says: "Pick up the next task from the board"

Actions:
1. Load project → fetch board via MCP
2. Filter by priority, pick highest-priority unassigned ticket
3. Show the ticket to the user: "Picked up ALPHA-501 — Add date filter. Shall I proceed?"
4. On confirmation → assess, explore, implement, verify, quality checklist, deliver, learn

Result: Top-priority task completed end-to-end.

### Example 3: Complex task, low knowledge

User says: "Implement ticket ALPHA-789"

Actions:
1. Load project → fetch ALPHA-789: "Migrate payment system to Stripe v2"
2. Assess: complex task, low knowledge → stop
3. Ask the user: "This task is complex and I don't have enough context. I'd like to ask Alice (tech lead) on Slack which migration approach the team prefers."
4. Send question via Slack MCP → wait for `/delivering-tickets:check`
5. On reply → plan, implement with checkpoints, verify, quality checklist, deliver, learn

Result: Complex task handled safely with human input at critical points.
