---
description: Stop guessing. Think through the problem before touching anything.
argument-hint: [error message or description of what's broken]
---

This command warrants more reasoning than a routine turn, regardless of the session's default effort level — a wrong root cause costs more than the extra time spent finding the right one. Slow down: consider more than the first plausible explanation, and don't shortcut to a fix.

The problem to diagnose: $ARGUMENTS

Load the **diagnostic-thinking** skill and follow its 5-phase protocol (Read Everything → Generate Hypotheses → Design a Diagnostic → Fix with Confidence → Explain What You Found) in full. Do not write code, edit files, or apply a fix before Phase 4 — do not skip phases even if the cause looks obvious.

## Subagents (leads, not proof)

For Phase 1, spawn a cheap locator subagent (1–2 in parallel) to locate the failure site, recent related changes, and existing tests. Prefer `cavecrew-investigator` if that agent type exists; otherwise Task/Explore on Haiku or the cheapest offered model. Demand a compressed `path:line` receipt — treat it as a lead, not a confirmed fact. Read every cited span yourself before building a hypothesis on it. A lead that doesn't hold gets discarded — do not re-spawn to "prove" it. The hypothesis in Phase 2 stays on the main thread; do not fix or theorize inside the subagent. Only if no subagent tool exists at all: grep/read on the main thread. Never spawn a full-size Sonnet agent just to locate files.

If you cannot confirm a hypothesis with available information, say so clearly and ask for the specific output or file you need. It is always better to say "I need X before I can fix this" than to apply a guess.
