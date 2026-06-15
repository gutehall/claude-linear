---
name: dedupe
description: Scan a Linear workspace for duplicate and near-duplicate issues, cluster them, and walk the user through merging or canceling them. Use this skill whenever the user runs /dedupe or asks to find duplicate issues, similar issues, redundant tickets, or clean up an overlapping backlog in Linear.
---

# Duplicate Detection

Find issues that describe the same work, group them into clusters, pick a canonical issue per cluster, and let the user decide how to collapse each group. Never merge or cancel anything without confirmation.

## Mindset

Two issues are duplicates when acting on one makes the other redundant — same bug, same feature, same fix. Don't flag issues that merely share a topic. The test: if you closed one, would the other still represent real outstanding work? If yes, they're related, not duplicate.

False positives waste the user's time and risk losing tracked work. Bias toward precision — when unsure, label a pair "possible" rather than "duplicate" and let the user judge.

## Loading the workspace

Fetch in parallel:

1. `mcp__claude_ai_Linear__list_teams` — active team
2. `mcp__claude_ai_Linear__list_issues` — all issues to compare

By default compare **open work**: Backlog, Todo, In Progress. Exclude Done and Canceled — they're already resolved and re-opening that history is rarely what the user wants. Honor flags that widen the set:

| Flag | Set compared |
|------|--------------|
| (none) | Backlog, Todo, In Progress |
| `--all` | every status including Done/Canceled |
| `--project "<name>"` | only issues in that project |
| `--label <name>` | only issues carrying that label |

If the issue list is large (>200), tell the user the count and that comparison is title/description-based, then proceed.

## Detecting duplicates

Compare issues pairwise. Signals, strongest first:

| Signal | Weight |
|--------|--------|
| Near-identical title (same words, reordered or reworded) | High |
| Description describes the same symptom, file, or feature | High |
| Same error message, stack trace, or reproduction steps | High |
| Same labels + same area/component + overlapping title terms | Medium |
| One issue's title is a subset of another ("Fix login" vs "Fix login redirect bug") | Medium — may be parent/child, not duplicate |
| Shared topic only, different specifics | Low — usually NOT a duplicate |

Read titles and descriptions semantically — "App crashes on logout" and "Logout throws null pointer" are the same bug despite no shared keywords. Conversely, "Add dark mode" and "Fix dark mode toggle" are different work.

Classify each pair/cluster by confidence:

- **Duplicate** — clearly the same work. Safe to recommend collapsing.
- **Possible** — overlapping, needs a human call. Present, don't recommend.
- **Related** — same area but distinct work. Suggest linking, not merging.

## Picking the canonical issue

Within a duplicate cluster, the canonical is the one to keep. Prefer, in order:

1. Most complete — has a real description, acceptance criteria, estimate
2. Most activity — comments, assignee, in-progress status, linked PR/branch
3. Oldest — the original, if the above tie

Everything else in the cluster collapses into the canonical.

## Presenting clusters

Group findings by confidence, most actionable first. Be specific — IDs and titles, not counts alone:

```
Duplicates (2 clusters)

  Cluster A — keep FIN-12
    FIN-12  "Login redirect loops on expired session"   (has description, assignee, in progress)
    FIN-47  "Infinite redirect when token expired"       (no description, backlog)
    → same bug. Recommend: cancel FIN-47, link to FIN-12.

  Cluster B — keep FIN-3
    FIN-3   "Add CSV export to billing"
    FIN-29  "Export invoices as CSV"
    → same feature. Recommend: merge FIN-29 into FIN-3.

Possible (1 cluster) — needs your call
    FIN-8   "Dashboard slow to load"
    FIN-31  "Optimize dashboard queries"
    → FIN-31 may be the fix for FIN-8, or a separate task.

Related (linking only)
    FIN-5 / FIN-19 — both touch auth, different work. Offer to link as related.
```

## Acting on a cluster

Linear has no merge or delete via API or CLI — true merge (moving comments, sub-issues, relations) happens only in the Linear UI. So offer these concrete actions per cluster and apply only the confirmed one:

**1. Collapse (cancel duplicate, keep canonical)** — the default for clear duplicates:
- Copy any context the duplicate has that the canonical lacks into the canonical:
  - `mcp__claude_ai_Linear__save_issue` to append to the canonical's description, or
  - `mcp__claude_ai_Linear__save_comment` to add the duplicate's useful notes
- Comment on the duplicate linking the canonical: `mcp__claude_ai_Linear__save_comment` → "Duplicate of FIN-12. Collapsed via /dedupe."
- Cancel the duplicate: `linear issue update FIN-47 --status "Canceled"`

**2. True merge in Linear UI** — when the duplicate has comments, sub-issues, or relations worth preserving:
- Don't cancel. Tell the user to merge in Linear: open the duplicate, "⋯ → Merge into issue", target the canonical.
- Provide both issue identifiers so they can act fast.

**3. Link as related** — for "related", not duplicate:
- Add a cross-referencing comment on each via `mcp__claude_ai_Linear__save_comment`.

**4. Skip** — leave the cluster untouched.

Present per cluster:

```
Cluster A — keep FIN-12, collapse FIN-47
[Enter] Collapse (cancel FIN-47, link to FIN-12)
[m] Merge in UI instead   [l] Link as related   [s] Skip   [q] Quit
```

## Output

After all clusters:

```
Reviewed N clusters. Collapsed M, merged-in-UI K, linked L, skipped S.
```

List each collapsed duplicate (`FIN-47 → FIN-12`) and any cluster left for manual UI merge.

## Error handling

- `save_issue` / `save_comment` / status update fails: print the error, offer retry or skip — never silently continue, never leave a duplicate canceled without its cross-link comment.
- MCP unreachable: "Could not reach Linear. Check your MCP connection with `/mcp`." and stop.
- Canonical pick is ambiguous (true tie): present the cluster and ask the user which to keep before acting.

## Rules

- Never cancel or modify an issue without explicit confirmation for that cluster.
- Always preserve context — copy useful description/comments into the canonical before canceling a duplicate.
- Never use delete; Linear has no API delete and cancellation is reversible. Cancellation is the strongest action this skill takes.
- When a duplicate carries comments, sub-issues, or relations, prefer UI merge over collapse so nothing is orphaned.
- Bias toward "possible" over "duplicate" when uncertain — let the human decide.

## Related skills

- **linear-cli** — issue status updates and CLI reference
- **triage** — clean up metadata on the surviving issues after deduping
- **scope** — broader project audit for gaps and quality
