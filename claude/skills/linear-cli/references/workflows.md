# Linear CLI — Workflow Guidelines & Parent Context

Worked examples for common linear-cli workflows, referenced from the main `linear-cli` SKILL.md. Read this file only when one of these specific scenarios applies.

## Workflow Guidelines

### Setting context
Working on a specific project/milestone: set as default to avoid repeating flags:
```bash
linear project open "Phase 1"    # All commands now default to Phase 1
linear milestone open "Sprint 3" # And to Sprint 3 milestone
linear issues                    # Shows Phase 1 > Sprint 3 issues only
linear issue create --title "Fix" # Created in Phase 1, Sprint 3
linear project close             # Done? Clear the context
```

### Getting oriented
```bash
linear roadmap                  # See all projects, milestones, progress
linear issues --project "P1"    # Issues in a specific project
linear issues --milestone "M1"  # Issues in a specific milestone
```

### Starting work on an issue
```bash
linear issues --unblocked       # Find what's ready
linear issue show ISSUE-2        # Review it (shows parent context)
linear issue start ISSUE-2       # Assign + set In Progress
linear branch ISSUE-2            # Create git branch
```

### When you hit a blocker
Work can't continue due to a dependency or external factor:

```bash
# Create the blocking issue
linear issue create --title "Need API credentials" --blocks ISSUE-5

# Or mark existing issue as blocking
linear issue update ISSUE-3 --blocks ISSUE-5
```

Removes ISSUE-5 from `--unblocked` results until the blocker is resolved.

### When a task is larger than expected
Discover an M issue is actually L/XL — break it down:

```bash
# Create sub-issues
linear issue create --title "Step 1: Research approach" --parent ISSUE-5 --estimate S
linear issue create --title "Step 2: Implement core logic" --parent ISSUE-5 --estimate M
linear issue create --title "Step 3: Add tests" --parent ISSUE-5 --estimate S

# Start working on the first sub-issue
linear issue start ISSUE-6
```

### Checklists vs. sub-issues
Description checklists for lightweight steps within a single issue. Sub-issues when items need their own status, assignee, or estimate.

```bash
# Checklist — quick implementation steps, a punch list, acceptance criteria
linear issue update ISSUE-5 --append "## TODO\n- [ ] Add validation\n- [ ] Update tests\n- [ ] Check edge cases"

# Check off completed items (fuzzy matches the item text)
linear issue update ISSUE-5 --check "validation"
linear issue update ISSUE-5 --check "tests"

# Uncheck if needed
linear issue update ISSUE-5 --uncheck "validation"

# Sub-issues — substantial, independently trackable work
linear issue create --title "Add login endpoint" --parent ISSUE-5 --estimate S
```

Prefer checklists when items are small and don't need independent tracking. Prefer sub-issues when you'd want to assign, estimate, or block on them individually. Use `--check` to mark items complete as you finish them.

### Completing work
After finishing implementation, ask the developer if they want to close the issue:

```bash
# Suggest closing
linear issue close ISSUE-5
```

Don't auto-close issues. Let the developer review the work first.

### Adding notes while working
```bash
linear issue update ISSUE-2 --append "## Notes\n\nDiscovered X, trying Y approach..."
# or for quick updates
linear issue comment ISSUE-2 "Found the root cause in auth.ts:142"
```

### Organizing with milestones
Milestones group related issues within a project:

```bash
# Create milestone for a release
linear milestone create "Beta" --project "Phase 1" --target-date 2024-03-01

# Add issues to milestone
linear issue create --title "Core feature" --milestone "Beta" --estimate M
linear issue update ISSUE-5 --milestone "Beta"

# Reorder milestones to reflect priority
linear milestones reorder "Alpha" "Beta" "Stable" --project "Phase 1"
```

### Completing a phase
```bash
linear issue close ISSUE-7       # Close remaining issues
linear project complete "Phase 1"
# Then update CLAUDE.md status table
```

### Multi-team setup

Workspace has multiple teams — switch between them:

```bash
linear login                          # Re-run to switch teams
linear whoami                         # Confirm current team
LINEAR_TEAM=OTHER linear issues       # Override team for a single command
```

For per-project team configuration, set `team=TEAMKEY` in the project's `.linear` file. Local `.linear` overrides global `~/.linear`, so each project can target a different team automatically.

## Parent Context

Viewing an issue with `linear issue show`, you'll see where it fits in the larger work:

```
# ISSUE-6: Add JWT validation

State: In Progress
...

## Context

ISSUE-3: Implement authentication system
  - [Done] ISSUE-4: Add login endpoint
  → [In Progress] ISSUE-6: Add JWT validation  ← you are here
  - [Backlog] ISSUE-7: Add refresh tokens
```

Helps understand scope and what comes before/after the current task.
