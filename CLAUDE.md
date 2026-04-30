# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Two parallel sets of workflow automation — one for Claude Code + Linear, one for Codex + Jira — that implement the same development loop: pick issue → branch → implement → PR → merge → repeat.

| Variant | Directory | Issue tracker | Commands location |
|---------|-----------|--------------|-------------------|
| Claude + Linear | `claude/` | Linear (via MCP + CLI) | `claude/commands/` |
| Codex + Jira | `codex/` | Jira (via CLI) | `codex/instructions/` |

## Structure

```
claude/
  commands/           # Slash command specs (.md prompt files loaded by Claude Code)
  skills/             # Reusable skill modules referenced from commands
    linear-cli/       # Linear CLI reference and best practices
    github-cli/       # GitHub CLI reference (gh)
    product-planning/ # Product thinking methodology
    diagnostic/       # Structured debugging and diagnosis protocol
    sit/              # Stop, Inspect, Think — mid-task self-audit
    bugs/             # Bug scanning methodology
    debt/             # Tech debt scanning methodology
    code-review/      # PR review methodology
    (+ more)

codex/
  instructions/       # Codex instruction files (equivalent of claude/commands/)
  skills/             # Same skill modules, adapted for Jira instead of Linear
    jira-cli/         # Jira CLI reference (instead of linear-cli)
    (rest mirrors claude/skills/)
```

## How commands and skills work

**Commands** (`claude/commands/*.md`, `codex/instructions/*.md`) are prompt files — they are instructions written *to* Claude/Codex, not documentation for humans. They are prescriptive: exact CLI commands, exact MCP tool names, exact output formats. Each command is self-contained.

**Skills** (`*/skills/*/SKILL.md`) are reusable modules. A command invokes a skill by name: e.g. `Follow the diagnostic-thinking skill.` Skills have YAML frontmatter:

```yaml
---
name: skill-name
description: When to invoke this skill (used by the AI to decide relevance)
allowed-tools: Bash(linear:*), Bash(gh:*)  # optional tool restrictions
---
```

## MCP tool names (Claude + Linear only)

Commands reference Linear MCP tools by name. The README setup registers the server as:

```bash
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```

This produces the prefix `mcp__claude_ai_Linear__` (e.g. `mcp__claude_ai_Linear__list_issues`). If the server was registered under a different name, all references in `claude/commands/` and `claude/skills/` must be updated to match.

## Key differences between the two variants

- `claude/` commands use both MCP tools (`mcp__claude_ai_Linear__*`) and the `linear` CLI for issue management
- `codex/` instructions use only the `jira` CLI — no MCP
- `claude/` commands reference `gh` for GitHub; `codex/` does too
- Skills are largely identical between variants; the only structural difference is `linear-cli` (claude) vs `jira-cli` (codex)

## Working in this repo

- Write commands as instructions to the AI, not as human documentation
- Keep each command self-contained — no runtime dependencies on other commands
- When adding a command to one variant, add the equivalent to the other
- Skills are shared methodology; keep them tool-agnostic where possible, or fork with a variant-specific note

## Testing changes

No test suite. Validate by installing commands in a test project and running the slash command in Claude Code:

```bash
cp claude/commands/* /path/to/project/.claude/commands/
cp -r claude/skills/* /path/to/project/.claude/skills/
```
