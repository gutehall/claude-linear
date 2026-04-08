# claude-linear

A set of Claude Code slash commands and skills for managing a Linear + GitHub development workflow without leaving Claude Code.

## What this is

Slash commands (`/next`, `/done`, `/plan`, `/standup`, `/issues`, `/review`, `/sync`) and supporting skills (`linear-cli`, `product-planning`) that automate the full development loop.

## Structure

```
claude/
  commands/           # Slash command specs (Markdown prompt files)
  skills/
    linear-cli/       # Linear CLI reference and best practices
    product-planning/ # Product thinking methodology
```

## Working in this repo

When editing command or skill files:
- Commands are Claude prompts — write them as instructions to Claude, not documentation for humans
- Be prescriptive: specify exact CLI commands, exact MCP tool names, exact output formats
- Keep each command self-contained; avoid cross-command dependencies at runtime
- Skills are referenced from commands; a command may say "follow the product-planning skill"

## Testing changes

There is no test suite. Validate by:
1. Installing the commands in a test project (`cp claude/commands/* /path/to/project/.claude/commands/`)
2. Running the slash command in Claude Code and verifying the behavior

## Installation

See README.md for full setup instructions.
