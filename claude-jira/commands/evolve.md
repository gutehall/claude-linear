# /evolve - Plan the next version of an existing solution

A conversation about what the *next version* of a working solution should be. Read the actual code, understand what the user wants to achieve, propose concrete improvements grounded in what already exists, and converge on a brief you can hand straight to `/plan`.

## Usage

```
/evolve                          # Open-ended: what should the next version be?
/evolve "the API"                # Focus on a specific component or subsystem
/evolve "make it multi-tenant"   # Evolve toward a specific goal
/evolve "v2"                     # Plan a major-version rethink
```

## What this is

A grounded design conversation, not a strategy session and not a ticket dump.

- Unlike `/vision` — this is concrete and code-anchored, not 6mo–1yr horizons.
- Unlike `/think` — this *reads the codebase* first, so improvements are real, not hypothetical.
- Unlike `/plan` — no Jira writes. The output is a brief that feeds `/plan`.

This command uses the evolve skill. Follow it exactly.

---

## Flow

### 1. Understand what the user wants to achieve

If input was given, treat it as the goal. If not, ask:

> "What do you want the next version to do that the current one doesn't?"

Pin down the goal before reading code. One or two questions max — don't interrogate.

### 2. Read the current solution

Understand what exists before proposing changes. Anchor every improvement to real code.

- Read `README.md`, `product.md`, and any architecture/design docs in the repo
- Map the structure: entry points, core modules, data flow, key abstractions
- Read the code that's relevant to the user's goal — actually open the files, don't guess
- Check recent direction: `git log --oneline -20`
- For broad codebases, dispatch the **Explore** agent to map relevant areas in parallel, then read the critical files yourself

State back what the current solution does and where its limits are. If the codebase contradicts how the user described it, say so.

### 3. Invoke the evolve skill

> **Follow the evolve skill.**

The skill walks through:
- Grounding the goal: what "better" means here, and for whom
- Reading the current design honestly: strengths, limits, what's load-bearing
- Generating improvements anchored to real code — not generic best-practice
- Pressure-testing each: value vs. cost, what breaks, what it locks in
- Sequencing: what the smallest valuable next version is vs. the full ambition
- Migration reality: how to get from current to next without a big-bang rewrite

Disagree when the evidence supports it. A rewrite is rarely the answer — prove the increment can't work before reaching for one.

### 4. Synthesize

When there's enough clarity, produce the brief:

```markdown
## Next version: [name / scope]

### Goal
[What the next version achieves and why it's worth doing now]

### Current state
[What exists today, what's load-bearing, where the limits are — anchored to files]

### Proposed changes
[Each change: what, why, rough cost, what it affects. Ordered by value.]

### Smallest valuable version
[The minimum slice that delivers real value — the first thing to build]

### Migration path
[How to get from current to next incrementally. What can't break.]

### Out of scope
[What this version explicitly does not do, and why]

### Open questions
[Anything /plan must still decide]
```

### 5. Hand off

End with:

> "Ready to turn this into work? Run `/plan` and share this brief, or I can start it for you."

---

## Rules

- Read real code before proposing anything — no improvements that aren't anchored to a file
- No Jira writes — this is thinking, not execution
- Default to incremental evolution; justify any rewrite against the increment
- Name the cost of every proposal, not just the upside
- Always identify the smallest valuable version — don't only design the endgame
- Migration is part of the plan, not an afterthought
- Keep exchanges short — converge through conversation, not walls of text
- The brief is the deliverable, not the conversation
