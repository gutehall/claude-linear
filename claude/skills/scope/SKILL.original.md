---
name: scope
description: How to audit a Linear project for gaps, unclear issues, and missing work. Used by /scope.
---

# Scope Audit

How to look at a project's issues and find what's missing, unclear, or out of order.

## Mindset

You're not padding the backlog — you're finding things that will surprise the team mid-sprint. The test for whether a gap is worth raising: would discovering it mid-implementation block progress or change the plan?

Ask before creating anything. Present findings; let the user decide what to act on.

## What to look for

### Unclear issues
An issue is unclear if someone picking it up tomorrow couldn't start without asking questions. Signs:
- Title is vague ("improvements", "refactor", "update X")
- No description, or description is just the title repeated
- No acceptance criteria — how do you know it's done?
- No estimate — is this an hour or a week?

### Scope gaps
Work that's implied by the project goals but not tracked anywhere. Sources:
- `product.md` mentions a feature or requirement with no corresponding issue
- An issue's acceptance criteria mentions a dependency that has no ticket
- A milestone has a logical step clearly missing (e.g., "design" and "deploy" but no "implement")
- Integration points mentioned in one issue that another team/system would need to handle

### Orphaned issues
Issues that don't belong to any milestone in a project that uses milestones. They tend to get forgotten. Either assign them to a milestone, or confirm they're intentionally unscheduled.

### Stale in-progress
Issues that have been "In Progress" for an unusually long time (relative to their estimate). May be blocked, abandoned, or just forgotten. Worth a conversation.

### Oversized issues
L/XL issues that haven't been split. These are planning liabilities — hard to schedule, easy to underestimate. Flag them with a suggestion to `/split`.

## How to assess gaps

Read `product.md` first. Without it, you're guessing at intended scope.

For each milestone or phase: trace through the work like a user journey or a data flow. What needs to happen for this milestone to deliver its stated goal? Is there a ticket for each step?

Don't invent requirements. If you're unsure whether something is missing or just out of scope, say so and let the user decide.

## Presenting findings

Group by category and be specific — include issue IDs, not just counts:

```
Unclear (3):
  FIN-4: "Fix auth" — no description, no acceptance criteria
  FIN-7: "Update dashboard" — unclear what "update" means
  FIN-12: no description at all

Unestimated (2): FIN-8, FIN-11

Orphaned (2): FIN-15, FIN-16 — not in any milestone

Potential gap:
  product.md §3 mentions email notifications for order status,
  but no issue covers this.

Oversized (1): FIN-2 (XL) — consider splitting

Stale in-progress (1): FIN-6 — In Progress for 14 days, estimated S
```

## Offering actions

For each finding, name the action but wait for confirmation:

- Unclear → "Want me to add a description/AC draft for FIN-4?"
- Unestimated → "Run `/estimate` to work through these?"
- Orphaned → "Assign FIN-15 and FIN-16 to a milestone?"
- Gap → "Worth creating an issue for email notifications? Tell me more and I'll draft it."
- Oversized → "Run `/split FIN-2`?"
- Stale → "Is FIN-6 still active? Should I check its status?"

## Related Skills

- **estimate** — for working through unestimated issues found during the audit
- **split** — for breaking down oversized issues
- **product-planning** — for adding new issues when genuine gaps are found
- **linear-cli** — for updating issues and assigning milestones
