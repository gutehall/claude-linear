# /done - Complete work and ship

> **Project conventions:** (1) If this repo has a skill whose description says it defines coding rules, verify/test commands, or pre-merge gates — invoke it. (2) Else if `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` exists at the repo root (then the nearest subdirectory you are editing) — follow those. (3) Else detect test/build from tooling and continue.

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks. Supports **project** mode (one PR for the whole project branch) or **issue** mode (one PR for a single issue).

## Usage

```
/done                      # Ask project vs issue, then proceed
/done project              # Ship the current project branch
/done issue                # Ship the current issue
/done project "Phase 1"    # Named project (when branch is ambiguous)
/done issue ISSUE-12       # Specific issue
```

---

## Choose scope (always ask first)

Unless the user already chose via `project` or `issue` in the command:

> **Ship at project level or for a single issue?**
> - **Issue** (recommended) — one PR closing the issue on this issue branch. Smaller diff, its own CI run, far easier to review.
> - **Project** — one PR closing all issues worked on this project branch. Use only when the issues are inseparable — a multi-issue squashed PR is the hardest artifact to review.

Wait for the answer before pushing, creating a PR, or closing issues.

**Skip the question** when:
- The command includes `project` or `issue`
- The current branch clearly indicates mode: `<project-slug>-YYYY-MM-DD` → project; `TEAM-123-*` → issue

---

## Work Type Detection

- Project branch (`<project-slug>-YYYY-MM-DD`) or issue branch (`TEAM-123-*`) with git repo → code work
- No matching branch → non-code work
- If ambiguous, ask

---

## Quality gate (both code paths — mandatory before any push)

Follow the **ship-gate** skill in **interactive** mode after the work summary and before pushing. Compute `<base>` once (origin HEAD branch, else `main`) and reuse it. Do not push until G1–G3 pass. Their results feed the PR body.

---

## Path: Project — ship the whole project

### Resolve the project

1. From branch name: map `<project-slug>-YYYY-MM-DD` back via `linear projects`
2. If a project name is provided, use it
3. Otherwise ask which project this branch completes

### 1. Detect base branch

Follow the **ship-gate** skill's base-branch rule. Compute `<base>` once here and reuse it for every step below — do not re-derive it later in this run.

If `linear project show "<project>"` was already fetched earlier this session for the same project and no issue status/comment changed since, reuse it instead of re-fetching (see prior-work skill's session-cache note).

### 2. Gather project issues

```bash
linear project show "<project>"
linear issues --project "<project>" --status "In Progress"
```

Union In Progress issues with `TEAM-123` patterns from commit messages.

### 3. Show work summary

```bash
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

List every issue ID in this project push. Reuse this `git log`/`git diff --stat` output later in this run instead of re-running it.

### 4. Run the quality gate

Follow the **ship-gate** skill in **interactive** mode. All three checks must pass before anything is pushed.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create one project PR

Title: `<Project Name>: <short summary>`

Body — one `Closes ISSUE-ID` per issue, and a test plan filled with the **real commands and results from G2** (never an unchecked checkbox):

```bash
gh pr create --title "Phase 1: Caching and auth improvements" --body "$(cat <<'EOF'
## Summary

- What changed across the project

## Issues closed

Closes ISSUE-10
Closes ISSUE-12

## Test plan

- `npm test` — 42 passed, 0 failed
- `npm run build` — clean

## Known issues

- (Medium/Low findings from G3 not fixed in this PR, if any)
EOF
)"
```

Print the PR URL immediately.

### 7. Wait for CI

```bash
gh pr checks --watch
```

- Pass → continue to merge
- Fail → stop; fix and run `/done project` again
- **No checks configured** → there is no CI gate; the quality gate (G1–G3) was the only validation. Tell the user this repo has no CI, suggest branch protection (see the github-cli skill), and do **not** merge without their explicit confirmation.

### 8. Merge (confirm first)

Show the PR URL, CI result, and quality-gate results, then ask: **"Merge now?"** Do not merge without a yes.

```bash
gh pr merge --squash --delete-branch
```

If branch protection requires an approving review, arm auto-merge instead and finish — GitHub merges once a reviewer approves:

```bash
gh pr merge --auto --squash --delete-branch
```

### 9. Return to base

```bash
git checkout <base>
git pull
```

### 10. Complete the Linear project

- `linear issues --project "<project>" --open` — if none remain: `linear project complete "<project>"`
- If issues remain, list them and offer `/next project`

### Project code rules

- One PR per project branch from `/done project`
- Do not run `linear issue close` before merge — GitHub integration closes on merge
- No commits → skip PR and note it

---

## Path: Issue — ship a single issue

### 1. Detect the issue

From branch name (`TEAM-123` pattern) or the provided ID. If branch does not match:

1. `git log --oneline -5`
2. Ask: "Which issue does this complete? (e.g., ISSUE-12)"

### 2. Detect base branch

Follow the **ship-gate** skill's base-branch rule. Compute `<base>` once here and reuse it for every step below — do not re-derive it later in this run.

### 3. Show work summary

```bash
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

Reuse this output later in this run instead of re-running it.

### 4. Run the quality gate

Follow the **ship-gate** skill in **interactive** mode. All three checks must pass before anything is pushed.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create issue PR

Test plan must contain the **real commands and results from G2** — never an unchecked checkbox:

```bash
gh pr create --title "ISSUE-12: Issue title" --body "$(cat <<'EOF'
## Summary

- What changed and why

## Test plan

- `npm test` — 42 passed, 0 failed
- `npm run build` — clean

Closes ISSUE-12
EOF
)"
```

Print the PR URL immediately.

### 7. Wait for CI

```bash
gh pr checks --watch
```

- Pass → continue to merge
- Fail → stop; fix and run `/done issue` again
- **No checks configured** → there is no CI gate; the quality gate (G1–G3) was the only validation. Tell the user this repo has no CI, suggest branch protection (see the github-cli skill), and do **not** merge without their explicit confirmation.

### 8. Merge (confirm first)

Show the PR URL, CI result, and quality-gate results, then ask: **"Merge now?"** Do not merge without a yes.

```bash
gh pr merge --squash --delete-branch
```

If branch protection requires an approving review, arm auto-merge instead and finish — GitHub merges once a reviewer approves:

```bash
gh pr merge --auto --squash --delete-branch
```

### 9. Return to base

```bash
git checkout <base>
git pull
git log --oneline -5
```

### Issue code rules

- PR body **must** contain `Closes <ID>` for Linear GitHub integration
- Do **not** run `linear issue close` before merge
- If integration does not close the issue after merge, run `linear issue close <id>` as fallback
- No commits → skip PR and note it
- Worktree: show cleanup commands (`git worktree remove <path>`) after PR creation

### If push is rejected (both code paths)

Follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, stop, list files, tell the user to resolve and re-run `/done`. Never force-push.

---

## Path B: Non-Code Work

Ask scope if not already chosen, then:

**Project:** identify project, summarize all deliverables, collect artifact links, `linear issue close` per completed issue, `linear project complete` if done.

**Issue:** identify issue, summarize deliverable, artifact link via `mcp__claude_ai_Linear__save_comment`, `linear issue close <id>`.

Offer `/next` with matching scope.

No git, no PR.

---

## Notes

- Scope question comes **before** git operations
- Use `/done project` after `/next project` and `/done issue` after `/next issue`
- When in doubt about work type, check the branch pattern first
- The quality gate (G1–G3) is prompt-level discipline — make it unbypassable with branch protection on `main` (required checks + 1 approving review). See the github-cli skill's "Branch protection" section.
