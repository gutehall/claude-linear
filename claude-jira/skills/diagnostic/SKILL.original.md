---
name: diagnostic
description: Structured diagnosis protocol — stop guessing, gather facts, generate hypotheses, run the fastest test, then fix the confirmed root cause. Use this skill whenever the user runs /diagnose, reports a bug, encounters an error, or asks "why is X broken".
---

# Diagnostic Thinking

Stop guessing. Do not touch code until the root cause is confirmed.

## Phase 1: Read Everything

Before forming any theory, gather the facts:
- Read the full error — not just the last line
- Identify where the failure *originates* vs where it *surfaces*
- Check what changed recently: last commit, new installs, config edits
- Note the environment: versions, env vars, file paths, permissions

State what you have found before moving on.

## Phase 2: Generate Hypotheses

List **at least 3 candidate causes**, ranked by likelihood:

**Hypothesis A (most likely):** [specific mechanical explanation]
- Evidence for:
- Evidence against:

**Hypothesis B:** [alternative explanation]
- Evidence for:
- Evidence against:

**Hypothesis C:** [less likely but possible]
- Evidence for:
- Evidence against:

Hypotheses must be *mechanical and specific*. "Maybe it's a version issue" is not a hypothesis. "Package X requires peer dependency Y@^2 but Y@3 is installed, which removed the `connect()` method" is a hypothesis.

## Phase 3: Design a Diagnostic

Pick the fastest test that distinguishes between your top two hypotheses:
- A specific log or print statement at the right location
- A direct inspection command (`which`, `env`, `ls -la`, `cat`, `curl`)
- Temporarily hardcoding a value to isolate a variable
- A minimal reproduction (strip everything until only the broken part remains)

Run the diagnostic. Update your hypothesis ranking based on the result.

## Phase 4: Fix with Confidence

Only now — with a confirmed root cause — write the fix:
- Address the root cause, not the symptom
- Make the minimum change required
- Check for side effects

Verify it worked: run the thing, check the output, confirm the error is gone.

## Phase 5: Explain What You Found

1. What was actually wrong (root cause)
2. Why it was wrong (what condition caused it)
3. What the fix does (why it resolves the root cause)

## Rules

- Do not write any code until Phase 4
- If you cannot confirm a hypothesis with available information, say so and ask for the specific output or file you need
- "I need X before I can fix this" is always better than applying a guess

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

This skill is invoked by the `/diagnose` command (`claude-jira/commands/diagnose.md`), installed the same way as the rest of this package — see the "Testing changes" section of the repo's CLAUDE.md.

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
