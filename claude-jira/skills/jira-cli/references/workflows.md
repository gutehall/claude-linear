# Jira CLI — Worked Examples

Referenced from `jira-cli/SKILL.md`. Read this file when you need a worked example for one of these situations — not needed for a quick command lookup (that's in the main SKILL.md Quick Reference).

## Getting oriented

```bash
jira board list                           # Find your board ID
jira sprint list --board-id <id>          # See active sprint
jira issue list --sprint "Sprint 12" --resolution Unresolved --plain
jira issue list -a"$(jira me)" --plain    # Your issues
```

## Starting work

```bash
jira issue list -s"To Do" -a"$(jira me)" --plain   # What's assigned and ready
jira issue view PROJ-123                            # Read the full issue
jira issue move PROJ-123 "In Progress"              # Transition to in progress
jira issue assign PROJ-123 "$(jira me)"             # Assign to yourself
git checkout -b PROJ-123-short-description          # Create branch
```

## When a task is larger than expected

Break it into child issues:

```bash
jira issue create -tTask -s"Step 1: Research approach" --parent PROJ-123
jira issue create -tTask -s"Step 2: Implement core logic" --parent PROJ-123
jira issue create -tTask -s"Step 3: Add tests" --parent PROJ-123
jira issue move PROJ-124 "In Progress"              # Start the first child
```

## When you hit a blocker

```bash
# Link the blocking issue
jira issue link PROJ-123 PROJ-456 "blocks"
# PROJ-456 now blocks PROJ-123

# Add a comment explaining the blocker
jira issue comment add PROJ-123 "Blocked on PROJ-456: need API credentials from infra team"
```

## Adding notes while working

```bash
jira issue comment add PROJ-123 "Found root cause in auth.ts:142"
jira issue edit PROJ-123 --body "Updated: discovered X, trying Y approach"
```

## Completing work (no GitHub integration)

```bash
jira issue move PROJ-123 "Done"
```

Don't auto-close issues without confirming with the developer first.

## Sprint management

```bash
jira sprint list --board-id <id>           # See all sprints and their state
jira sprint add --board-id <id> PROJ-123   # Add an issue to the active sprint
```

## Parent context and epics

Epics group related stories. When creating stories under an epic:

```bash
# Create child issue under an epic
jira issue create -tStory -s"Add login page" --parent PROJ-100

# View all child issues of an epic
jira issue list --epics PROJ-100 --plain
```
