# /triage - Work through untriaged Linear issues interactively

Fetch all issues missing triage metadata and work through them one by one, suggesting values and applying confirmed changes via MCP.

## Usage

```
/triage              # All issues needing triage (no priority, no labels, no estimate, or unassigned)
/triage --priority   # Only issues with no priority set
/triage --unlabeled  # Only issues with no labels
```

## Flow

### 1. Load context

Fetch in parallel:
- `mcp__claude_ai_Linear__list_teams` — get team ID
- `mcp__claude_ai_Linear__list_issues` — all open issues
- `mcp__claude_ai_Linear__list_issue_labels` — available labels
- `mcp__claude_ai_Linear__list_projects` — available projects

Filter issues based on the flag passed:
- No flag: issues where priority is 0 (none), OR labels are empty, OR estimate is unset
- `--priority`: issues where priority is 0 (none) only
- `--unlabeled`: issues where labels are empty only

Report: `Found N issues needing triage.`

If 0 issues: say "All issues are triaged. Nothing to do." and stop.

If the MCP call fails: say "Could not reach Linear. Check your MCP connection with `/mcp`." and stop.

### 2. Work through issues one at a time

For each issue, fetch full details via `mcp__claude_ai_Linear__get_issue`, then display:

```
--- Issue FIN-42 ---
Title: <title>
Description: <first 200 chars of description, or "(no description)" if blank>
Current: priority=<none|Urgent|High|Medium|Low>, labels=<none|label1, label2>, estimate=<none|XS|S|M|L|XL>
```

Analyze the title and description to suggest values. Apply these rules:

**Priority suggestion:**
- Suggest based on urgency signals in the title and description
- Words like "crash", "broken", "data loss", "security", "blocked" → Urgent or High
- New features, improvements, refactors → Medium or Low
- Vague or very short descriptions → Low, and note that more detail is needed
- Do not suggest a new priority if one is already set — leave it as-is

**Labels suggestion:**
- Pick from the available labels fetched in step 1 — never invent labels
- Match on content: a bug report gets "bug", an auth issue gets "auth" if that label exists, etc.
- Suggest multiple labels if more than one fits
- If no labels are available in Linear: skip label suggestion

**Estimate suggestion:**
- XS: trivial change, one file, clearly understood
- S: small, well-scoped, a few files
- M: moderate scope, some uncertainty
- L: significant work, multiple components
- XL: large, not well-defined, or clearly a multi-PR effort
- Do not suggest an estimate if one is already set

**Project suggestion:**
- Only suggest a project if the issue clearly belongs to one based on its content
- If no project is an obvious match: do not suggest one — leave it unset

Present suggestions as:

```
Suggested:
  priority: High
  labels: [bug, auth]
  estimate: S
  project: Phase 1  (or "(none)" if no match)

[Enter] Accept  [e] Edit  [s] Skip  [q] Quit
```

**On accept:** call `mcp__claude_ai_Linear__save_issue` with all suggested fields that are not already set. Print: `Updated FIN-42.`

**On edit:** prompt for each field individually in sequence:
- `Priority (current: <value>) [1 Urgent / 2 High / 3 Medium / 4 Low / Enter to keep]:`
- `Labels (available: <list>) [comma-separated names / Enter to keep]:`
- `Estimate [XS / S / M / L / XL / Enter to keep]:`
- `Project [<name> / Enter to keep / n for none]:`
Then apply via `mcp__claude_ai_Linear__save_issue`. Print: `Updated FIN-42.`

**On skip:** move to next issue. Print: `Skipped FIN-42.`

**On quit:** stop immediately and go to summary.

### 3. Summary

```
Triaged N issues. Skipped M.
```

List each triaged issue ID and title on its own line.

## Error Handling

- If `save_issue` fails for an issue: print the error, offer to retry or skip, do not silently continue
- If a label name entered during edit does not match any available label: warn and re-prompt
- If the user quits mid-batch: report progress on issues already handled before stopping

## Rules

- Never override a field that already has a value, unless the user explicitly edits it
- Only use labels that already exist in Linear — do not create new ones
- Do not force a project assignment — leave it unset if nothing clearly fits
- Keep suggestions grounded in the issue content — do not guess randomly
- No clipboard, no JSON payloads, no external scripts
