---
name: whatchanged
description: Show everything that has changed since the last time this command was run — git commits, files, Linear issues, and PRs. Persists a checkpoint so each run only shows new changes. Use this skill whenever the user runs /whatchanged or asks what has changed since the last run.
---

# What Changed

Show a focused diff of activity since the last checkpoint. Persist the checkpoint so the next run starts where this one left off.

## State file

The checkpoint lives at `.claude/whatchanged` in the project root (create `.claude/` if it does not exist).

File format (plain text, two lines):

```
timestamp=<ISO-8601 datetime>
commit=<full SHA>
```

Example:

```
timestamp=2026-05-01T14:32:00Z
commit=a3f9d12e8b1c4f7a0e2d5b6c9f3a8e1d4b7c0f2
```

## Step 1: Read the checkpoint

```
cat .claude/whatchanged 2>/dev/null
```

- If the file does **not** exist: this is the first run. Print:

  > "No previous checkpoint found. Recording current state as the baseline — run `/whatchanged` again to see future changes."

  Then jump straight to **Step 5** (write the checkpoint and stop — do not show a diff).

- If the file exists: parse `timestamp` and `commit` from it.

## Step 2: Gather changes (run all in parallel)

Use the parsed `<commit>` as the git baseline and `<timestamp>` as the time baseline.

### Git commits

```
git log <commit>..HEAD --oneline --no-merges
```

If `<commit>` is no longer in the repo history (e.g. after a rebase): fall back to `--since=<timestamp>`.

### Files changed

```
git diff --stat <commit>..HEAD
```

Summarise by grouping changes under top-level directories. Cap the display at 20 files; if more, show the count.

### Merged PRs

```
gh pr list --state merged --json number,title,mergedAt,author --limit 50
```

Filter client-side to PRs where `mergedAt` > `<timestamp>`.

If `gh` is unavailable: skip silently and note "GitHub data unavailable" at the bottom.

### Open PRs created since last run

```
gh pr list --state open --json number,title,createdAt,author --limit 50
```

Filter to `createdAt` > `<timestamp>`.

### Linear issues updated since last run

Call `mcp__claude_ai_Linear__list_teams` to get the team ID, then `mcp__claude_ai_Linear__list_issues` filtered to `updatedAt` >= `<timestamp>`.

Group by status change direction:
- Newly created (createdAt >= timestamp)
- Moved to Done/Cancelled
- Moved to In Progress
- Everything else updated

If Linear MCP is unavailable: skip silently and note "Linear data unavailable" at the bottom.

## Step 3: Present the changes

Use this format. Omit any section that has no items.

```
## What changed since <timestamp>

### Commits (<N>)
- <sha7> <message>
...

### Files changed
src/
  foo.ts (M), bar.ts (A)
lib/
  util.ts (D)
<N more files not shown>

### PRs merged (<N>)
- #N: <title> — @author

### PRs opened (<N>)
- #N: <title> — @author

### Linear issues
#### New
- FIN-N: <title>

#### Shipped (→ Done)
- FIN-N: <title>

#### Started (→ In Progress)
- FIN-N: <title>

#### Updated
- FIN-N: <title>

---
Checkpoint: <old-timestamp> → <new-timestamp>
```

If **nothing changed** in any category: print "No changes since `<timestamp>`." and still update the checkpoint.

## Step 4: Offer actions

After showing the summary, offer contextually relevant follow-ups:

- If there are Linear issues in Done: suggest `/retro` to reflect on shipped work
- If there are open PRs: suggest `/review` to review them
- If there are commits but no PRs: suggest `/pr` to open a PR

Only mention actions that are relevant to what was found.

## Step 5: Write the checkpoint

Always update the checkpoint at the end, even on first run or when nothing changed.

```
git rev-parse HEAD
```

Write `.claude/whatchanged` with:

```
timestamp=<current ISO-8601 datetime in UTC>
commit=<HEAD sha>
```

Example write command:
```
printf 'timestamp=%s\ncommit=%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "$(git rev-parse HEAD)" > .claude/whatchanged
```

## Rules

- Never show changes older than the checkpoint — the point is a focused view, not a full history
- Always update the checkpoint, even if the run fails mid-way or shows nothing
- Do not create any Linear issues or PRs from this command — read-only only
- If the state file is corrupt or unparseable: treat it as missing (first-run path)
- The `.claude/` directory is project-local; do not write the state file anywhere else

## Related skills

- **retro** — for a deeper retrospective view with action item creation
- **standup** — for a time-windowed summary oriented around your own activity
- **release** — for cutting a release based on what changed
