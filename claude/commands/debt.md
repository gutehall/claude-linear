# /debt - Scan the codebase for tech debt and create Linear issues

## Usage

```
/debt              # Full codebase scan
/debt <path>       # Scan a specific directory or file
```

## Flow

### 1. Setup: Read Linear state

- `mcp__plugin_linear_linear__list_teams` → pick the active team
- `mcp__plugin_linear_linear__list_issue_labels` → note available labels
- If a "tech-debt" label does not exist, create it with `mcp__plugin_linear_linear__create_issue_label`

### 2. Scan: Systematic codebase audit

Explore the full directory tree first (LS, Glob), then read every non-trivial source file. Don't skip files because they look tidy — debt hides in overlooked corners.

Scan for these categories in every file:

**Missing tests** → Medium or High
- Source files with no corresponding test file
- Functions or classes with no visible test coverage
- Critical code paths (auth, payments, data mutations) with zero test coverage → High

**Complexity** → Medium
- Functions over ~50 lines
- Deeply nested conditionals (3+ levels)
- Functions with 5 or more parameters

**Dead code** → Low or Medium
- Unused exports or functions never imported elsewhere
- Unreachable code blocks
- Large sections of commented-out code

**TODO/FIXME/HACK/XXX comments** → Low or Medium
- Surface all of them; judge priority by the comment content
- Use Medium if the comment indicates a real correctness or reliability problem

**Hardcoded configuration** → Medium
- URLs, ports, timeouts, or limits hardcoded in logic (not in config files)
- Values that belong in env vars or named constants

**Duplicated code** → Medium
- Similar functions or logic blocks that could be shared
- Copy-paste patterns across files

**Missing TypeScript / type safety** → Low
- `any` casts in TypeScript codebases
- Missing return types on exported functions
- Untyped parameters

**Outdated patterns** → Low or Medium
- Deprecated API usage within the codebase's own framework or libraries
- Old async patterns (callbacks where promises are the project standard)
- Inconsistent patterns across files (some use one approach, others a different one)

Scan everything. Depth matters more than speed here.

### 3. Prioritize all findings

After the full scan, rank every finding before creating any issues:

| Priority | Criteria |
|----------|----------|
| 1 Urgent | Hardcoded secret-adjacent config (credentials, tokens, keys) |
| 2 High   | Critical paths with zero test coverage, severe complexity making bugs likely |
| 3 Medium | Most debt: complexity, duplication, hardcoded config, missing types |
| 4 Low    | TODOs, dead code, style inconsistencies |

### 4. Create Linear issues — highest priority first

Create all Urgent issues, then High, then Medium, then Low in a new project. This ensures the backlog is sorted correctly on creation.

For each debt item:

**Title**: `[DebtType] Short description`
Example: `[Missing Tests] No test coverage for auth token validation`

**Description**:
```
## Issue
<what the debt is>

## Location
`path/to/file.ext:line` (or function/class name)

## Impact
<why this matters — what risks does it create>

## Suggested fix
<specific action to take>
```

**Priority**: 1 (Urgent), 2 (High), 3 (Medium), or 4 (Low)

**Labels**: `["tech-debt"]` — add any additional matching type labels that already exist in the workspace (e.g. "testing", "refactor"). Do not create extra labels for categories.

Use `mcp__plugin_linear_linear__save_issue` for each issue.

### 5. Report

When all issues are created, print a summary:

```
Found N debt items — X urgent, Y high, Z medium, W low.
Created N Linear issues.
```

List each created issue ID and title.

## Error handling

- If `list_teams` fails: "Could not reach Linear. Is the MCP server connected?" and stop.
- If issue creation fails: report the error, do not silently skip. Offer to retry.
- If the codebase is empty or has no source files: say so and stop.

## Rules

- Read every source file — do not sample or skip
- Never create an issue for a pattern that is not actually present in this codebase (no hypothetical debt)
- One issue per debt item, not one issue per file or category
- Keep descriptions concrete: file path, line reference, exact problem, specific fix
