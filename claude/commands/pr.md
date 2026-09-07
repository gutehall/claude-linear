# /pr - Open a pull request for current work

> **Project conventions:** follow the **ship-gate** skill's "Project conventions" rule (conventions skill → `CLAUDE.md`/`AGENTS.md`/`CONTRIBUTING.md` → detect from tooling).

Creates a PR from the current branch. Use this when you want review before the work is fully done, or when you want to separate "open PR" from "close issue".

For completed work, prefer `/done` — it handles commit, push, PR, and offers `/next` in one step.

## Usage

```
/pr                    # PR for current branch
/pr --draft            # Open as draft PR
/pr "custom title"     # Override the generated title
```

## Flow

1. **Detect the issue ID** from branch name (`TEAM-123` pattern)
   - If not found: ask "Which issue does this branch belong to? (e.g. ISSUE-12)"

2. **Detect `<base>` once** — follow the **ship-gate** skill's base-branch rule. Reuse `git log --oneline <base>..HEAD` and `git diff --stat <base>..HEAD` later in this run.

3. **Show work summary** (reuse the log/stat from step 2)

4. **Quality gate** — follow the **ship-gate** skill in **interactive** mode (G1 commit, G2 verify, G3 size-gated review). `/pr --draft` / user wants a WIP PR: ship-gate allows pushing with `--draft` and stating failures in the body.

5. **Push:**
   ```bash
   git push -u origin HEAD
   ```

6. **Build PR title:** `"ISSUE-12: Issue title"` — use custom title if provided

7. **Build PR body:**
   - Brief bullet summary of what changed
   - `## Test plan` with the real commands and results from G2 — never an unchecked checkbox
   - `## Known issues` for unresolved Medium/Low findings from G3, if any
   - `Closes ISSUE-12` at the end (required — triggers Linear integration)

8. **Create PR:**
   ```bash
   gh pr create --title "..." --body "..."
   # With --draft flag if requested
   ```

9. **Print the PR URL**

## If push fails due to diverged history

Follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, tell the user: "Conflicts in `<files>`. Resolve, run `git rebase --continue`, then run `/pr` again." Never force-push.

## Notes

- Body **must** contain `Closes <ID>` — this triggers Linear's GitHub integration
- Base branch: follow ship-gate (origin HEAD, else `main`)
- If a PR already exists for this branch (`gh pr view`), report the existing URL instead of creating a duplicate
- Do NOT run `linear issue close` — GitHub integration moves the Linear issue when the PR merges
- Follow the github-cli skill for PR body formatting
