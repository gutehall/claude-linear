# /deps - Audit dependencies for vulnerabilities and outdated packages, then create Jira issues

## Usage

```
/deps              # Full audit (security + outdated)
/deps --security   # Security vulnerabilities only
/deps --outdated   # Outdated packages only
```

## Flow

Follow the **deps** skill for the full detect → audit → prioritize → issue-creation procedure, honoring whichever flag from `$ARGUMENTS` was passed (default: full audit).

## Error handling

- If `jira project list` fails: "Could not reach Jira. Is `jira` configured? Run `jira init`." and stop.
- If an audit tool is not installed: note it, skip that check, continue.
- If a command exits non-zero with no parseable output: show raw error, skip that check, continue.
- If issue creation fails: report the error, do not silently skip. Offer to retry.
