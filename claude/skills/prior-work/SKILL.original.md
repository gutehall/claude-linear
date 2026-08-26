---
name: prior-work
description: Before implementing a Linear issue, check whether the same work has already been done — a merged PR, existing code, an open branch, or a duplicate issue. Use this skill whenever picking up an issue to implement (/next, /grind, /autopilot) so effort isn't duplicated and already-solved work is caught before any code is written.
---

# Prior-Work Check

Before writing a line of code for an issue, find out whether the solution already exists. Catching "this is already done" or "this duplicates ISSUE-X" before implementing saves a wasted cycle and a redundant PR.

Run this **after** reading the issue's description and acceptance criteria, **before** branching/implementing.

## What "already done" means

Four outcomes, strongest evidence first:

| Outcome | Evidence |
|---------|----------|
| **Shipped** | A merged PR already closes this issue, or the codebase already implements the acceptance criteria in full. |
| **In flight** | An open PR or branch already references this issue / implements this work. |
| **Duplicate** | Another issue describes the same work and is open, in progress, or done. |
| **Partial** | A related solution exists that this work should extend, not rebuild from scratch. |
| **Clear** | No meaningful prior work — proceed normally. |

## How to check

Run these in parallel; each is cheap. Use the issue's key terms (feature name, file/module names, symbols, error text from the description).

1. **PRs and branches referencing the issue**
   ```bash
   gh pr list --state all --search "<ISSUE-ID>"          # PR that closes it
   gh pr list --state all --search "<key terms>"          # PR doing the same work
   git branch -a --list "*<issue-id>*"                    # existing local/remote branch
   ```
   Also check the issue itself for linked PRs/attachments — `mcp__claude_ai_Linear__get_issue` returns links; a merged-PR link is the strongest "shipped" signal.

2. **Duplicate or sibling issues**
   ```bash
   linear issues --open
   ```
   Compare the target against other issues by title/description (same signals the **dedupe** skill uses). A near-identical issue that's Done → likely shipped; one that's open/in-progress → duplicate or parallel work.

3. **Existing implementation in the code**
   Search the repo for the capability the issue asks for — function names, routes, config keys, UI strings from the acceptance criteria. If the feature is already present and meets the criteria, it's shipped (the issue is stale). If part of it exists, it's partial — extend it.

Keep the search proportional to the issue. A one-line fix needs a quick grep; a feature warrants checking PRs, issues, and code.

## Judging the match

Be precise, like in **dedupe** — a shared topic is not a duplicate. The test: would the prior work, as-is, satisfy this issue's acceptance criteria?

- **Yes, fully** → Shipped.
- **Yes, but it's still open/unmerged** → In flight.
- **Another ticket covers it** → Duplicate.
- **It gets you partway / is the right foundation** → Partial (extend).
- **Only superficially related** → Clear; proceed.

When uncertain between "partial" and "clear", treat as clear and proceed — but note what you found.

## Reporting

Always report the finding before acting, with specifics:

```
Prior-work check — ISSUE-12 "Add CSV export to billing"
  Shipped?   PR #188 "Export invoices as CSV" merged 2026-05-30 — implements src/billing/export.ts
  Duplicate? ISSUE-29 (Done) covers the same export
  → Looks already done. The acceptance criteria appear satisfied by existing code.
```

If clear, say so in one line and continue: `Prior-work check: no existing solution found. Proceeding.`

## Acting on the finding

### Interactive callers (/next, /start) — ask the user

Present the finding and the next-step options; do not decide unilaterally:

```
[Enter] Proceed anyway — implement as specified
[c]     Close ISSUE-12 as already done (comment linking PR #188, set Done)
[e]     Extend the existing solution — build on PR #188's code instead of starting fresh
[d]     Mark duplicate of ISSUE-29 — collapse via the dedupe flow
[q]     Quit
```

- **Close:** `mcp__claude_ai_Linear__save_comment` linking the prior work, then `linear issue update <id> --status "Done"`.
- **Extend:** point at the existing files/PR, then implement on top — reuse, don't rebuild.
- **Duplicate:** hand off to the **dedupe** skill's collapse action.

### Autonomous callers (/grind, /autopilot) — never prompt, never auto-close

An unattended run must not guess that work is "done" and close it, nor reimplement something already shipped. So:

- **Shipped / Duplicate (strong match):** do **not** implement. Move the issue out of the ready queue for a human to confirm:
  ```bash
  mcp__claude_ai_Linear__save_comment   # "Prior-work check: appears already done by PR #188 / duplicate of ISSUE-29. Flagged for human review."
  linear issue update <id> --status "Backlog" --label "needs-human"
  ```
  Print `<caller>: <id> appears already solved (PR #188 / ISSUE-29) — flagged needs-human, skipping.` Continue the loop; do not implement.
- **Partial:** proceed, but build on the existing code — reuse the foundation, keep the change minimal, note in the PR body what was extended.
- **In flight (open PR/branch by someone else):** do **not** stack a competing PR. Flag needs-human and skip, same as Shipped.
- **Clear:** proceed normally.

## Rules

- Run the check before branching/implementing — never after writing code.
- Never auto-close an issue on an autonomous run; flag `needs-human` and skip instead.
- A shared topic is not a duplicate — apply the dedupe precision test.
- When extending prior work, reuse it; do not rebuild what already exists.
- Report what you found even when proceeding — the finding is useful context for the PR.

## Related skills

- **dedupe** — detection signals and the collapse flow for true duplicate issues
- **github-cli** — PR and branch search reference
- **code-review** — verifying that existing code actually meets the acceptance criteria
