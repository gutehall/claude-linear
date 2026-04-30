# /blocked - Mark an issue as blocked and document the blocker

## Usage

```
/blocked                     # Block the current issue (detect from branch)
/blocked ISSUE-12            # Block a specific issue
/blocked ISSUE-12 ISSUE-99   # ISSUE-12 is blocked by the existing ISSUE-99
```

## Flow

### Detect the issue

If no ID is provided, detect from the current branch name (`TEAM-123` pattern).
If that fails, ask: "Which issue is blocked?"

---

### Case A: blocker is an existing issue

If the user provides a blocking issue ID:

1. `linear issue update <id> --blocked-by <blocker-id>`
2. Confirm: "ISSUE-12 is now blocked by ISSUE-99. It will be hidden from `--unblocked` until ISSUE-99 is resolved."
3. Offer `/next` to find something else to work on

---

### Case B: blocker is unknown or external

If no blocker ID is provided:

1. Ask: "What's blocking this? Describe it or give an issue ID."
2. If they describe something external (waiting on credentials, a third party, a decision):
   - Determine priority: Urgent if it's actively preventing all work on the issue, High if it's blocking but work can partially continue, Medium otherwise.
   - Create the blocker with a proper description:
   ```bash
   linear issue create \
     --title "<short description of what is needed>" \
     --description "## Blocker\n<what is missing or needed>\n\n## Unblocks\n<ISSUE-ID>: <issue title>\n\n## Done when\n<what needs to happen for this to be resolved>" \
     --priority <urgent|high|medium> \
     --label blocker \
     --blocks <id>
   ```
   - If a "blocker" label does not exist in the workspace, omit `--label blocker`
3. Show the created blocker ID
4. Confirm: "Created <NEW-ID> as a blocker. ISSUE-12 is now blocked."
5. Offer `/next` to continue with unblocked work

---

## Notes

- Always offer `/next` after blocking — keep momentum going
- New blocker issues must have a description and a "done when" condition — a blocker with no resolution criteria is just noise
- Priority should reflect actual urgency, not always Urgent — reserve Urgent for blockers that halt all progress today
- A blocked issue disappears from `linear issues --unblocked` automatically once the blocker relationship is set
- To unblock later: `linear issue update <blocker-id> --status done` or close it with `linear issue close <blocker-id>`
