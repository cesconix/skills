# Health Check Output Format

Output a compact status block. No box-drawing borders (emojis have variable width in terminals and break alignment). Use indentation, blank lines as separators, and emoji status indicators at the **start** of each checkable line.

## Example — All Green

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

## Example — Issues Found

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

## Rules

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
