# /done - Complete work on an issue

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks.

## Usage

```
/done           # Complete the current issue
/done ISSUE-12  # Complete a specific issue
```

## Work Type Detection

Determine work type the same way `/next` does — issue title, description, labels, and whether a git branch matching the issue ID exists.

If a branch like `fin-42-*` or `lin-42-*` is checked out, treat as code work.
If no such branch exists, treat as non-code work.
If ambiguous, ask.

---

## Path A: Code Work

1. **Detect the issue** from the branch name (extract `TEAM-123` pattern) or use the provided ID
2. **Show work summary:**
   - `git log --oneline <base>..HEAD`
   - `git diff --stat <base>..HEAD`
3. **Stage and commit** any uncommitted changes (if any exist)
4. **Push branch** to origin
5. **Create PR:**
   ```
   gh pr create --title "ISSUE-12: Issue title" --body "## Summary\n...\n\nCloses ISSUE-12"
   ```
6. Print the PR URL
7. Offer `/next` to continue

### Code Rules

- The PR body **must** contain `Closes <ID>` (e.g., `Closes FIN-42`) — this triggers Linear's GitHub integration to auto-move the issue to Done on merge
- Do **not** run `linear done` when creating a PR — GitHub integration handles Linear status on merge
- If there are no commits, skip the PR step and note it
- If in a worktree, run `linear done <id>` after PR creation and show the cleanup commands

---

## Path B: Non-Code Work

1. **Identify the issue** from the provided ID, or ask which issue was just completed
2. **Summarize** what was produced (document title, deck name, decision made, etc.)
3. **Ask for an artifact link:** "Any link to attach? (Google Doc, Slides, Notion, etc.)"
   - If yes: post it as a Linear comment via `mcp__claude_ai_Linear__save_comment` before closing
   - If no: skip
4. **Close in Linear:** run `linear done <id>`
5. Offer `/next` to continue

No git, no PR, no branch. Just capture the output and close.

---

## Notes

- The same command works regardless of role — manager closing a deck, engineer shipping a PR
- When in doubt about work type, check for a matching git branch first
