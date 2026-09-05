# /done - Complete work and ship

> **Project conventions:** (1) If this repo has a skill whose description says it defines coding rules, verify/test commands, or pre-merge gates — invoke it. (2) Else if `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` exists at the repo root (then the nearest subdirectory you are editing) — follow those. (3) Else detect test/build from tooling and continue.

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks. Supports **project** mode (one PR for the epic branch) or **issue** mode (one PR for a single issue).

## Usage

```
/done                      # Ask project vs issue, then proceed
/done project              # Ship the current epic branch
/done issue                # Ship the current issue
/done project PROJ-100     # Specific epic (when branch is ambiguous)
/done issue PROJ-12        # Specific issue
```

---

## Choose scope (always ask first)

Unless the user already chose via `project` or `issue` in the command:

> **Ship at project level or for a single issue?**
> - **Issue** (recommended) — one PR closing the issue on this issue branch. Smaller diff, its own CI run, far easier to review.
> - **Project** — one PR closing all issues worked on this epic branch. Use only when the issues are inseparable — a multi-issue squashed PR is the hardest artifact to review.

Wait for the answer before pushing, creating a PR, or closing issues.

**Skip the question** when:
- The command includes `project` or `issue`
- The current branch clearly indicates mode: `<epic-slug>-YYYY-MM-DD` → project; `PROJ-123-*` → issue

---

## Work Type Detection

- Epic branch or `PROJ-123-*` issue branch with git repo → code work
- No matching branch → non-code work
- If ambiguous, ask

---

## Quality gate (both code paths — mandatory before any push)

Follow the **ship-gate** skill in **interactive** mode after the work summary and before pushing. Compute `<base>` once (origin HEAD branch, else `main`) and reuse it. Do not push until G1–G3 pass. Their results feed the PR body.

---

## Path: Project — ship the epic

### Resolve the epic

1. From branch: map `<epic-slug>-YYYY-MM-DD` via `jira epic list --plain`
2. If epic key provided, use it
3. Otherwise ask which epic this branch completes

### 1. Detect base branch

Follow the **ship-gate** skill's base-branch rule. Compute `<base>` once here and reuse it for every step below — do not re-derive it later in this run.

### 2. Gather epic issues

```bash
jira issue view <epic-key>
jira issue list --epics <epic-key> -s"In Progress" --plain
```

Union In Progress children with `[A-Z]+-[0-9]+` from commits.

### 3. Show work summary

Show `git log --oneline <base>..HEAD` and `git diff --stat <base>..HEAD`. List every issue key in this push. Reuse this output later in this run instead of re-running it.

### 4. Run the quality gate

Follow the **ship-gate** skill in **interactive** mode. All three checks must pass before anything is pushed.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create one project PR

Title: `<Epic summary>: <short summary>`

Body — one `Closes PROJ-ID` per issue, and a test plan filled with the **real commands and results from G2** (never an unchecked checkbox):

```bash
gh pr create --title "Auth System: Caching fixes" --body "$(cat <<'EOF'
## Summary

- What changed across the epic

## Issues closed

Closes PROJ-10
Closes PROJ-12

## Test plan

- `npm test` — 42 passed, 0 failed
- `npm run build` — clean

## Known issues

- (Unresolved suggestions or scope observations from G3 not fixed in this PR, if any)
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
git checkout <base> && git pull
```

### 10. Complete the epic

- Unresolved children remain → list them; offer `/next project`
- None remain → `jira issue move <epic-key> "Done"` (or workflow epic-done transition)

### Project code rules

- One PR per epic branch from `/done project`
- Without GitHub-Jira integration: `jira issue move <id> "Done"` per issue after merge
- No commits → skip PR and note it

---

## Path: Issue — ship a single issue

### 1. Detect the issue

From branch (`PROJ-123-*`) or provided key. Fallback: ask which issue.

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
gh pr create --title "PROJ-12: Issue summary" --body "$(cat <<'EOF'
## Summary

- What changed and why

## Test plan

- `npm test` — 42 passed, 0 failed
- `npm run build` — clean

Closes PROJ-12
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
git checkout <base> && git pull
```

### Issue code rules

- PR title and body must include issue key; `Closes PROJ-42` for auto-transition
- Without integration: `jira issue move PROJ-42 "Done"` after merge
- Smart Commits: `PROJ-42 #done #comment …` if DVCS connector is enabled
- No commits → skip PR and note it
- Worktree: show cleanup commands (`git worktree remove <path>`) after PR creation

### If push is rejected (both code paths)

Follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, stop, list files, re-run `/done`. Never force-push.

---

## Path B: Non-Code Work

**Project:** summarize epic deliverables, artifact links via `jira issue comment add`, `jira issue move` Done per child, epic Done if complete.

**Issue:** summarize, artifact comment, `jira issue move <key> "Done"`.

Offer `/next` with matching scope.

---

## Notes

- Scope question comes **before** git operations
- `/done project` pairs with `/next project`; `/done issue` with `/next issue`
- Check branch pattern when work type is unclear
- The quality gate (G1–G3) is prompt-level discipline — make it unbypassable with branch protection on `main` (required checks + 1 approving review). See the github-cli skill's "Branch protection" section.
