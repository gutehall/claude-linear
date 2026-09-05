---
name: jira-cli
description: Manage Jira issues, sprints, and epics from the command line. Do not trigger proactively. Load only when a command says to follow this skill, or the user asks for Jira CLI syntax.
---

# Jira CLI

A terminal client for Jira Cloud and Jira Server.

Install: `go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest`
Or: `brew install jira-cli`

## First-Time Setup

```bash
jira init
```

Interactive setup that:
1. Asks for your Jira server URL (e.g. `https://yourcompany.atlassian.net`)
2. Asks for your email and API token (create at https://id.atlassian.com/manage-profile/security/api-tokens)
3. Asks which project to default to
4. Asks for board type (scrum or kanban)
5. Saves config to `~/.config/.jira/.config.yml`

## Configuration

Config file: `~/.config/.jira/.config.yml`

```yaml
server: https://yourcompany.atlassian.net
login: your@email.com
project: PROJ
board:
  type: scrum
  id: 1
```

Environment vars:
- `JIRA_API_TOKEN` — API token (overrides config file)
- `JIRA_AUTH_TYPE` — `bearer` or `basic`

### Avoid redundant lookups

Default `project` and `board.id` live in this config file — that's the cache for those facts. Check it (or what you already learned this session) before re-running `jira project list` or `jira board list` for the same facts. Only re-fetch if config is empty, stale, or the user is working a different project than the default.

## Quick Reference

```bash
# Identity
jira me                                   # Show current user account ID

# Issues
jira issue list                           # Open issues in configured project
jira issue list -a"$(jira me)"            # Your assigned issues
jira issue list -s"To Do"                 # Filter by status
jira issue list -s"In Progress"
jira issue list --sprint "Sprint 12"      # Issues in a sprint
jira issue list --epics PROJ-100          # Child issues of an epic
jira issue list --label bug               # Filter by label
jira issue list --type Bug                # Filter by type (Bug/Story/Task/Epic)
jira issue list --priority High           # Filter by priority
jira issue list --resolution Unresolved   # Open issues only
jira issue list --plain                   # Suppress interactive UI
jira issue view PROJ-123                  # Full issue details + comments
jira issue create -tStory -s"Title" -b"Description"
jira issue create -tBug -s"Bug title" -b"Desc" --priority High
jira issue create -tTask -s"Task" --parent PROJ-100
jira issue create -tEpic -s"Epic title" -b"Epic description"
jira issue create -tStory -s"Title" --label "bug,auth"
jira issue move PROJ-123 "In Progress"    # Transition status
jira issue assign PROJ-123 "$(jira me)"   # Assign to yourself
jira issue comment add PROJ-123 "Text"    # Add a comment
jira issue edit PROJ-123 --summary "New summary"
jira issue edit PROJ-123 --priority High
jira issue edit PROJ-123 --label "bug,security"
jira issue link PROJ-123 PROJ-456 "blocks"    # Link as blocking
jira issue unlink PROJ-123 PROJ-456
jira open PROJ-123                        # Open in browser

# Sprints
jira sprint list --board-id <id>          # All sprints
jira sprint list --board-id <id> --plain  # Clean output
jira sprint add --board-id <id> PROJ-123  # Add issue to sprint

# Epics
jira epic list                            # Epics in current project
jira epic list --plain
jira epic create -s"Epic title" -b"Description"

# Boards and projects
jira board list                           # All boards
jira board list --type scrum
jira project list                         # All accessible projects
```

## Issue Types

| Type | Use for |
|------|---------|
| Epic | Large initiative spanning multiple sprints |
| Story | User-facing feature or improvement |
| Bug | Defect or incorrect behavior |
| Task | Technical work, ops, chore |
| Sub-task | Child of any of the above |

## Priority Levels

| Priority | Meaning |
|----------|---------|
| Highest | Critical, blocking production or a release |
| High | Important, should be this sprint |
| Medium | Normal priority |
| Low | Nice to have |
| Lowest | Trivial |

## Common Statuses

Statuses are defined per workflow. Typical Scrum values:
- `Backlog` — not yet pulled into a sprint
- `To Do` — in sprint, not started
- `In Progress` — being worked on
- `In Review` — under code review or QA
- `Done` — complete

Run `jira issue move PROJ-1 --help` or view an issue to see valid transition targets for your workflow.

## Git Conventions

Always link git work to Jira issues:

```bash
# Branch naming — include issue key
git checkout -b PROJ-123-short-description

# Commit messages — include issue key for traceability
git commit -m "PROJ-123: Add cache invalidation on logout"

# PR title — include issue key for GitHub-Jira dev panel linking
gh pr create --title "PROJ-123: Add caching layer" \
  --body "## Summary\n...\n\nCloses PROJ-123"
```

### GitHub-Jira Integration

When the Jira GitHub app is installed, PRs containing `Closes PROJ-123` in the body auto-transition the issue to Done on merge.

Smart Commits (requires Jira DVCS connector):
```bash
git commit -m "PROJ-123 #in-progress Start implementation"
git commit -m "PROJ-123 #done #comment Fixes the root cause"
git commit -m "PROJ-123 #time 2h Investigating the issue"
```

## Workflow Guidelines

For worked examples (getting oriented, starting work, breaking down large tasks, blockers, adding notes, completing work, sprint management, parent epics), read `references/workflows.md`.

## Related Skills

- **product-planning** — for thinking through a problem before creating tickets. Use when the user has an idea or vague direction, not a ready-to-implement task.
- **github-cli** — for PR creation, review, and CI checks once the Jira issue is in progress.
