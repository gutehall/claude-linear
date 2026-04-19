# /scope - Audit a project's issues for gaps and quality

Review a Linear project to find missing work, unclear requirements, or under-specified issues. Use this before a milestone or sprint to make sure nothing is falling through the cracks.

## Usage

```
/scope                   # Audit the current default project
/scope "Phase 1"         # Audit a specific project
```

## Flow

1. **Load project state:**
   ```bash
   linear project show "<project>"   # Issues, milestones, progress
   linear roadmap                    # Context in the broader roadmap
   ```
   Also use `mcp__claude_ai_Linear__list_issues` to get full issue list with descriptions.

2. **Read context:** Load `product.md` if it exists — understand intended scope before judging gaps.

3. **Analyze using the scope skill** — follow it for what to look for and how to reason about gaps.

4. **Present findings** grouped by category:
   ```
   Unclear (3): FIN-4, FIN-7, FIN-12 — missing descriptions
   Unestimated (5): FIN-8, FIN-9, ...
   Orphaned (2): FIN-15, FIN-16 — not in any milestone
   Gaps: no issue covers user notification flow (mentioned in product.md §3)
   Oversized: FIN-2 (XL) — should be broken down
   ```

5. **For each finding, offer an action** — do not auto-create or auto-edit:
   - Unclear → offer to add description/AC inline
   - Unestimated → offer `/estimate` for the batch
   - Orphaned → offer to assign to a milestone: `linear issue update <id> --milestone "<m>"`
   - Gap → ask for details before creating: `mcp__claude_ai_Linear__save_issue`
   - Oversized → suggest `/split <id>`

## Notes

- Follow the scope skill for what to look for and how to present findings
- Read `product.md` before judging scope gaps — never speculate without product context
- Ask before creating any new issues — don't pad the backlog speculatively
