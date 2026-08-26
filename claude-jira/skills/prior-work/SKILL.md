---
name: prior-work
description: Before implementing a Jira issue, check whether the same work has already been done — a merged PR, existing code, an open branch, or a duplicate issue. Use this skill whenever picking up an issue to implement (/next, /grind, /autopilot, /start) so effort isn't duplicated and already-solved work is caught before any code is written.
---

# Prior-Work Check

Before writing a line of code for an issue, find out whether the solution already exists. Catching "this is already done" or "this duplicates PROJ-X" before implementing saves a wasted cycle and a redundant PR.

Run this **after** reading the issue's description and acceptance criteria, **before** branching/implementing.

## What "already done" means

| Outcome | Evidence |
|---------|----------|
| **Shipped** | A merged PR already closes this issue, or the codebase already implements the acceptance criteria in full. |
| **In flight** | An open PR or branch already references this issue / implements this work. |
| **Duplicate** | Another issue describes the same work and is open, in progress, or done. |
| **Partial** | A related solution exists that this work should extend, not rebuild from scratch. |
| **Clear** | No meaningful prior work — proceed normally. |

## How to check

Run these in parallel; each is cheap. Use the issue's key terms (feature name, file/module names, symbols, error text).

1. **PRs and branches referencing the issue**
   ```bash
   gh pr list --state all --search "<ISSUE-KEY>"          # PR that closes it
   gh pr list --state all --search "<key terms>"           # PR doing the same work
   git branch -a --list "*<issue-key>*"                    # existing local/remote branch
   ```
   The Jira GitHub dev panel also surfaces linked PRs — `jira issue view <key>` or the Atlassian MCP shows development links; a merged-PR link is the strongest "shipped" signal.

2. **Duplicate or sibling issues**
   Use the Atlassian MCP's issue-search tool (check available `mcp__claude_ai_Atlassian__*` tools for exact name) or `jira issue list` to pull candidates, then compare by summary/description (same signals the **dedupe** skill uses). A near-identical issue that's Done → likely shipped; one open/in-progress → duplicate or parallel work. Check existing **Duplicate** links on the issue too.

3. **Existing implementation in the code**
   Search the repo for the capability the issue asks for — function names, routes, config keys, UI strings from the acceptance criteria. If the feature is already present and meets the criteria, it's shipped (issue is stale). If part of it exists, it's partial — extend it.

Keep the search proportional to the issue. A one-line fix needs a quick grep; a feature warrants checking PRs, issues, and code.

## Judging the match

Be precise, like in **dedupe** — a shared topic is not a duplicate. Test: would the prior work, as-is, satisfy this issue's acceptance criteria?

- **Yes, fully** → Shipped.
- **Yes, but still open/unmerged** → In flight.
- **Another ticket covers it** → Duplicate.
- **It gets you partway / is the right foundation** → Partial (extend).
- **Only superficially related** → Clear; proceed.

When uncertain between "partial" and "clear", treat as clear and proceed — but note what you found.

## Reporting

Always report the finding before acting, with specifics:

```
Prior-work check — PROJ-12 "Add CSV export to billing"
  Shipped?   PR #188 "Export invoices as CSV" merged 2026-05-30 — implements src/billing/export.ts
  Duplicate? PROJ-29 (Done) covers the same export
  → Looks already done. The acceptance criteria appear satisfied by existing code.
```

If clear: `Prior-work check: no existing solution found. Proceeding.`

## Acting on the finding

### Interactive callers (/next, /start) — ask the user

```
[Enter] Proceed anyway — implement as specified
[c]     Close PROJ-12 as already done (link PR #188, transition to Done)
[e]     Extend the existing solution — build on PR #188's code instead of starting fresh
[d]     Mark duplicate of PROJ-29 — collapse via the dedupe flow
[q]     Quit
```

- **Close:** `jira issue comment add <key> "Already done by PR #188."`, then `jira issue move <key> "Done"`.
- **Extend:** point at the existing files/PR, then implement on top — reuse, don't rebuild.
- **Duplicate:** hand off to the **dedupe** skill's collapse action (`jira issue link <dup> <canon> "Duplicate"` + close).

### Autonomous callers (/grind, /autopilot) — never prompt, never auto-close

An unattended run must not guess that work is "done" and close it, nor reimplement something already shipped:

- **Shipped / Duplicate / In flight (strong match):** do **not** implement. Comment, then move it out of the queue for a human:
  ```bash
  jira issue comment add <key> "Prior-work check: appears already done by PR #188 / duplicate of PROJ-29. Flagged for human review."
  jira issue move <key> "Backlog"
  jira issue edit <key> --label needs-human --no-input
  ```
  Print `<caller>: <key> appears already solved (PR #188 / PROJ-29) — flagged needs-human, skipping.` Continue the loop; do not implement.
- **Partial:** proceed, but build on the existing code — reuse the foundation, keep the change minimal, note in the PR body what was extended.
- **Clear:** proceed normally.

## Rules

- Run the check before branching/implementing — never after writing code.
- Never auto-close an issue on an autonomous run; flag `needs-human` and skip instead.
- A shared topic is not a duplicate — apply the dedupe precision test.
- When extending prior work, reuse it; don't rebuild what already exists.
- Report what you found even when proceeding — the finding is useful PR context.

## Related skills

- **dedupe** — detection signals and the collapse flow for true duplicate issues
- **github-cli** — PR and branch search reference
- **code-review** — verifying that existing code actually meets the acceptance criteria
