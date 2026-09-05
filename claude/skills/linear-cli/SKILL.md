---
name: linear-cli
description: Manage Linear issues and projects from the command line. Do not trigger proactively. Load only when a command says to follow this skill, or the user asks for Linear CLI syntax.
allowed-tools: Bash(linear:*), Bash(curl:*)
---

# Linear CLI

Cross-platform CLI for Linear's GraphQL API, with unblocked issue filtering.

Install: `npm install -g @dabble/linear-cli`

## First-Time Setup

```bash
linear login
```

This will:
1. Ask where to save credentials (project or global)
2. Open Linear API settings in your browser
3. Prompt you to paste your API key
4. Show available teams, let you pick one (or create a new team)
5. Save config to chosen location

## Configuration

Config is layered: `~/.linear` (global) loads first, then `./.linear` (local) overrides on top. Local values override global; unset local values inherit from global. Env vars (`LINEAR_API_KEY`, `LINEAR_TEAM`, `LINEAR_PROJECT`, `LINEAR_MILESTONE`) used as fallbacks.

```
# .linear file format
api_key=lin_api_xxx
team=ISSUE
project=My Project
milestone=Sprint 3

[aliases]
V2=Version 2.0 Release
MVP=MVP Milestone
```

**Durable cache:** `.linear` config (team, default project, aliases) is the stable cache for these facts across sessions. Check it before calling `list_teams`/`list_projects`/`list_issue_labels` MCP tools again — only re-fetch if config is stale, empty, or the workspace changed.

## Aliases

Create short codes for projects and milestones:

```bash
# Create aliases
linear alias V2 "Version 2.0"           # For a project
linear alias MVP "MVP Milestone"        # For a milestone

# List all aliases
linear alias --list

# Remove an alias
linear alias --remove MVP
```

Use aliases anywhere a project or milestone name is accepted:

```bash
linear issues --project V2
linear issues --milestone MVP
linear issue create --project V2 --milestone MVP "New feature"
linear milestones --project V2
```

Aliases shown in `linear projects` and `linear milestones` output:

```
Projects:
[V2] Version 2.0 Release  started  58%

Milestones:
[MVP] MVP Milestone [next]
```

## Quick Reference

