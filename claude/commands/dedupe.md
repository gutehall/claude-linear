# /dedupe - Find duplicate and similar issues, then collapse them

Scan the Linear workspace for duplicate and near-duplicate issues, cluster them, and walk through merging or canceling each cluster. Nothing is changed without confirmation.

## Usage

```
/dedupe                      # Compare open work (Backlog, Todo, In Progress)
/dedupe --all                # Include Done and Canceled issues too
/dedupe --project "Phase 1"  # Only issues in one project
/dedupe --label bug          # Only issues carrying a label
```

## Flow

### 1. Load issues

Fetch in parallel:
- `mcp__claude_ai_Linear__list_teams` — active team
- `mcp__claude_ai_Linear__list_issues` — issues to compare

Filter to the set implied by the flag (default: Backlog, Todo, In Progress — exclude Done/Canceled).

Report: `Comparing N issues for duplicates.`

If <2 issues in scope: say "Not enough issues to compare." and stop.
If the MCP call fails: say "Could not reach Linear. Check your MCP connection with `/mcp`." and stop.

### 2. Detect and cluster

Analyze using the **dedupe skill** — follow it for detection signals, confidence levels, and how to pick the canonical issue per cluster.

### 3. Present clusters

Group by confidence (Duplicate → Possible → Related), most actionable first. Show issue IDs, titles, why they cluster, and the recommended canonical. Format per the dedupe skill.

If no clusters found: say "No duplicates found across N issues." and stop.

### 4. Act on each cluster — confirm before any change

For each cluster offer:

```
Cluster A — keep FIN-12, collapse FIN-47
[Enter] Collapse (cancel FIN-47, link to FIN-12)
[m] Merge in UI instead   [l] Link as related   [s] Skip   [q] Quit
```

- **Collapse:** copy any context the duplicate has into the canonical (`mcp__claude_ai_Linear__save_issue` / `mcp__claude_ai_Linear__save_comment`), add a "Duplicate of FIN-12" comment on the duplicate, then `linear issue update FIN-47 --status "Canceled"`.
- **Merge in UI:** don't cancel — give both identifiers and tell the user to use Linear's ⋯ → Merge into issue (preserves comments/sub-issues/relations).
- **Link:** add cross-referencing comments via `mcp__claude_ai_Linear__save_comment`.
- **Skip / Quit:** leave untouched / stop and summarize.

### 5. Summary

```
Reviewed N clusters. Collapsed M, merged-in-UI K, linked L, skipped S.
```

List each collapse as `FIN-47 → FIN-12` and any cluster left for manual UI merge.

## Notes

- Follow the dedupe skill for detection methodology and canonical selection.
- Linear has no API merge or delete — collapse = cancel + cross-link; true merge is UI-only. Cancellation is reversible; deletion is never used.
- Bias toward "possible" when unsure. Never cancel an issue without confirmation for that specific cluster.
- Always preserve context — never cancel a duplicate before copying its useful description/comments into the canonical.
