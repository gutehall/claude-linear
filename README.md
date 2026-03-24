# Linear Workflow with Claude Code

This documents the development loop used in this project: Linear for issue tracking, Claude Code for implementation, and a set of slash commands that tie them together.

Everything runs inside Claude Code — no browser, no second terminal window.

---

## How Commands Work in Claude Code

There are two types of commands:

**Slash commands** — typed directly in the Claude Code chat. Claude executes them.
```
/next
/done
/plan "Add retry logic"
/standup
```

**Shell commands** — prefixed with `!` to run in the current terminal session without leaving Claude Code.
```
!gh pr checks
!gh pr merge --squash --delete-branch
!git log --oneline
```

---

## Installation

### 1. Get the commands

Clone this repo:

```bash
git clone https://github.com/gutehall/claude-linear.git
```

The slash commands live in `.claude/commands/` and skills in `.claude/skills/`. Two options:

**Per-project** — copy `.claude` into your project root:
```bash
cp -r claude-linear/.claude /path/to/your/project/
```

**Global** — available in all projects:
```bash
cp claude-linear/.claude/commands/* ~/.claude/commands/
cp -r claude-linear/.claude/skills/* ~/.claude/skills/
```

### 2. Linear CLI

```bash
npm install -g @dabble/linear-cli
linear login
```

`linear login` will:
1. Ask where to save credentials (project or global)
2. Open Linear API settings in your browser
3. Prompt you to paste your API key
4. Let you select your team
5. Save config

Config is layered: `~/.linear` (global) is loaded first, then `./.linear` (project-level) overrides it. Env vars (`LINEAR_API_KEY`, `LINEAR_TEAM`) are used as fallbacks.

### 3. Linear MCP Server

The slash commands (`/plan`, `/standup`) talk to Linear via MCP. Configure the Linear MCP server in Claude Code settings so it has access to your Linear workspace.

```bash
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```

Then run `/mcp` once you've opened a Claude Code session to go through the authentication flow.

### 4. GitHub CLI

`/done` creates PRs via `gh`. Install and authenticate:

```bash
brew install gh
gh auth login
```

### 5. gh aliases

The workflow uses short aliases for common PR commands. Set them up once:

```bash
gh alias set prc 'pr checks'
gh alias set prv 'pr view'
gh alias set prd 'pr diff'
gh alias set prl 'pr list'
gh alias set prm 'pr merge --squash --delete-branch'
gh alias set co 'pr checkout'
```

Verify with `gh alias list`.

---

## Daily Loop

```
/standup → /next → implement → /done → !gh pr checks → !gh pr merge → /next → repeat
```

---

## Daily Workflow — Step by Step

### Morning

**1. Get oriented**

Type in Claude Code:
```
/standup
```
Shows what you completed yesterday, what's in progress, and any blocked issues. Pulls from both Linear and GitHub.

**2. Pick the next issue**

```
/next
```
Claude shows up to 3 unblocked issues (yours first). Select one — it gets marked In Progress in Linear, a git branch is created, and Claude starts reading the issue and relevant code.

---

### The implementation loop — repeat until done for the day

**3. Implement**

Claude reads the issue, explores the codebase, and implements. You review and guide as needed. When the work is done, move to the next step.

**4. Commit, push, and open a PR**

```
/done
```
Claude stages any uncommitted changes, commits, pushes the branch, and creates a PR with `Closes FIN-X` in the body. The PR URL is printed.

**5. Check CI**

```
!gh prc
```
Wait for green. If a check fails, re-run it:
```
!gh run list
!gh run rerun <run-id> --failed
```

**6. Review the diff**

```
!gh prv
!gh prd
```

**7. Merge**

```
!gh prm
```

Note: GitHub does not allow approving your own PR. Skip the approve step when self-merging. `gh pr review --approve` is only used when reviewing someone else's PR.

On merge, the `Closes FIN-X` in the PR body automatically moves the Linear issue to Done. No manual step needed.

**8. Pick the next issue**

```
/next
```
Repeat from step 3.

---

### When you need to plan new work

```
/plan "Add retry logic to the analyzer"
```

Claude reads current Linear state, asks clarifying questions if needed, drafts issues with descriptions and acceptance criteria, creates them directly in Linear, then suggests `/next <ID>` to start immediately.

---

### End of day (optional)

```
/standup
```
Quick summary of what shipped, what's still open, any PRs pending review.

---

## Commands Reference

### `/next` — Pick the next issue and start working

```
/next           # Show unblocked issues to choose from
/next FIN-12    # Jump straight to a specific issue
```

