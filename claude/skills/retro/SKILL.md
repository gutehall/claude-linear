---
name: retro
description: Run a sprint retrospective by pulling completed and in-progress work from Linear and git/GitHub, then create Linear issues for action items. Use this skill whenever the user runs /retro or asks for a sprint retrospective.
---

# Sprint Retrospective

Surface what shipped, what slipped, and what never started — then turn patterns into Linear issues.

## Setup

Before fetching data:

1. `mcp__claude_ai_Linear__list_teams` — identify the active team and its ID
2. `mcp__claude_ai_Linear__list_cycles` — check for a recently completed cycle
3. `mcp__claude_ai_Linear__list_issue_statuses` — map status names to categories (Done, In Progress, Todo, Blocked, Cancelled)

## Time window resolution

Resolve the period in this order:

1. If a completed cycle exists with end date within the last 14 days: use `cycle.startsAt` → `cycle.endsAt`
2. If the user passed `Nw`: start = N weeks ago, end = today
3. If the user passed `YYYY-MM-DD`: start = that date, end = today
4. Default: start = 14 days ago, end = today

Always print the resolved period before any data output.

## Data gathering

Fetch all sources in parallel.

### Linear issues

Use `mcp__claude_ai_Linear__list_issues` with the resolved team ID and appropriate filters:

| Category | Filter criteria |
|----------|----------------|
| Shipped | Status = Done or Cancelled, updatedAt >= start |
| Slipped | Status = In Progress, createdAt < start, updatedAt >= start |
| Blocked | Status = Blocked or has blocking label, active in period |
| Never started | Status = Todo or Backlog, createdAt >= start |

For slipped issues, calculate days in progress: today minus the date the issue moved to In Progress (use `issue.startedAt` if available, else `issue.createdAt`).

### Git commits

```
git log --since=<start_date> --oneline --no-merges
```

Count total commits. Note the range for the velocity section.

### GitHub PRs

```
gh pr list --state merged --json number,title,mergedAt --limit 50
```

Filter client-side to mergedAt >= start_date.

```
gh pr list --state open --json number,title,createdAt
```

If `gh` fails or is not installed, continue with Linear data only and note the gap.

## Linking issues to PRs

For each shipped issue, check if a merged PR title or branch contains the issue ID (e.g. `FIN-42`). If matched, append `(PR #N)` to that line.

## Output format

```
## Retro: <start_date> → <end_date>

### Shipped
✓ FIN-N: Issue title (PR #N if available)

### Slipped (started but not finished)
→ FIN-N: Issue title — in progress for N days

### Never started
○ FIN-N: Issue title — created but not picked up

### Blocked
⊘ FIN-N: Issue title — blocked by FIN-X

### Velocity
- Closed: N issues
- PRs merged: N
- Commits: N
```

Omit sections with no items. Do not pad empty sections.

## Pattern analysis

After presenting the summary, ask the user whether to create action items. Do not create issues unprompted.

When the user confirms, scan the data for these patterns:

| Pattern | Signal | Action item type |
|---------|--------|-----------------|
| Recurring blockers | Same issue type blocked 2+ times | Root cause investigation issue |
| Chronic slippage | Same issue or area slips 2+ periods | Scoping or estimation issue |
| Empty cycles | Never-started count > shipped count | Planning or prioritization issue |
| No PR coverage | Commits with no associated PR | Process or workflow issue |
| Single reviewer | All PRs reviewed by same person | Team process issue |
| Late PR creation | PRs opened the same day as merge | Review process issue |

Only surface patterns that are actually present in the data. Do not invent findings.

## Creating action items

Use `mcp__claude_ai_Linear__save_issue` for each action item.

**Title format:** `[Retro] Short description of the action`
Example: `[Retro] Resolve repeated auth-service blocking dependency`

**Description template:**

```
## Pattern observed
<what was seen in the retro data — be specific, reference issue IDs or PR numbers>

## Proposed action
<what to do about it>

## Definition of done
<how you know this is resolved>
```

**Priority mapping:**

| Priority | When to use |
|----------|-------------|
| 2 High | Blocking work, repeated failures in critical path |
| 3 Medium | Process gaps, estimation or scoping problems |
| 4 Low | Nice-to-have process improvements |

Print each created issue ID and title after creation.

## Rules

- Never create action items without user confirmation
- One issue per pattern, not one per occurrence
- Keep titles action-oriented, not symptom-oriented
- Reference concrete data (issue IDs, counts) in descriptions — do not generalize
- If the user dismisses a pattern, drop it without argument

## Related skills

- **product-planning** — for turning action items into structured work
- **linear-cli** — for full CLI reference on issue and label management
