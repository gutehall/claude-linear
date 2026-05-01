# Product

## Vision

A dual-variant AI workflow toolkit that implements the full development loop — pick issue → branch → implement → PR → merge → repeat — entirely inside the AI interface, for teams using either Claude+Linear or Codex+Jira.

## Problem

Developers using AI coding assistants constantly context-switch: open the issue tracker, copy context to the AI, switch to a terminal for git, open GitHub for the PR. There's no integrated loop that connects issue selection, implementation, and shipping into a single AI-driven workflow. This repo is that loop.

## Users

- **Claude Code + Linear developers** — teams using Linear for issue tracking who want their entire dev loop inside Claude Code with no browser switching
- **Codex + Jira developers** — teams using Jira who want the same loop with Codex and the Jira CLI
- **Teams standardizing AI-assisted development** — anyone installing the commands globally or per-project to enforce a consistent workflow

Primary concern: minimal friction between having an idea and shipping code. Users care that the loop is fast, deterministic, and doesn't require learning new abstractions.

## Architecture

**Two parallel variants, one loop:**

| Layer | Claude + Linear | Codex + Jira |
|-------|----------------|--------------|
| Issue tracking | Linear MCP + `linear` CLI | `jira` CLI only |
| Branch creation | `linear branch <id>` | `git checkout -b PROJ-12-slug` |
| PR creation | `gh pr create` | `gh pr create` |
| Directory | `claude/` | `codex/` |

**No runtime code.** Everything is markdown — prompt files read by the AI at invocation time. No build step, no dependencies, no servers.

**Commands** (`claude/commands/`, `codex/instructions/`) are self-contained prompt files. Each command is a complete specification — exact CLI commands, exact MCP tool names, exact output format. No runtime dependencies between commands.

**Skills** (`claude/skills/`, `codex/skills/`) are reusable methodology modules. A command invokes a skill by name (e.g. "follow the product-planning skill"). Skills have YAML frontmatter so the AI can route to them. Some skills have a companion file (e.g. `sit/sit.md`) for the detailed protocol; the `SKILL.md` frontmatter is the entry point.

**Key constraint:** MCP tools are available in Claude Code but not Codex, which is why the claude/ variant uses `mcp__claude_ai_Linear__*` for richer Linear integration while codex/ uses CLI only.

## Roadmap

No versioned roadmap. Goals in order of priority:

1. **Variant parity** — every command and skill that exists in `claude/` should have a functional equivalent in `codex/`, and vice versa
2. **Protocol completeness** — every SKILL.md that references a companion file should have that file present and populated
3. **Self-contained commands** — no command should depend on another command at runtime; each must work in isolation

## Decisions

| Decision | Rationale | Date |
|----------|-----------|------|
| Commands written as AI instructions, not human docs | Commands are loaded into AI context at invocation — they must be prescriptive, not descriptive | — |
| Self-contained commands (no cross-command dependencies) | Prevents fragile chains; any command can be used independently or installed selectively | — |
| MCP for Linear in claude/, CLI-only in codex/ | No MCP equivalent exists for Jira; keeping codex/ CLI-only avoids a two-tier complexity | — |
| `sit.md` as a separate file from `SKILL.md` in claude/skills/sit/ | Keeps `SKILL.md` as a routing/metadata layer; puts the full protocol in a readable companion file | 2026-05-01 |
| PR creation lives in `/done` and `/pr`, not `/plan` | `/plan` ends at issue creation; mixing planning and execution in one command conflates two distinct phases of work | 2026-05-01 |

## Deferred

| Item | Why deferred |
|------|-------------|
| MCP integration for Jira variant | No Jira MCP server equivalent to `mcp.linear.app` at time of writing |
| Automated testing of commands | Commands are AI prompt files — no unit test framework applies; validation is manual (install + run in a test project) |
| Versioning / changelog for the toolkit itself | Low priority; users pull latest from git rather than pinning versions |