What it does:
1. Runs `linear issues --unblocked` to find issues ready to work on
2. Presents up to 3 options (your assigned issues sorted first), plus a "Product planning" option
3. On selection: marks the issue In Progress, creates a git branch, reads the issue, explores relevant code, and begins implementation

The branch name is derived from the issue ID and title: `fin-42-add-caching-layer`.

---

### `/plan` — Plan work and create Linear issues

```
/plan                        # Open-ended planning session
/plan "Add retry logic"      # Focused planning for a specific area
```

What it does:
1. Reads current Linear state (open issues, projects, teams) via MCP
2. If the input is vague, asks: what needs to happen, who is it for, how do you know it's done
3. Drafts issues with titles, descriptions, and acceptance criteria
4. Creates them directly in Linear via MCP — no clipboard, no copy-paste
5. Prints each created issue ID and title, then suggests `/next <ID>` for the top priority

Issues are kept small (one PR each). L/XL issues get broken into sub-issues.

---

### `/done` — Commit, push, and create a PR

```
/done           # Complete the current issue (detected from branch name)
/done FIN-12    # Complete a specific issue
```

What it does:
1. Detects the issue ID from the current branch name
2. Shows a summary: commits and files changed
3. Stages and commits any uncommitted changes
4. Pushes the branch to origin
5. Creates a PR titled `FIN-12: Issue title` with `Closes FIN-12` in the body
6. Prints the PR URL

The `Closes FIN-12` triggers Linear's GitHub integration — the issue moves to Done automatically when the PR merges. Do not manually close the issue.

For non-code work (documents, decks, research): closes the issue directly in Linear, optionally attaching a link first.

---

### `/standup` — Daily standup summary

```
/standup
```

What it does:
1. Fetches via MCP: issues completed in the last 2 days, issues currently in progress, blocked issues
2. Fetches recent git commits and open/merged PRs via `gh`
3. Presents a summary:

```
Yesterday:
✓ FIN-X: Issue title

Today (in progress):
→ FIN-X: Issue title

Blocked:
⊘ FIN-X: Issue title

Recent commits:
- repo: N commits
- PR #X merged: title
```

---

## PR Lifecycle Reference

All run inside Claude Code using the `!` prefix. No browser needed.

Aliases are configured globally (`gh alias list` to verify) for the most common commands:

| What | Short | Full command |
|------|-------|-------------|
| Check CI status | `!gh prc` | `!gh pr checks` |
| View PR description | `!gh prv` | `!gh pr view` |
| View full diff | `!gh prd` | `!gh pr diff` |
| List all open PRs | `!gh prl` | `!gh pr list` |
| Merge your own PR (squash) | `!gh prm` | `!gh pr merge --squash --delete-branch` |
| Check out a PR locally | `!gh co 42` | `!gh pr checkout 42` |
| Approve (someone else's PR) | — | `!gh pr review --approve` |
| Request changes | — | `!gh pr review --request-changes -b "reason"` |
| Re-run failed CI | — | `!gh run rerun <run-id> --failed` |
| Watch live CI output | — | `!gh run watch` |

> Note: GitHub does not allow approving your own PR. Skip the approve step when self-merging.

---

## Useful Linear CLI Commands

Run these in a terminal or with `!` prefix inside Claude Code.

```bash
# Overview
linear roadmap                      # Projects, milestones, progress
linear issues --unblocked           # Ready to work on
linear issues --mine                # Your assigned issues
linear issue show FIN-5             # Full issue details with parent context

# Context (scope all commands to a project/milestone)
linear project open "Phase 1"
linear milestone open "Sprint 3"
linear project close

# Create issues
linear issue create --title "Fix bug" --assign --estimate S
linear issue create --title "Step 2" --parent FIN-5 --estimate M

# Update issues
linear issue update FIN-5 --append "## Notes\n\nFound root cause in auth.ts:142"
linear issue update FIN-5 --check "validation"   # Check off a checklist item

# Blockers
linear issue create --title "Need API key" --blocks FIN-5
# FIN-5 disappears from --unblocked until this is resolved

# Git
linear branch FIN-5                 # Create and check out branch: fin-5-issue-title
```

---

## Estimate Sizes

| Size | Meaning |
|------|---------|
| XS | Trivial, < 1 hour |
| S | Small, a couple hours |
| M | Medium, about a day |
| L | Large, multi-day — consider breaking down |
| XL | Very large — break down before starting |

L/XL issues should be split into sub-issues before work begins.

#WayOfWorking
