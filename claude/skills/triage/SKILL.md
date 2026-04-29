---
name: triage
description: Interactively triage Linear issues that are missing priority, labels, estimate, or project assignment. Use this skill whenever the user runs /triage or asks to triage, review, or clean up untriaged issues in Linear.
---

# Issue Triage

Work through untriaged Linear issues interactively, suggesting metadata based on issue content and applying confirmed values via MCP.

## Setup

Before triaging:

1. `mcp__claude_ai_Linear__list_teams` — identify the active team
2. `mcp__claude_ai_Linear__list_issue_labels` — fetch all available labels
3. `mcp__claude_ai_Linear__list_projects` — fetch all active projects
4. `mcp__claude_ai_Linear__list_issues` — fetch open issues, then filter to untriaged

An issue needs triage if any of the following are true:
- Priority is 0 (none)
- Labels list is empty
- Estimate is unset

## Filtering

| Flag | Filter |
|------|--------|
| (none) | priority=none OR labels=empty OR estimate=unset |
| `--priority` | priority=none only |
| `--unlabeled` | labels=empty only |

## Suggesting values

For each issue, fetch full details via `mcp__claude_ai_Linear__get_issue` and analyze the title and description.

### Priority

| Signal | Suggested priority |
|--------|--------------------|
| "crash", "broken", "data loss", "security", "blocked", "outage" | 1 Urgent or 2 High |
| Feature request, improvement, refactor | 3 Medium or 4 Low |
| Vague title, missing description | 4 Low — note that more detail is needed |
| Priority already set | Do not override |

### Labels

- Only suggest labels that exist in Linear — never invent new ones
- Match on content: bug reports → "bug", auth issues → "auth" (if label exists), etc.
- Suggest multiple labels if more than one applies
- Skip label suggestion if no labels exist in the workspace

### Estimate

| Size | Criteria |
|------|----------|
| XS | Trivial change, one file, clearly understood |
| S | Small, well-scoped, a few files |
| M | Moderate scope, some uncertainty |
| L | Significant work, multiple components |
| XL | Large, poorly defined, or clearly multi-PR |
| (skip) | Estimate already set — do not override |

### Project

- Only suggest a project if the issue clearly belongs to one
- If no project is an obvious match, leave it unset — do not force an assignment

## Interaction loop

Present one issue at a time:

```
--- Issue FIN-42 ---
Title: <title>
Description: <first 200 chars or "(no description)">
Current: priority=<value>, labels=<values or none>, estimate=<value or none>

Suggested:
  priority: High
  labels: [bug, auth]
  estimate: S
  project: Phase 1

[Enter] Accept  [e] Edit  [s] Skip  [q] Quit
```

**Accept:** apply all suggested values for fields not already set via `mcp__claude_ai_Linear__save_issue`.

**Edit:** prompt each field in sequence with current value shown; apply on confirmation.

**Skip:** move to next issue without changes.

**Quit:** stop and print the summary.

## Applying updates

Use `mcp__claude_ai_Linear__save_issue` with only the fields being changed. Never send a field that is already set unless the user explicitly changed it during edit.

## Output

After all issues are processed:

```
Triaged N issues. Skipped M.
```

List each triaged issue ID and title.

## Error handling

- `save_issue` failure: print the error, offer retry or skip — do not silently continue
- Unknown label name during edit: warn and re-prompt
- MCP unreachable: say "Could not reach Linear. Check your MCP connection with `/mcp`." and stop

## Rules

- Never override a field that already has a value unless the user explicitly edits it
- Only use labels that already exist in Linear
- Do not force project assignment
- Keep suggestions grounded in actual issue content

## Related skills

- **linear-cli** — full MCP tool reference for issue and label management
- **product-planning** — for planning new issues after a triage pass
