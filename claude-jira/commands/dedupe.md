# /dedupe - Find duplicate and similar issues, then collapse them

Scan a Jira project for duplicate and near-duplicate issues, cluster them, and walk through linking and closing each cluster. Nothing is changed without confirmation.

## Usage

```
/dedupe                       # Compare open work
/dedupe --all                 # Include Done/Closed issues too
/dedupe --project "PROJ"      # Only issues in one project
/dedupe --label bug           # Only issues carrying a label
```

## Flow

### 1. Load issues

Use the Atlassian MCP (`mcp__claude_ai_Atlassian__` search) for rich reads, or `jira issue list`. Filter to the set implied by the flag (default: open statuses — exclude Done/Closed).

Report: `Comparing N issues for duplicates.`

If <2 issues in scope: say "Not enough issues to compare." and stop.
If the MCP/CLI call fails: report it and stop.

### 2. Detect and cluster

Analyze using the **dedupe skill** — follow it for detection signals, confidence levels, and how to pick the canonical issue per cluster.

### 3. Present clusters

Group by confidence (Duplicate → Possible → Related), most actionable first. Show keys, summaries, why they cluster, and the recommended canonical. Format per the dedupe skill.

If no clusters found: say "No duplicates found across N issues." and stop.

### 4. Act on each cluster — confirm before any change

For each cluster offer:

```
Cluster A — keep PROJ-12, collapse PROJ-47
[Enter] Collapse (link + close PROJ-47 as duplicate of PROJ-12)
[l] Link as related only   [s] Skip   [q] Quit
```

- **Collapse:** copy any context the duplicate has into the canonical (`jira issue comment add` / edit description), link `jira issue link PROJ-47 PROJ-12 "Duplicate"`, add a "Duplicate of PROJ-12" comment on the duplicate, then `jira issue move PROJ-47 "Done"` (or the workflow's Won't Do/Closed status).
- **Link:** `jira issue link PROJ-5 PROJ-19 "relates to"` — keep both open.
- **Skip / Quit:** leave untouched / stop and summarize.

Never use `jira issue delete` — closing as duplicate preserves history and is reversible.

### 5. Summary

```
Reviewed N clusters. Collapsed M, linked L, skipped S.
```

List each collapse as `PROJ-47 → PROJ-12`.

## Notes

- Follow the dedupe skill for detection methodology and canonical selection.
- Jira has a native Duplicate link type — always create the link before closing so the closed issue points to its survivor.
- Bias toward "possible" when unsure. Never close an issue without confirmation for that specific cluster.
- Always preserve context — never close a duplicate before copying its useful description/comments into the canonical.
