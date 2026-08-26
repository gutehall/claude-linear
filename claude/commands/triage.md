# /triage - Work through untriaged Linear issues interactively

## Usage

```
/triage              # All issues needing triage (no priority, no labels, no estimate, or unassigned)
/triage --priority   # Only issues with no priority set
/triage --unlabeled  # Only issues with no labels
```

## Flow

Follow the **triage** skill for the full load → suggest → interaction-loop → summary procedure, honoring whichever flag from `$ARGUMENTS` was passed (default: all untriaged issues).

## Error Handling

- If `save_issue` fails for an issue: print the error, offer to retry or skip, do not silently continue
- If a label name entered during edit does not match any available label: warn and re-prompt
- If the user quits mid-batch: report progress on issues already handled before stopping
