# /review

Perform a complete code review using the code-review skill.

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
3. Follow the full four-phase protocol: **Understand → Audit → Plan & Fix → Verify**
4. Do not skip phases even if the code looks simple
5. For every Critical or High issue: fix it, then verify with a test run
6. End with a clear summary of what was found, fixed, and verified

## Tone

Be direct. "This function has a bug on line 42" is better than "you might want to consider whether line 42 could potentially cause issues in some scenarios." If the code is correct, say so clearly and explain what you checked.
