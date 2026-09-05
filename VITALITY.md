# Command & skill vitality

How essential each command/skill is to the default loop: `/next` → implement → `/done` on Linear or Jira, automation-first. Use this to decide what to keep installed, what to trim, and what is ceremony.

Three tiers:

- **Core** — the daily loop. Remove these and the workflow breaks.
- **Situational** — real value, used periodically (planning, audits, releases). Keep, don't trim.
- **Non-vital** — rarely used, overlapping, or ceremony. Candidates to cut from a global install.

Project-specific rules (verify commands, coding standards) never live here — they live in the consuming repo as an optional conventions skill or `CLAUDE.md` / `AGENTS.md`.

## Commands

| Command | Tier | Why |
|---------|------|-----|
| `/next` | Core | Entry point of the loop — pick + branch + start. |
| `/done` | Core | Ship: G1–G3 gate, PR, CI, merge. |
| `/pr` | Core | Open a PR for review without merging. |
| `/grind` | Core | Unattended one-cycle; the autonomous driver. |
| `/autopilot` | Core | Allowlisted unattended cycle (`auto-claude`). |
| `/plan` | Core | Creates the issues the loop consumes. |
| `/standup` | Core | Daily orientation from tracker + GitHub. |
| `/sync` | Core | Reconcile tracker with GitHub after merges/time off. |
| `/review` | Core | Review an open PR. |
| `/diagnose` | Situational | Root-cause a bug before touching code. |
| `/bugs` | Situational | Codebase bug scan → issues. Periodic. |
| `/debt` | Situational | Tech-debt scan → issues. Periodic. |
| `/deps` | Situational | Dependency/CVE audit → issues. Periodic. |
| `/release` | Situational | Changelog + GitHub release. Per release. |
| `/split` | Situational | Break a large issue into sub-issues. |
| `/scope` | Situational | Audit a project for gaps. |
| `/think` | Situational | Reason through a problem pre-planning. |
| `/issues` | Situational | Browse/filter issues. |
| `/sit` | Situational | Mid-task self-audit. On demand. |
| `/estimate` | Non-vital | Bulk t-shirt estimation — ceremony for a solo driver. |
| `/triage` | Non-vital | Backlog grooming — usually done at `/plan` time. |
| `/dedupe` | Non-vital | Duplicate hunting matters at team scale. |
| `/vision` | Non-vital | 1–3 year strategy. Rare. |
| `/evolve` | Non-vital | Next-version rethink. Rare. |
| `/whatchanged` | Non-vital | Management-facing progress report. |

## Skills

| Skill | Tier | Why |
|-------|------|-----|
| `ship-gate` | Core | Shared G1–G3 quality gate for every ship command. |
| `linear-cli` / `jira-cli` | Core | Backs tracker writes across the loop. |
| `github-cli` | Core | Backs PR/CI/merge and branch protection. |
| `code-review` | Core | `/review` and ship-gate G3 on L/XL diffs. |
| `prior-work` | Core | Stops re-implementing already-solved issues. |
| `product-planning` | Core | Backs `/plan`. |
| `diagnostic` | Situational | Backs `/diagnose`. |
| `bugs` / `debt` / `deps` | Situational | Back the matching scan commands. |
| `release` | Situational | Backs `/release`. |
| `scope` / `split` | Situational | Back the matching planning commands. |
| `sit` | Situational | Backs `/sit`. |
| `estimate` | Non-vital | Backs `/estimate`. |
| `triage` | Non-vital | Backs `/triage`. |
| `dedupe` | Non-vital | Backs `/dedupe`. |
| `vision` | Non-vital | Backs `/vision`. |
| `evolve` | Non-vital | Backs `/evolve`. |
| `whatchanged` | Non-vital | Backs `/whatchanged`. |

## If you trim

The default install copies **all commands** and **Core + Situational skills only**. Non-vital skill folders stay in this repo; copy any of them back into `.claude/skills/` (or `~/.claude/skills/`) to restore. Commands for those skills still work if you invoke `/estimate` etc. — Claude will look up the skill when named — but their `description:` frontmatter will not sit in every prompt's routing context.
