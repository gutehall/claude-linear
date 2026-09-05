# /review

Perform a complete code review using the code-review skill.

This command warrants more reasoning than a routine turn, regardless of the session's default effort level — a missed bug here ships to production. Read carefully rather than skimming for the obvious issues.

## What this does

This command triggers a full structured code review that:
1. Understands the intent of the code
2. Audits for correctness, edge cases, error handling, and broken contracts
3. Reports all issues with severity ratings
4. Produces a fix plan and applies fixes (with your confirmation)
5. Runs tests to verify the fixes hold

## Arguments

`$ARGUMENTS` — Optional. Can be:
- A file path (`src/auth/token.ts`)
- A module or feature name (`the payment flow`, `the Lambda handler`)
- A description of scope (`everything in the auth module`)
- Empty — reviews the current working context

## Instructions

Read the code-review SKILL.md before starting. Then:

1. If `$ARGUMENTS` is empty, ask the user what code to review, or infer from recent context
2. Read the relevant files
3. Spawn a cheap reviewer on the diff (`git diff <base>..HEAD`, or `gh pr diff` if `$ARGUMENTS` names a PR) for the Audit pass — see **Subagents** below
4. Follow the full four-phase protocol: **Understand → Audit → Plan & Fix → Verify**, using verified reviewer findings as the Audit input
5. Do not skip phases even if the code looks simple
6. For every Critical or High issue: fix it, then verify with a test run
7. End with a clear summary of what was found, fixed, and verified

## Subagents (leads, not proof)

Prefer `cavecrew-reviewer` if that agent type exists; otherwise Task/Explore on Haiku or the cheapest offered model. Demand a compressed `path:line: problem. fix.` receipt — treat every line as a lead, not a confirmed finding. Read the cited span yourself and confirm it before fixing, commenting, or including it in the summary. Discard a finding that doesn't hold — do not re-spawn to "prove" it. Only if no subagent tool exists at all: review the diff on the main thread.

## Tone

Be direct. "This function has a bug on line 42" is better than "you might want to consider whether line 42 could potentially cause issues in some scenarios." If the code is correct, say so clearly and explain what you checked.
