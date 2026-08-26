# /debt - Scan the codebase for tech debt and create Jira issues

## Usage

```
/debt              # Full codebase scan
/debt <path>       # Scan a specific directory or file
```

## Flow

Follow the **debt** skill for the full scan, prioritization, and issue-creation procedure. Scope the scan to `$ARGUMENTS` if a path was given, otherwise scan the full codebase.

## Error handling

- If `jira project list` fails: "Could not reach Jira. Is `jira` configured? Run `jira init`." and stop.
- If issue creation fails: report the error, do not silently skip. Offer to retry.
- If the codebase is empty or has no source files: say so and stop.
