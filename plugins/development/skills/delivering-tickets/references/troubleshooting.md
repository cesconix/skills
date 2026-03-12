# Troubleshooting

## Project Issues

### Error: Project not found
**Cause**: No `.yml` file in `~/.config/delivering-tickets/projects/` matching the project name.
**Solution**: Run `/delivering-tickets:project` to create one interactively, or manually create `~/.config/delivering-tickets/projects/{project-name}.yml` using the schema in `references/project-schema.md`.

### Error: Local config missing
**Cause**: `~/.config/delivering-tickets/config.yml` doesn't exist.
**Solution**: Create it with your projects path and repositories:
```yaml
projects: ~/.config/delivering-tickets/projects
repositories:
  my-repo: ~/workspace/my-repo
```

### Error: Repository not cloned
**Cause**: The repository path from `config.yml` doesn't exist locally, or the repo name is missing from the `repositories` map.
**Solution**: Clone it using the `repo` URL from the project file, or run `/delivering-tickets:setup` to check all repos at once.

## MCP Connection Issues

### Board MCP not responding (Jira, ClickUp, GitHub)
**Cause**: MCP tool not configured or authentication expired.
**Solution**:
1. Check MCP is listed in Claude Code settings
2. Verify API key/OAuth token is valid
3. Test directly: ask Claude to fetch a ticket without the skill

### Slack/Teams MCP not sending messages
**Cause**: Missing permissions or wrong channel name.
**Solution**:
1. Verify the channel name in the project matches exactly (including `#`)
2. Check that the MCP has permission to post in that channel
3. Test directly: ask Claude to send a test message via the MCP

### Playwright MCP not available
**Cause**: Playwright MCP server not configured or browser not installed.
**Solution**:
1. Verify Playwright MCP is configured in Claude Code settings
2. Run browser install if needed

## Workflow Issues

### Skill doesn't trigger
**Symptom**: Saying "work on a ticket" doesn't activate delivering-tickets.
**Solution**: Use explicit invocation with `/delivering-tickets` or mention "delivering-tickets" in your prompt.

### Skill triggers but skips steps
**Symptom**: Agent jumps to implementation without exploring or planning.
**Solution**: Check the Decision Matrix — for complex tasks, say "check with me at each step" to force pauses.

### Integration tests can't connect
**Symptom**: Playwright or API tests fail with connection errors.
**Solution**:
1. Make sure services are running from the worktree path (not the main repo)
2. Confirm the URLs/ports the user provided are correct
3. Check that no firewall or proxy is blocking local connections

### Agent asks the wrong person
**Symptom**: Technical questions going to PM, or business questions going to developers.
**Solution**: Review the `contacts` section in the project file. Ensure `role` and `ask_about` fields are accurate.
