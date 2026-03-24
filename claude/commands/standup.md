# /standup - Daily standup summary

Use Linear MCP tools to generate a standup summary.

## Steps

1. **Fetch issues** using `mcp__claude_ai_Linear__list_teams` to find your team, then `mcp__claude_ai_Linear__list_issues` for your team:
   - Issues completed/done in the last 2 days
   - Issues currently in progress (assigned to me)
   - Issues that are blocked

2. **Fetch recent GitHub activity:**
   - `git log --oneline --since="2 days ago" --author="$(git config user.email)"` across current repo
   - `gh pr list --author @me --state all --limit 10 --json number,title,state,updatedAt` to find recent PRs

3. **Present the summary** in this format:

```
Yesterday:
✓ FIN-X: Issue title

Today (in progress):
→ FIN-X: Issue title

Blocked:
⊘ FIN-X: Issue title (reason if available)

Recent commits:
- repo: N commits
- PR #X merged/open: title
```

4. Offer `/next` as the default next step.

## Notes

- Skip sections that have nothing to show
- GitHub info requires `gh` CLI authenticated
- Always offer `/next` at the end
