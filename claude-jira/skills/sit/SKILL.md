---
name: sit
description: Forces Claude Code to pause, reflect, and honestly assess whether the current approach is correct before continuing. Use this skill whenever the user says "sit", "pause and think", "wait, stop", "take a step back", "are you sure about this?", "rethink what you're doing", or any phrase that signals they want Claude to stop and self-audit rather than keep going. Also trigger proactively when many tool calls have been made without user confirmation, when something unexpected has happened, when the task has grown more complex than anticipated, or before any destructive/hard-to-reverse action. SIT is a reflective pause — not a summary of what was done, but an honest evaluation of whether the work is on the right track and what should happen next.
---

# SIT Skill

SIT stands for **Stop, Inspect, Think**. It is a structured self-audit protocol for Claude Code.

## When to use

Trigger this skill when:
- The user says "sit", "pause", "stop and think", "rethink", "wait", "are you sure?", "step back"
- Many consecutive tool calls have been made without checking in
- Something unexpected, ambiguous, or error-like has occurred
- The task has grown in scope or complexity beyond what was originally described
- You are about to do something that is hard to undo
- Two approaches seem equally valid and a choice must be made

## How to use

1. **Read the protocol**: Load `sit.md` from this skill folder — it contains the full SIT protocol with all inspection questions and output format.

2. **Run the protocol honestly**: Work through each section. Do not skip steps. Do not write optimistic filler.

3. **Output the structured reflection**: Use the format defined in `sit.md`.

4. **Commit to a decision**: End with one of the four outcomes — Continue, Correct course, Ask the user, or Stop and report.

## Reference file

→ Read [`sit.md`](./sit.md) for the full protocol, reflection questions, output template, and decision table.

## Key principle

SIT is not a summary. It is a genuine reassessment. The test is: *would the user, reading the output, trust that Claude actually stopped and thought — or does it read like going-through-the-motions?*

Honest uncertainty is correct output. False confidence is not.
