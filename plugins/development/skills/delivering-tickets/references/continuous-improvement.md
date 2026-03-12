# Continuous Improvement

After delivering a task (Step 7a), review what you learned during the work and propose improvements. This step makes the project smarter over time.

## What to Capture

During Steps 1-7a, keep track of discoveries that fall into these categories:

### 1. Tribal Knowledge (→ project `tribal_knowledge`)

Things you discovered the hard way that would save time for the next person:

- Undocumented prerequisites (e.g., "run `npm run db:seed` before tests")
- Non-obvious patterns (e.g., "all endpoints require the `withAuth()` wrapper from `src/middleware/auth.ts`")
- Service quirks (e.g., "the staging payment service has a 30s timeout — mock it in tests")
- Pitfalls (e.g., "don't import directly from `src/legacy/` — use the adapters in `src/adapters/`")
- Environment gotchas (e.g., "Redis must be running locally for the cache layer, even in dev")

**Rule**: If you had to figure it out by reading code, asking someone, or failing first — it's tribal knowledge.

### 2. Project Conventions (→ repo `CLAUDE.md`)

Patterns that are consistently followed in the codebase but not documented:

- Validation library used across the project (e.g., "always use `zod` for input validation")
- Error handling patterns (e.g., "use Result types, not exceptions")
- File/folder naming conventions (e.g., "services go in `src/services/{name}/index.ts`")
- Testing conventions (e.g., "co-locate tests as `__tests__/{name}.test.ts`")
- Import ordering or aliasing rules

**Rule**: If the existing code follows a pattern consistently, it belongs in CLAUDE.md so future work follows it too.

### 3. Documentation Sources (→ project `documentation.sources`)

Useful docs you discovered during exploration:

- README files in subdirectories
- Inline docs or wikis you found via MCP
- Confluence/Notion pages referenced in code comments
- API docs or OpenAPI specs in the repo

**Rule**: If you read it and it helped you understand the codebase, add it to sources.

### 4. Contacts & Expertise (→ project `contacts`)

People knowledge you gained:

- Someone who helped you that's not in the project file yet
- Updated `ask_about` topics based on who actually knew the answer
- New communication channels discovered

## How to Propose

After delivery, present discoveries grouped by destination. Be concise — one line per item.

**Format:**

```
Completed {TICKET-ID}. During the work I discovered a few things worth capturing:

**Tribal knowledge** (project file {project}.yml):
- "{discovery 1}"
- "{discovery 2}"

**Project conventions** (repo CLAUDE.md):
- "{convention 1}"

**New documentation sources** (project file {project}.yml):
- {path or URL}

Want me to apply any of these changes?
```

## Rules

1. **Never write without approval** — always propose, never apply autonomously
2. **Be specific** — "use withAuth() wrapper" is good, "follow auth patterns" is vague
3. **Don't duplicate** — check existing tribal knowledge and CLAUDE.md before proposing
4. **Skip if nothing new** — if a task was routine and you learned nothing new, say so and move on
5. **Keep items atomic** — one concept per line, easy to accept or reject individually
6. **Prefer tribal_knowledge for project-specific gotchas**, CLAUDE.md for coding conventions that tools can enforce

## Applying Approved Changes

When the user approves items:

- **Tribal knowledge**: append new entries to the `tribal_knowledge` list in the project YAML
- **CLAUDE.md**: add conventions to the appropriate section (create one if it doesn't exist). Follow the existing CLAUDE.md style of the repo.
- **Documentation sources**: append to `documentation.sources` in the project file
- **Contacts**: add or update entries in `contacts` in the project file
