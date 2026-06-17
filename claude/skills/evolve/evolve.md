# Evolve — Full Protocol

The full protocol for planning the next version of an existing solution. Code-anchored, incremental, migration-aware.

## When to use

The user has something that works and wants the next version of it. They may know the goal ("make it multi-tenant") or just feel the current version has hit a limit. Either way: read the code, understand intent, propose grounded improvements, converge on a brief.

Do **not** use this for greenfield ideas (use `/think`), long-horizon strategy (use `/vision`), or ticket creation (use `/plan`).

---

## Phase 1 — Ground the goal

Before reading anything, know what the user is trying to achieve.

Ask, conversationally (skip any the input already answers):

- **What should the next version do that this one doesn't?** The capability or quality gap.
- **What's forcing it now?** Scale, a new use case, pain, a deadline, tech debt blocking progress.
- **What does "better" mean here, and for whom?** Faster? Cheaper to run? Easier to extend? For users, operators, or the team?

Stop at one or two exchanges. The goal is direction, not a spec.

## Phase 2 — Read the current solution

This is the phase that makes `/evolve` different. Read before proposing.

- Read `README.md`, `product.md`, architecture/design docs.
- Map the structure: entry points, core modules, data flow, key abstractions, external dependencies.
- Read the code relevant to the goal — open the files. Don't infer from names.
- `git log --oneline -20` for recent direction and churn hotspots.
- Broad codebase? Dispatch the **Explore** agent across relevant areas in parallel, then read the load-bearing files yourself.

Then state back, honestly:

- What the current solution does and how it's built.
- What's **load-bearing** — the parts everything depends on.
- Where the limits are relative to the goal — and *why* they exist (often a past decision that made sense then).

If the code contradicts how the user described it, say so now. That gap is often the real insight.

## Phase 3 — Generate improvements

Anchored to real code. Each improvement names the file/module it touches.

For the goal, explore:

- **What blocks the goal today?** The specific abstraction, schema, or coupling in the way.
- **What's the cleanest change that unblocks it?** Prefer extending existing seams over new layers.
- **What gets simpler?** The best changes remove things. Look for deletion, not just addition.
- **What's adjacent and cheap?** Improvements that ride along once you're already in that code.

Reject generic advice ("add caching", "use microservices") unless the code shows it's warranted. Every proposal must point at a real thing.

## Phase 4 — Pressure-test each proposal

For every candidate change:

- **Value vs. cost.** What does it buy, what does it take to build?
- **Blast radius.** What else touches this? What breaks?
- **What it locks in.** Does it make a future change easier or harder?
- **Reversibility.** Can it be undone if it's wrong?

Kill proposals that don't survive. Surface tensions and tradeoffs the user may not see. Disagree when evidence supports it.

## Phase 5 — Sequence and find the smallest valuable version

- Order surviving changes by value and dependency.
- Identify the **smallest valuable version** — the minimum slice that delivers real value and could ship on its own. This is the first thing to build, not a throwaway.
- Separate the endgame (full ambition) from the first increment.

If the only path the user sees is a rewrite, push hard: prove the increment can't work before accepting a big-bang. Rewrites discard working knowledge baked into the current code.

## Phase 6 — Migration reality

How do you get from current to next without breaking what works?

- What can run side-by-side? What needs a cutover?
- What data/schema migration is implied?
- What must keep working throughout?
- Feature flags, dual-write, expand-contract, strangler — name the strategy.

A next-version plan with no migration path is incomplete.

---

## Output — the brief

```markdown
## Next version: [name / scope]

### Goal
[What the next version achieves and why now]

### Current state
[What exists, what's load-bearing, where limits are and why — anchored to files]

### Proposed changes
[Each: what, why, rough cost, blast radius. Ordered by value.]

### Smallest valuable version
[Minimum slice delivering real value — the first thing to build]

### Migration path
[Current → next, incrementally. Strategy named. What can't break.]

### Out of scope
[What this version does not do, and why]

### Open questions
[What /plan must still decide]
```

End with:

> "Ready to turn this into work? Run `/plan` and share this brief, or I can start it for you."

---

## Rules

- Read real code before proposing — no unanchored improvements.
- No Linear writes. Thinking, not execution.
- Default incremental; justify any rewrite against the increment.
- Name the cost and blast radius of every proposal.
- Always produce a smallest-valuable-version, not just the endgame.
- Migration is part of the plan.
- Short exchanges. Converge through conversation.
- The brief is the deliverable.
