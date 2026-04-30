# /standup - Daily standup summary

Use Linear MCP tools to generate a standup summary.

## Steps

1. **Determine the lookback window:**
   - If today is **Monday**: use **3 days** (covers Friday through Sunday)
   - All other days: use **2 days**

2. **Check sprint/cycle context:** call `mcp__claude_ai_Linear__list_cycles` to find the active cycle. If the cycle ends within 2 days, note it in the summary and suggest `/retro` at the end instead of `/next`.

3. **Fetch issues** using `mcp__claude_ai_Linear__list_teams` to find your team, then `mcp__claude_ai_Linear__list_issues` for your team:
   - Issues completed/done within the lookback window
   - Issues currently in progress (assigned to me)
   - Issues that are blocked

4. **Fetch recent GitHub activity:**
   - `git log --oneline --since="<N> days ago" --author="$(git config user.email)"` (use the lookback window from step 1)
   - `gh pr list --author @me --state all --limit 10 --json number,title,state,updatedAt` to find recent PRs

5. **Present the summary** in this format:

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

6. **Offer the right next step:**
   - If the cycle ends within 2 days: suggest `/retro` to wrap up the sprint
   - Otherwise: suggest `/next` to pick up work

## Notes

- Skip sections that have nothing to show
- GitHub info requires `gh` CLI authenticated
- If `list_cycles` fails or returns no cycles, skip cycle detection silently and always offer `/next`