```bash
# Auth
linear login                    # Interactive setup
linear logout                   # Remove config
linear whoami                   # Show current user/team

# Roadmap (overview)
linear roadmap                  # Projects with milestones and progress

# Default context (sets project/milestone for issues & create)
linear project open "Phase 1"    # Set default project
linear milestone open "Sprint 3" # Set default milestone
linear project close             # Clear default project
linear milestone close           # Clear default milestone

# Issues
linear issues                    # Default: backlog + todo issues (filtered by open project/milestone)
linear issues --no-project       # Bypass default project filter
linear issues --no-milestone     # Bypass default milestone filter
linear issues --unblocked       # Ready to work on (no blockers)
linear issues --open            # All non-completed issues
linear issues --status todo     # Only todo issues
linear issues --status backlog  # Only backlog issues
linear issues --status in-progress # Issues currently in progress
linear issues --status todo --status in-progress # Multiple statuses
linear issues --mine            # Only your assigned issues
linear issues --project "Name"  # Issues in a project
linear issues --milestone "M1"  # Issues in a milestone
linear issues --label bug       # Filter by label
linear issues --priority urgent # Filter by priority (urgent/high/medium/low/none)
# Flags can be combined: linear issues --status todo --mine
linear issue show ISSUE-1        # Full details with parent context
linear issue start ISSUE-1       # Assign to you + set In Progress
linear issue create --title "Fix bug" --project "Phase 1" --assign --estimate M
linear issue create --title "Urgent bug" --priority urgent --assign
linear issue create --title "Task" --milestone "Beta" --estimate S
linear issue create --title "Blocked task" --blocked-by ISSUE-1 --blocked-by ISSUE-2
linear issue create --title "Labeled" --label bug --label frontend  # Multiple labels
linear issue update ISSUE-1 --status "In Progress"
linear issue update ISSUE-1 --priority high   # Set priority
linear issue update ISSUE-1 --estimate M      # Set estimate
linear issue update ISSUE-1 --label bug --label frontend  # Set labels (repeatable)
linear issue update ISSUE-1 --assign          # Assign to yourself
linear issue update ISSUE-1 --parent ISSUE-2  # Set parent issue
linear issue update ISSUE-1 --milestone "Beta"
linear issue update ISSUE-1 --append "Notes..."
linear issue update ISSUE-1 --check "validation" # Check off a todo item
linear issue update ISSUE-1 --blocks ISSUE-2 --blocks ISSUE-3  # Repeatable
linear issue close ISSUE-1
linear issue comment ISSUE-1 "Comment text"

# Projects
linear projects                 # Active projects
linear projects --all           # Include completed
linear project show "Phase 1"   # Details with issues
linear project create "Name" --description "..."
linear project complete "Phase 1"
linear project open "Phase 1"   # Set as default project
linear project close            # Clear default project

# Milestones
linear milestones --project "P1" # Milestones in a project
linear milestone show "Beta"     # Details with issues
linear milestone create "Beta" --project "P1" --target-date 2024-03-01
linear milestone open "Beta"    # Set as default milestone
linear milestone close          # Clear default milestone

# Reordering (drag-drop equivalent)
linear projects reorder "P1" "P2" "P3"           # Set project order
linear project move "Urgent" --before "Phase 1"  # Move single project
linear milestones reorder "Alpha" "Beta" --project "P1"
linear milestone move "Beta" --after "Alpha" --project "P1"
linear issues reorder ISSUE-1 ISSUE-2 ISSUE-3    # Set issue order
linear issue move ISSUE-5 --before ISSUE-1       # Move single issue

# Labels
linear labels                   # List all labels
linear label create "bug" --color "#FF0000"

# Aliases
linear alias V2 "Version 2.0"        # Create alias for project/milestone
linear alias --list                  # List all aliases
linear alias --remove V2             # Remove alias
# Then use: linear issues --project V2

# Git
linear branch ISSUE-1            # Create branch: ISSUE-1-issue-title
```

## Estimation

Use t-shirt sizes for estimates. Always use `--estimate` (not `-e`) for clarity.

| Size | Meaning |
|------|---------|
| XS | Trivial, < 1 hour |
| S | Small, couple hours |
| M | Medium, a day or so |
| L | Large, multi-day - consider breaking down |
| XL | Very large - should definitely break down |

```bash
# Create with estimate (use long flags for clarity)
linear issue create --title "Add caching" --estimate M --assign

# L/XL issues should be broken into sub-issues
linear issue create --title "Implement auth" --estimate L
linear issue create --title "Add login endpoint" --parent ISSUE-5 --estimate S
linear issue create --title "Add JWT validation" --parent ISSUE-5 --estimate S
```

## Git Conventions

Always link git work to Linear issues:

```bash
# Create branch from issue (recommended)
linear branch ISSUE-5            # Creates: ISSUE-5-add-caching-layer

# Commit message format
git commit -m "ISSUE-5: Add cache invalidation on logout"

# Include issue ID in PR title
gh pr create --title "ISSUE-5: Add caching layer"
```

## Workflow Guidelines

For worked examples (setting context, getting oriented, starting work, hitting a blocker, breaking down large tasks, checklists vs sub-issues, completing work, adding notes, organizing with milestones, completing a phase, multi-team setup) and parent-issue context, read `references/workflows.md`.

## Related Skills

- **product-planning** — for thinking through a problem before creating tickets. Use when user has an idea or vague direction, not a ready-to-implement task.
- **github-cli** — for PR creation, review, and CI checks once the Linear issue is in progress.
