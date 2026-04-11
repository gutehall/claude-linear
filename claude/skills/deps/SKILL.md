---
name: deps
description: Audit project dependencies for security vulnerabilities and outdated packages, then create prioritized Linear issues. Use this skill whenever the user runs /deps, asks to audit dependencies, wants a security scan of packages, or wants dependency issues tracked in Linear.
---

# Dependency Auditor

Detect all package managers in the project, run security and outdated checks, and create one Linear issue per finding, ordered by severity.

## Setup

Before auditing:

1. `mcp__plugin_linear_linear__list_teams` — identify the active team
2. `mcp__plugin_linear_linear__list_issue_labels` — note available labels
3. If no "dependencies" label exists: `mcp__plugin_linear_linear__create_issue_label` to create one

## Package manager detection

Check for manifest files in the project root and subdirectories:

| Manifest | Package manager |
|----------|----------------|
| `package.json` + `pnpm-lock.yaml` | pnpm |
| `package.json` + `yarn.lock` | yarn |
| `package.json` (no lock file) | npm |
| `requirements.txt`, `pyproject.toml`, or `Pipfile` | pip/pip-audit |
| `Cargo.toml` | cargo |
| `go.mod` | go |
| `Gemfile` | bundler |

If no manifests found: "No package manifests found. Is this a new project?" and stop.

## Audit commands

### Security

| Package manager | Command |
|----------------|---------|
| npm | `npm audit --json` |
| yarn | `yarn audit --json` |
| pnpm | `pnpm audit --json` |
| pip | `pip-audit --format json` (fall back to `safety check --json`) |
| cargo | `cargo audit --json` |
| go | `govulncheck ./...` |
| bundler | `bundle audit --format json` |

### Outdated

| Package manager | Command |
|----------------|---------|
| npm | `npm outdated --json` |
| yarn | `yarn outdated --json` |
| pnpm | `pnpm outdated --json` |
| pip | `pip list --outdated --format json` |
| cargo | `cargo outdated` |
| go | `go list -m -u all` |
| bundler | `bundle outdated` |

If a tool is not installed: note it, skip that check, continue.
If a command exits non-zero with no parseable output: show raw error, skip, continue.

## Priority mapping

### Security vulnerabilities

| Severity | Priority |
|----------|----------|
| Critical | 1 Urgent |
| High | 2 High |
| Moderate / Medium | 3 Medium |
| Low / Info | 4 Low |

### Outdated packages (no CVE)

| Type | Priority |
|------|----------|
| Major version behind (e.g. 2.x → 3.x) | 3 Medium |
| Minor version behind (e.g. 2.1 → 2.5) | 4 Low |
| Patch only | skip — do not create a Linear issue |

## Creating Linear issues

Rank all findings first, then create issues in order: Urgent → High → Medium → Low.
Creating in priority order ensures the backlog is correctly sorted.

**Title for vulnerabilities:** `[Security] <package>@<current>: <CVE or brief description>`

**Title for outdated packages:** `[Dependencies] Update <package> from <current> to <latest>`

**Description template:**
```
## Issue
<vulnerability description OR "Package is N major versions behind">

## Package
`<package-name>` — current: `<version>`, latest: `<latest-version>`

## CVE / Advisory
<CVE ID and link if applicable, or "None">

## Impact
<what could go wrong — use CVE description for vulnerabilities, "compatibility risk / missing features" for outdated>

## Fix
<exact command to update, e.g. `npm install package@latest`>
```

**Labels:** `["dependencies"]` for all issues. Add `["security"]` for any issue with a CVE or advisory. Do not create extra labels.

Use `mcp__plugin_linear_linear__save_issue` for each issue.

## Output

When all issues are created:

```
Audited <N> package manager(s). Found X vulnerabilities, Y outdated packages.
Created N Linear issues.
```

List each created issue ID and title.

If everything is clean: "No vulnerabilities found. All packages are up to date (or within acceptable range)." — do not create any Linear issues.

## Rules

- Run every detected package manager — do not skip any
- One issue per vulnerable or outdated package
- Never create an issue for a patch-only version difference
- Keep descriptions concrete: exact package name, exact version, exact fix command

## Related skills

- **linear-cli** — full CLI reference for label and issue management
- **bugs** — for scanning the codebase itself for code-level bugs
