---
name: dedupe
description: Scan a Jira project for duplicate and near-duplicate issues, cluster them, and walk the user through linking and closing them. Use this skill whenever the user runs /dedupe or asks to find duplicate issues, similar issues, redundant tickets, or clean up an overlapping backlog in Jira.
---

# Duplicate Detection

Find issues that describe the same work, group them into clusters, pick a canonical issue per cluster, and let the user decide how to collapse each group. Never link or close anything without confirmation.

## Mindset

Two issues are duplicates when acting on one makes the other redundant — same bug, same feature, same fix. Don't flag issues that merely share a topic. The test: if you closed one, would the other still represent real outstanding work? If yes, they're related, not duplicate.

False positives waste the user's time and risk losing tracked work. Bias toward precision — when unsure, label a pair "possible" rather than "duplicate" and let the user judge.

## Loading the project

Use the Atlassian MCP for rich reads and `jira` CLI for writes:

1. The Atlassian MCP's issue-search tool (check your available `mcp__claude_ai_Atlassian__*` tools for the exact name) to pull the issue set (or `jira issue list`)
2. Read titles, descriptions, and where useful, comments

By default compare **open work** (not yet Done/Closed). Honor flags that widen the set:

| Flag | Set compared |
|------|--------------|
| (none) | open statuses (To Do, In Progress, etc.) |
| `--all` | every status including Done/Closed |
| `--project "<KEY>"` | only issues in that project |
| `--label <name>` | only issues carrying that label |

If the set is large (>200), tell the user the count and that comparison is title/description-based, then proceed.

## Detecting duplicates

Compare issues pairwise. Signals, strongest first:

| Signal | Weight |
|--------|--------|
| Near-identical summary (same words, reordered or reworded) | High |
| Description describes the same symptom, file, or feature | High |
| Same error message, stack trace, or reproduction steps | High |
| Same labels + same component + overlapping summary terms | Medium |
| One summary is a subset of another ("Fix login" vs "Fix login redirect bug") | Medium — may be epic/story, not duplicate |
| Shared topic only, different specifics | Low — usually NOT a duplicate |

Read summaries and descriptions semantically — "App crashes on logout" and "Logout throws null pointer" are the same bug despite no shared keywords. Conversely, "Add dark mode" and "Fix dark mode toggle" are different work.

Classify each pair/cluster by confidence:

- **Duplicate** — clearly the same work. Safe to recommend collapsing.
- **Possible** — overlapping, needs a human call. Present, don't recommend.
- **Related** — same area but distinct work. Suggest linking, not collapsing.

## Picking the canonical issue

Within a duplicate cluster, the canonical is the one to keep. Prefer, in order:

1. Most complete — real description, acceptance criteria, story points
2. Most activity — comments, assignee, in-progress status, linked PR/branch
3. Oldest — the original, if the above tie

Everything else in the cluster collapses into the canonical.

## Presenting clusters

Group findings by confidence, most actionable first. Be specific — keys and summaries, not counts alone:

```
Duplicates (2 clusters)

  Cluster A — keep PROJ-12
    PROJ-12  "Login redirect loops on expired session"   (has description, assignee, in progress)
    PROJ-47  "Infinite redirect when token expired"        (no description, backlog)
    → same bug. Recommend: close PROJ-47 as duplicate, link to PROJ-12.

  Cluster B — keep PROJ-3
    PROJ-3   "Add CSV export to billing"
    PROJ-29  "Export invoices as CSV"
    → same feature. Recommend: collapse PROJ-29 into PROJ-3.

Possible (1 cluster) — needs your call
    PROJ-8   "Dashboard slow to load"
    PROJ-31  "Optimize dashboard queries"
    → PROJ-31 may be the fix for PROJ-8, or a separate task.

Related (linking only)
    PROJ-5 / PROJ-19 — both touch auth, different work. Offer to link as related.
```

## Acting on a cluster

Jira has a native **Duplicate** link type and transitions, so collapsing is clean. Offer these actions per cluster and apply only the confirmed one:

**1. Collapse (link + close duplicate, keep canonical)** — the default for clear duplicates:
- Copy any context the duplicate has that the canonical lacks: `jira issue comment add PROJ-12 "..."` or edit the canonical's description.
- Link the duplicate to the canonical: `jira issue link PROJ-47 PROJ-12 "Duplicate"`
- Comment on the duplicate: `jira issue comment add PROJ-47 "Duplicate of PROJ-12. Collapsed via /dedupe."`
- Close the duplicate: `jira issue move PROJ-47 "Done"` (or your workflow's "Won't Do"/"Closed" status; set resolution Duplicate if the workflow supports it).

**2. Link only (related, not duplicate):**
- `jira issue link PROJ-5 PROJ-19 "relates to"` — keep both open.

**3. Skip** — leave the cluster untouched.

Avoid `jira issue delete` — it is destructive and irreversible. Closing as duplicate preserves history and the audit trail.

Present per cluster:

```
Cluster A — keep PROJ-12, collapse PROJ-47
[Enter] Collapse (link + close PROJ-47 as duplicate of PROJ-12)
[l] Link as related only   [s] Skip   [q] Quit
```

## Output

After all clusters:

```
Reviewed N clusters. Collapsed M, linked L, skipped S.
```

List each collapse as `PROJ-47 → PROJ-12`.

## Error handling

- `jira` link/comment/move fails: print the error, offer retry or skip — never silently continue, never close a duplicate without its link + cross-link comment first.
- Transition name invalid: run `jira issue move PROJ-47 --help` to find the valid target status for the workflow, then retry.
- MCP/CLI unreachable: report it and stop.
- Canonical pick is ambiguous (true tie): present the cluster and ask which to keep before acting.

## Rules

- Never link, close, or modify an issue without explicit confirmation for that cluster.
- Always preserve context — copy useful description/comments into the canonical before closing a duplicate.
- Never use `jira issue delete`. Closing as duplicate is the strongest action this skill takes, and it's reversible.
- Always create the Duplicate link before closing, so the closed issue points to its survivor.
- Bias toward "possible" over "duplicate" when uncertain — let the human decide.

## Related skills

- **jira-cli** — issue transitions, linking, and CLI reference
- **triage** — clean up metadata on the surviving issues after deduping
- **scope** — broader project audit for gaps and quality
