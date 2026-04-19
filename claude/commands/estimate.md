# /estimate - Estimate unestimated Linear issues

Bulk-review issues missing estimates and assign t-shirt sizes interactively.

## Usage

```
/estimate                    # Work through all unestimated issues
/estimate ISSUE-12           # Estimate a specific issue
/estimate --project "P1"     # Estimate issues in a specific project
```

## Flow

1. **Fetch issues to estimate:**
   - Use `mcp__claude_ai_Linear__list_issues` to get open issues
   - Filter to those with no estimate set
   - If `--project` given, scope to that project

2. **For each issue, one at a time:**
   - Show title, description, labels, and any sub-issues
   - Read referenced code files if the description names them
   - Apply the **estimate skill** to reason about size
   - Suggest an estimate with a one-line rationale:
     > "I'd say **M** — data model change plus a UI update, probably a day's work."
   - Ask: "M, or different?" — accept the suggestion, adjust, or skip

3. **On confirmation:**
   ```bash
   linear issue update <id> --estimate <size>
   ```

4. **Continue** to the next unestimated issue

5. **At the end, summarize:**
   > "Estimated 6 issues. Largest: FIN-12 (XL). 2 skipped."
   - For any XL issues, suggest: "FIN-12 is XL — consider running `/split FIN-12`"

## Notes

- Follow the estimate skill for how to reason about size and what signals to use
- Skip issues you can't reason about: missing description, no acceptance criteria, blocked — note them at the end
- If L or XL, suggest `/split` but don't block the session on it
