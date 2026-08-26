---
name: bugs
description: Scan the entire codebase for bugs and create prioritized Linear issues. Use this skill whenever the user runs /bugs, asks to find bugs, wants a bug audit, or wants code quality issues tracked in Linear.
---

# Bug Scanner

Exhaustively scan the codebase for bugs and create one Linear issue per finding, ordered by severity.

## Setup

Before scanning:

1. `mcp__claude_ai_Linear__list_teams` — identify the active team
2. `mcp__claude_ai_Linear__list_issue_labels` — note available labels
3. If no "bug" label exists: `mcp__claude_ai_Linear__create_issue_label` to create one

## Scanning

Explore the full directory tree first (LS, Glob), then read every non-trivial source file.
Don't skip files that look clean — bugs hide in boring code.

### What to look for

**Security** → Urgent or High
- Hardcoded secrets, API keys, passwords in source
- SQL or command injection via string concatenation or template literals
- Missing input validation at API or CLI boundaries
- Insecure deserialization or eval of external input
- Internal error details exposed to end users

**Error handling** → High or Medium
- Unhandled promise rejections (missing `.catch` or `await` without try/catch)
- Empty catch blocks that swallow exceptions silently
- Missing error checks after I/O operations
- Errors logged but not propagated when they should be

**Logic errors** → High or Medium
- Off-by-one errors in loops, slices, or index calculations
- Wrong comparison operators or inverted conditions
- Incorrect boolean logic (check De Morgan's law violations)
- State mutations that affect other code paths unexpectedly
- Missing edge cases (empty arrays, zero, negative numbers, null inputs)

**Null / undefined access** → Medium
- Property access without null checks where the value may be absent
- Missing optional chaining in chains that can be undefined
- Variables used before assignment

**Type safety** → Medium
- Implicit type coercion producing wrong results
- `parseInt` / `parseFloat` without radix
- Comparisons between values of incompatible types

**Resource management** → Medium or Low
- Connections or file handles not closed in finally blocks
- Timers or intervals created but never cleared
- Streams not destroyed on error

**Concurrency** → High or Medium
- Shared mutable state accessed from concurrent async paths
- Promises not awaited where ordering matters
- Race conditions in async event handlers

## Priority mapping

| Priority | Criteria |
|----------|----------|
| 1 Urgent | Security vulnerabilities, authentication bypass, data corruption, data loss |
| 2 High   | Crashes, incorrect behavior in critical paths, unhandled exceptions surfacing to users |
| 3 Medium | Logic errors in non-critical paths, missing null checks, incorrect error handling |
| 4 Low    | Edge cases, fragile patterns, minor incorrect behavior unlikely to be hit |

## Creating Linear issues

Rank all findings first, then create issues in order: Urgent → High → Medium → Low. Ask if it should be created in existing project or in a new project.
Creating in priority order ensures the backlog is correctly sorted.

**Title format:** `[Category] Short description`
Example: `[Security] SQL injection in user search endpoint`

**Description template:**
```
## Bug
<what the bug is and why it is wrong>

## Location
`path/to/file.ext:line`

## Impact
<what can go wrong if this is hit>

## Suggested fix
<concrete fix — be specific>
```

**Labels:** `["bug"]` plus any existing type labels that match (e.g. "security", "performance"). Do not create extra labels for categories.

Use `mcp__claude_ai_Linear__save_issue` for each issue.

## Output

When all issues are created:

```
Found N bugs — X urgent, Y high, Z medium, W low.
Created N Linear issues.
```

List each created issue ID and title.

## Rules

- Read every source file — do not sample or skip
- One issue per bug, not one per file or category
- Never create an issue for a pattern that is not actually wrong in this codebase
- Keep descriptions concrete: file path, line reference, exact problem, specific fix

## Related skills

- **linear-cli** — full CLI reference for label and issue management
- **product-planning** — for planning follow-up work after the bug audit
