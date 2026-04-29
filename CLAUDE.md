# claude-linear

A set of Claude Code slash commands and skills for managing a Linear + GitHub development workflow without leaving Claude Code.

## What this is

Slash commands (`/next`, `/done`, `/plan`, `/standup`, `/issues`, `/review`, `/sync`, `/diagnose`, `/sit`) and supporting skills (`linear-cli`, `product-planning`, `diagnostic`, `sit`) that automate the full development loop.

## Structure

```
claude/
  commands/           # Slash command specs (Markdown prompt files)
  skills/
    linear-cli/       # Linear CLI reference and best practices
    product-planning/ # Product thinking methodology
    diagnostic/       # Structured debugging and diagnosis protocol
    sit/              # Stop, Inspect, Think — structured self-audit mid-task
```

## Working in this repo

When editing command or skill files:
- Commands are Claude prompts — write them as instructions to Claude, not documentation for humans
- Be prescriptive: specify exact CLI commands, exact MCP tool names, exact output formats
- Keep each command self-contained; avoid cross-command dependencies at runtime
- Skills are referenced from commands; a command may say "follow the product-planning skill"

## MCP tool names

Command and skill files reference Linear MCP tools by name. The names depend on how the MCP server was registered. The README setup uses:

```bash
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```

This produces tool names prefixed `mcp__claude_ai_Linear__` (e.g. `mcp__claude_ai_Linear__list_issues`). If you registered the server under a different name or as a local plugin, the prefix will differ — update all `mcp__claude_ai_Linear__` references in the command and skill files to match.

## Testing changes

There is no test suite. Validate by:
1. Installing the commands in a test project (`cp claude/commands/* /path/to/project/.claude/commands/`)
2. Running the slash command in Claude Code and verifying the behavior

## Installation

See README.md for full setup instructions.
