---
name: code-review
description: >
  Performs a thorough, structured code review that goes beyond surface-level linting — it
  validates that the code actually works as intended, finds logic errors, missing edge cases,
  broken contracts between components, and anything that would cause the code to silently fail
  or behave incorrectly. If issues are found, the skill produces a concrete fix plan and then
  executes it, followed by test runs to confirm the fix holds. Use this skill whenever the user
  asks to "review my code", "check if this works", "validate this function", "audit this module",
  "does this look right", "review before PR", or any time code correctness is in question. Also
  trigger when the user shares code and asks for feedback, wants a sanity check, or is about to
  merge/deploy and wants confidence. This skill should run proactively whenever Claude notices
  it's about to work on code that hasn't been reviewed and correctness matters.
---

# Code Review

**The goal**: Determine with confidence whether this code does what it's supposed to do — not just whether it looks clean, but whether it actually works correctly in all meaningful cases.

This is a four-phase protocol: **Understand → Audit → Plan & Fix → Verify**.

---

## Phase 1: Understand Intent

Before looking for problems, establish what "correct" means for this code.

**Gather:**
- What is this code supposed to do? (Read docstrings, comments, function names, call sites)
- What are the inputs and their valid ranges/types?
- What are the expected outputs or side effects?
- What invariants must hold? (e.g. "this list is always sorted", "this value is never null")
- What's the broader context? (What calls this? What does it feed into?)

If intent is unclear from the code itself, ask the user one targeted question before proceeding. Don't guess at intent — a "correct" review of the wrong spec is worthless.

---

## Phase 2: Audit — Does It Actually Work?

Go through the code systematically. For each function, module, or logical unit, check:

### Correctness
- [ ] Does the logic match the stated intent?
- [ ] Are there off-by-one errors? (loop bounds, slice indices, pagination)
- [ ] Are comparisons correct? (`>` vs `>=`, `===` vs `==`, wrong type comparisons)
- [ ] Is mutable state modified correctly? (mutation when copy was intended, or vice versa)
- [ ] Are async operations awaited where needed? Any unhandled promise rejections?
- [ ] Are return values used correctly by callers?

### Edge Cases
- [ ] Empty inputs (empty array, empty string, zero, null/undefined)
- [ ] Single-element inputs
- [ ] Maximum/minimum values
- [ ] Concurrent access (if relevant)
- [ ] First/last items in collections

### Error Handling
- [ ] Are errors caught at the right level?
- [ ] Do errors propagate in a way callers can act on?
- [ ] Are there silent failures? (catching and swallowing exceptions, returning null instead of throwing)
- [ ] Is cleanup done correctly on error paths?

### Data Flow
- [ ] Is data transformed correctly at each step?
- [ ] Are there implicit type coercions that could go wrong?
- [ ] Are nullable/optional values checked before use?
- [ ] Is the correct data passed between functions (not stale, not mutated unexpectedly)?

### Dependencies and Contracts
- [ ] Does this code rely on behavior from external functions/APIs that may not hold?
- [ ] Are there implicit assumptions about the environment (env vars, file paths, network state)?
- [ ] Do interfaces between components match? (function signatures, API shapes, event payloads)

### Tests (if present)
- [ ] Do existing tests actually cover the main logic paths?
- [ ] Are there tests for failure cases and edge cases?
- [ ] Do tests actually assert the right things, or just that nothing throws?
- [ ] Could a test pass even if the code is wrong?

---

## Phase 3: Report and Fix Plan

### If no issues found

State clearly: "This code appears correct. Here's what I verified: [brief summary of what was checked]." Point out anything worth noting even if not a bug (e.g., fragile patterns, missing tests for edge cases).

### If issues are found

For each issue, document:

```
Issue #N: [Short title]
Severity: Critical | High | Medium | Low
Location: [file:line or function name]
Problem: [What is wrong — specific and mechanical, not vague]
Why it matters: [What goes wrong at runtime when this triggers]
Reproduction: [Minimal input/state that demonstrates the bug]
Fix: [What change resolves the root cause]
```

Severity guide:
- **Critical** — Causes data loss, security vulnerability, crash in normal use, or silently returns wrong results
- **High** — Causes incorrect behavior in a common case or edge case likely to be hit in production
- **Medium** — Incorrect in uncommon cases, or code that could break with minor changes to surrounding code
- **Low** — Doesn't cause bugs now but is fragile, misleading, or will cause bugs later

After listing all issues, produce a **Fix Plan** ordered by priority:
1. Issues to fix immediately (Critical/High)
2. Issues to fix before merge (Medium)
3. Issues to note/defer (Low)

Ask the user: "Should I proceed with the fixes?" — unless the user already said to fix everything, in which case proceed directly.

---

## Phase 4: Apply Fixes

Fix one issue at a time. For each fix:

1. **State what you're changing and why** (one sentence)
2. **Make the minimum change** that resolves the root cause
3. **Don't refactor unrelated things** while fixing — stay focused
4. **After each fix**, briefly confirm: "Fixed: [what was changed]"

When all fixes are applied, move to verification.

---

## Phase 5: Verify

After fixing, validate that the fixes work and nothing broke.

### Run existing tests first
```bash
# Run whatever test command is appropriate for this project:
# npm test / pytest / go test ./... / cargo test / etc.
```

If tests pass: good. If tests fail: apply the diagnostic-thinking skill before touching anything else.

### Check fix coverage
For each Critical or High issue fixed:
- [ ] Is there an existing test that would catch a regression?
- [ ] If not, write a minimal test that demonstrates the fix works

Test structure:
```
Test name: [describes the scenario, not the implementation]
Setup: [minimal state to reproduce the original bug]
Action: [call the code]
Assert: [the correct output or behavior]
```

### Final run
Run the full test suite one more time after adding any new tests. Report:
- Tests passing: N
- Tests added: N  
- Issues resolved: N

---

## Anti-Patterns to Avoid

| What it looks like | Why it's a problem |
|---|---|
| Reviewing style instead of correctness | Doesn't tell you if the code works |
| "This looks fine to me" without checking edge cases | The most common bugs live in edge cases |
| Fixing symptoms instead of root causes | The bug comes back in a different form |
| Making multiple fixes in one change | You won't know which fix resolved which issue |
| Skipping verification | The fix may work for the wrong reason |
| Adding tests that only test the happy path | Gives false confidence |
| Not understanding intent before auditing | You can't tell if something is wrong without knowing what "right" looks like |

---

## `/review` Command

This skill ships with a `/review` slash command for direct invocation in Claude Code.

**Install:**
```bash
# Personal (all projects)
mkdir -p ~/.claude/commands
cp <skill-path>/commands/review.md ~/.claude/commands/review.md

# Or project-scoped
mkdir -p .claude/commands
cp <skill-path>/commands/review.md .claude/commands/review.md
```

**Usage:**
```
/review                        ← reviews whatever you're currently working on
/review src/auth/token.ts      ← reviews a specific file
/review the payment flow       ← reviews a feature area
/review before I open the PR   ← full review + fix + verify cycle
```

---

## Quick Reference Checklist

Before signing off on any review:

- [ ] Do I understand what this code is supposed to do?
- [ ] Have I checked edge cases: empty, null, single-item, boundary values?
- [ ] Have I checked the error paths, not just the happy path?
- [ ] Have I verified that data flows correctly between components?
- [ ] Have I checked that async operations are handled correctly?
- [ ] Do existing tests actually prove correctness, or just that nothing explodes?
- [ ] If I fixed something, did I verify it with a test run?
- [ ] Is every Critical/High issue either fixed or explicitly deferred with a reason?
