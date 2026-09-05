# /bugs - Scan the codebase for bugs and create Jira issues

> **Project conventions:** (1) If this repo has a skill whose description says it defines coding rules, verify/test commands, or pre-merge gates — invoke it. (2) Else if `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` exists at the repo root (then the nearest subdirectory you are editing) — follow those. (3) Else continue.

## Usage

```
/bugs              # Full codebase scan
/bugs <path>       # Scan a specific directory or file
/bugs --changed    # Only files changed since the last full /bugs scan
```

## Flow

Follow the **bugs** skill for the full scan, prioritization, and issue-creation procedure. Scope the scan to `$ARGUMENTS` if a path was given, otherwise scan the full codebase. Honor `--changed` as defined in the skill.

For a large tree, spawn several cheap locator sweeps in parallel (split by directory or category; prefer `cavecrew-investigator` if that type exists, else Task/Explore on Haiku) instead of reading everything on the main thread. Treat receipts as leads — read every cited span yourself. Do not re-spawn to "prove" a discarded lead.

## Error handling

- If `jira project list` fails: "Could not reach Jira. Is `jira` configured? Run `jira init`." and stop.
- If issue creation fails: report the error, do not silently skip. Offer to retry.
- If the codebase is empty or has no source files: say so and stop.
