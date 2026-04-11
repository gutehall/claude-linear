# /retro - Sprint retrospective

Pull completed and in-progress work from Linear and git/GitHub, then create action items as Linear issues.

## Usage

```
/retro              # Last 2 weeks
/retro 1w           # Last week
/retro 3w           # Last 3 weeks
/retro 2024-01-01   # Since a specific date
```

## Steps

### 1. Determine the time window

- Call `mcp__plugin_linear_linear__list_cycles` to find the most recently completed cycle.
- If a completed cycle exists and its end date is within the last 2 weeks: use that cycle's start and end dates as the period.
- Otherwise: parse the argument to determine the lookback date:
  - No argument → 2 weeks ago
  - `Nw` → N weeks ago
  - `YYYY-MM-DD` → that exact date
- Print the resolved period at the top: `Period: YYYY-MM-DD → YYYY-MM-DD`

### 2. Gather data (run in parallel)

**From Linear** — call `mcp__plugin_linear_linear__list_teams` first to get the team ID, then:
- Issues with status Done or Cancelled updated within the period
- Issues that were In Progress at the start of the period and are still open (slipped)
- Issues with a "Blocked" status or blocking label that were active in the period
- Issues created within the period that were never started (status still Todo/Backlog)

**From git/GitHub:**
- `git log --since=<start_date> --oneline --no-merges` — commits in period
- `gh pr list --state merged --json number,title,mergedAt --limit 50` — filter by mergedAt >= start_date
- `gh pr list --state open --json number,title,createdAt` — PRs still open

### 3. Present the retrospective

Use this exact format. Omit any section that has no items.

```
## Retro: <start_date> → <end_date>

### Shipped
✓ FIN-N: Issue title (PR #N if available)
...

### Slipped (started but not finished)
→ FIN-N: Issue title — in progress for N days
...

### Never started
○ FIN-N: Issue title — created but not picked up
...

### Blocked
⊘ FIN-N: Issue title — blocked by FIN-X
...

### Velocity
- Closed: N issues
- PRs merged: N
- Commits: N
```

### 4. Identify patterns and create action items

After presenting the summary, ask exactly:

> "Any patterns worth addressing? I can create Linear issues for action items."

If the user confirms or describes patterns, help identify concrete actions. Examples:
- Recurring blockers → issue to resolve the root cause
- Issues that keep slipping → scoping or estimation issue
- No tests in shipped code → tech-debt issue
- Single author on all PRs → process or pairing issue

Create each action item via `mcp__plugin_linear_linear__save_issue` with:
- A clear title describing the action, not the symptom
- Priority based on impact (blocking = High, process = Medium, polish = Low)
- Appropriate labels if they exist

Print each created issue ID and title after creation.

## Error handling

- If Linear returns no issues: say "No closed issues found in this period. Is the MCP connected? Try `/mcp`." and stop.
- If `gh` is not available or returns an error: show the Linear summary only, note "GitHub data unavailable (gh not authenticated or not installed)."
- If no cycles exist in Linear: skip cycle detection silently and use the time-based window.
- If issue creation fails: report the specific error and offer to retry.

## Notes

- Follow the retro skill for pattern analysis guidance
- Keep action items small enough to implement in one PR
- Do not create action items unless the user asks for them
