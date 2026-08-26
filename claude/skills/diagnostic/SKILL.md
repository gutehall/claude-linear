---
name: diagnostic-thinking
description: >
  Forces Claude to stop guessing and actually reason through what a problem could be before
  attempting a fix. Use this skill whenever Claude is debugging, troubleshooting, or asked to
  "fix" something — especially when the cause isn't immediately obvious, when a previous fix
  attempt didn't work, or when Claude would otherwise jump straight to editing files or running
  commands. Trigger this skill for any "it's not working", "why is X broken", "fix this error",
  "this keeps failing" type request. If Claude is about to guess-and-check, this skill should
  run instead. Also use when the user seems frustrated that Claude keeps trying things without
  understanding the root cause.
---

# Diagnostic Thinking

**The problem**: Claude's default mode when debugging is pattern-matching — it sees an error message, recalls a common fix, applies it, and hopes for the best. This feels fast but often wastes more time than it saves. It's the engineering equivalent of shaking the TV to fix the picture.

**The goal**: Before touching anything, Claude should build a real mental model of what's wrong and *why*, then act from that understanding.

---

## The Protocol

### Phase 1: Stop. Read Everything.

Before forming any hypothesis:

- Read the **full** error message — not just the last line
- Find where in the code/config the error originates, not just where it surfaces
- Check if there are **multiple** errors and understand their order (first error is often the real one)
- Note what **changed recently** if known (recent commits, installs, config edits)
- Check the environment: Node version, OS, env vars, file paths, permissions

Ask yourself: *"What system is actually failing, and at what point does it fail?"*

---

### Phase 2: Generate Real Hypotheses

Don't go with the first explanation that looks plausible. Generate **at least 3 candidate causes**, ranked by likelihood:

```
Hypothesis A (most likely): [specific mechanical explanation]
  Evidence for: ...
  Evidence against: ...

Hypothesis B: [alternative explanation]
  Evidence for: ...
  Evidence against: ...

Hypothesis C: [less likely but possible]
  Evidence for: ...
  Evidence against: ...
```

Good hypotheses are **mechanical and specific** — they describe *what is actually broken in the system*, not just "maybe it's a version issue" or "could be a config problem".

Bad hypothesis: "Maybe there's a dependency issue."
Good hypothesis: "Package X expects peer dependency Y@^2.0, but Y@3.1 is installed, which introduced a breaking API change in the `connect()` function signature."

---

### Phase 3: Design a Targeted Diagnostic

Before fixing anything, **confirm which hypothesis is correct**. Pick the fastest test that distinguishes between your top hypotheses:

- A log statement at the right place
- A minimal reproduction (strip everything until only the broken part remains)
- A direct inspection (`console.log`, `cat`, `env`, `which`, `ls -la`, `curl`, type-checking)
- Temporarily hardcoding a value to isolate a variable

Run the diagnostic. Let the output update your hypothesis ranking.

Don't skip this step even if you're "pretty sure" — being wrong here means applying the wrong fix, which creates new problems.

---

### Phase 4: Fix with Confidence, Not Hope

Only now, with a confirmed hypothesis, write the fix. The fix should:

- Address the **root cause**, not the symptom
- Be the **minimum change** required
- Not break anything adjacent (check side effects)

After applying the fix, **verify it worked** — run the thing, check the output, confirm the error is gone and nothing new appeared.

---

### Phase 5: Explain What You Found

Briefly state:
1. What was actually wrong (root cause)
2. Why it was wrong (what condition caused it)
3. What the fix does (why it resolves the root cause)

This is not just for the user — writing this out is how you confirm that you actually understood the problem.

---

## Anti-Patterns to Actively Avoid

| What it looks like | Why it's a problem |
|---|---|
| Trying the most common fix for an error message without reading context | Fixes a different instance of the same-looking error |
| Making multiple changes at once | You won't know which one worked (or caused a new problem) |
| "Let me try X and see if that helps" | You're now debugging by lottery |
| Fixing the line that threw the error | The error is often thrown far from where it was caused |
| Assuming the same fix as last time | Context changes; the same symptom can have different causes |
| Not verifying after fixing | The fix may work for the wrong reason, leaving a latent bug |

---

## When You're Stuck

If you've gone through this and still don't understand the problem:

1. **Say so clearly** — "I don't have enough information to diagnose this confidently. I need to see X."
2. **Ask a targeted question** — not "can you give me more context?" but "can you show me the output of `npm ls react` and the contents of your `package.json`?"
3. **Build a minimal reproduction** — if the real codebase is complex, strip it down until the error is isolated

It is always better to say "I need to understand this better before I can fix it" than to run three wrong fixes in a row.

---

## `/diagnose` Command

This skill is invoked by the `/diagnose` command (`claude/commands/diagnose.md`), installed the same way as the rest of this package — see the "Testing changes" section of the repo's CLAUDE.md.

**Usage:**
```
/diagnose TypeError: Cannot read properties of undefined (reading 'map')
/diagnose the build fails after I added the new env variable
/diagnose   ← (no args — Claude will ask what's broken)
```

The command forces the full 5-phase protocol: gather facts → hypotheses → diagnostic → fix → explain. It cannot be shortcut.

---

## Quick Reference Checklist

Before writing any fix:

- [ ] Have I read the full error, not just the last line?
- [ ] Do I know *where in the system* the failure originates?
- [ ] Have I generated multiple hypotheses, not just one?
- [ ] Have I run a diagnostic to confirm which hypothesis is right?
- [ ] Is my fix addressing the root cause, not the symptom?
- [ ] Will I verify it worked after applying it?