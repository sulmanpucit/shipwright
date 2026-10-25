# Preflight: capabilities and how to fix gaps

Shipwright ships no credentials and bundles no servers. It uses whatever the user has connected. Each person signs in with their own accounts.

Load candidate tools in one ToolSearch call, for example `jira issue`, `linear issue`, `browser navigate`, `figma design`. A tool that only appears as a deferred name counts as available once loaded.

| Capability | Look for | If missing, tell the user | Without it |
| --- | --- | --- | --- |
| Ticket source: Jira | Atlassian MCP tools (`getJiraIssue`, `addCommentToJiraIssue`, attachment download) | Connect Atlassian: the claude.ai Atlassian connector, or `claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp` and sign in | Ask the user to paste the ticket text and screenshots |
| Ticket source: Linear | Linear MCP tools | Connect the Linear connector or its MCP server | Same as above |
| Ticket source: GitHub Issues | `gh issue view` works | `gh auth login` | Same as above |
| Browser | Claude in Chrome (`mcp__claude-in-chrome__*`), Playwright MCP, or Chrome DevTools MCP | Install the Claude in Chrome extension, or `claude mcp add playwright npx @playwright/mcp@latest` | Steps 2 and 6 fall back to code reading and tests; say so in every check-in |
| Git host | `gh auth status` succeeds | `gh auth login` | Step 7 stops after commit; the user pushes and opens the PR |
| Design (optional) | Figma MCP tools (`get_design_context`, `get_screenshot`) | Connect Figma: `claude mcp add --transport http figma https://mcp.figma.com/mcp` | Compare against ticket screenshots only |

Also check:
- `git status`: uncommitted or untracked changes. Never stash or discard them; ask.
- Current branch and whether it tracks a remote.
- `.claude/shipwright.md` present. If not, offer to create it (see SKILL.md, Project config).

Report the result as a short table in the Step 0 check-in.
