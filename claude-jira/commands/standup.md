# /standup - Daily standup summary

Use the Jira CLI and git to generate a standup summary.

## Steps

1. **Determine the lookback window:**
   - If today is **Monday**: use **3 days** (covers Friday through Sunday)
   - All other days: use **2 days**

2. **Fetch Jira activity:**
   - Issues you recently completed: `jira issue list -a"$(jira me)" -s"Done" --plain`
   - Issues currently in progress: `jira issue list -a"$(jira me)" -s"In Progress" --plain`
   - Issues in review: `jira issue list -a"$(jira me)" -s"In Review" --plain`
   - Blocked issues: `jira issue list -a"$(jira me)" -s"Blocked" --plain` (if the project has no Blocked status, check flagged/"blocked"-labeled issues instead)

3. **Fetch recent GitHub activity:**
   - `git log --oneline --since="<N> days ago" --author="$(git config user.email)"` (use the lookback window from step 1)
   - `gh pr list --author @me --state all --limit 10 --json number,title,state,updatedAt` to find recent PRs

4. **Present the summary** in this format:

```
Yesterday:
✓ PROJ-X: Issue summary

Today (in progress):
→ PROJ-X: Issue summary

In review:
⏳ PROJ-X: Issue summary (PR #N)

Blocked:
⊘ PROJ-X: Issue summary (reason if available)

Recent commits:
- repo: N commits
- PR #X merged/open: title
```

5. Offer `/next` as the default next step.

## Notes

- Skip sections that have nothing to show
- GitHub info requires `gh` CLI authenticated
- Jira "Done" issues may include older closed issues — filter by recent activity visually if the list is long
- Always offer `/next` at the end
