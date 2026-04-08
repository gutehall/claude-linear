# /review - Review open pull requests

## Usage

```
/review           # List open PRs and select one to review
/review <PR#>     # Review a specific PR directly
```

## Flow

### 1. Fetch open PRs

```bash
gh pr list --state open --json number,title,author,headRefName,createdAt,isDraft
```

- Filter out draft PRs by default (include if user asks)
- Display as a numbered list: #, title, author, branch, age
- If no open PRs: say so and offer `/next` to continue working

### 2. User selects a PR

By number from the list, or via `/review <PR#>` directly.

### 3. Review the PR

Run in order:

```bash
gh pr view <number>        # Description, reviewers, CI summary
gh pr checks <number>      # CI check details
gh pr diff <number>        # Full diff
```

### 4. Summarize findings

Report:
- **What changed**: files touched, scope of change
- **CI status**: passing / failing (which checks)
- **Issues spotted**: correctness, missing tests, scope creep, security concerns
- **Questions**: anything unclear that the author should clarify

### 5. Offer actions

```
a) Approve              → !gh pr review <number> --approve
b) Request changes      → !gh pr review <number> --request-changes -b "reason"
c) Comment only         → !gh pr review <number> --comment -b "comment"
d) Check out locally    → !gh co <number>
e) Skip                 → continue
```

## Notes

- You cannot approve your own PR — GitHub will reject it. Skip the approve option when the PR author matches `git config user.email`
- If CI is failing, do not approve — note which checks failed and suggest fixes
- Focus review on: correctness, scope creep, missing tests, security
- For large diffs, summarize by file/area rather than line-by-line
