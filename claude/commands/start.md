# /start - Begin work on a Linear issue

Sets up your working context — assign, move to In Progress, create branch, show full issue — without immediately implementing. Use this when you already know what to work on and just need the context loaded.

## Usage

```
/start ISSUE-12    # Start a specific issue
/start             # Pick from unblocked issues
```

## Flow

1. **If no ID given:** run `linear issues --unblocked`, show up to 5 options, let user pick
2. **Assign and set In Progress:** `linear issue start <id>`
3. **Detect work type** (same logic as `/next`): code vs. non-code based on title, description, labels
4. **For code work:** `linear branch <id>` — creates and checks out a git branch
5. **Show full context:** `linear issue show <id>`
6. **Confirm:** "Branch `<branch-name>` checked out. Ready when you are."

## Notes

- This sets up context only — it does not start implementing. Start coding when you're ready.
- If the branch already exists locally, check it out without erroring
- For non-code issues, skip the branch step
- When done, run `/done` to create the PR and close the issue
- Prefer `/next` if you want to find work AND start implementing in one step
