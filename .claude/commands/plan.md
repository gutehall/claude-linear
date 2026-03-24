# /plan - Inline product planning with Linear

Create Linear issues directly via MCP, without clipboard or intermediate steps.

## Usage

```
/plan                    # Open-ended planning session
/plan "Add X feature"    # Focused planning for a specific area
```

## Flow

1. **Read Linear state:**
   - List open issues (`mcp__claude_ai_Linear__list_issues`)
   - List projects (`mcp__claude_ai_Linear__list_projects`)
   - List teams to get the correct team ID (`mcp__claude_ai_Linear__list_teams`)

2. **If input is vague or missing, ask three questions before doing anything else:**
   - What needs to happen?
   - Who is it for?
   - How do you know it's done?
   Draft the issue from those answers. Skip this step if the input already answers all three.

3. **Run product planning** inline — follow the product-planning skill guidelines:
   - Understand what's already planned
   - Identify gaps, priorities, or the focus area provided
   - Draft issues with clear titles, descriptions, and acceptance criteria

4. **Create issues via MCP** directly:
   - Use `mcp__claude_ai_Linear__save_issue` for each issue
   - Set appropriate team, priority, and labels
   - Print each created issue ID and title

5. **Offer to start:** suggest `/next <ID>` for the highest-priority issue created

## Rules

- Always read current Linear state before planning — don't create duplicates
- Issues need: title, description, acceptance criteria, team, priority
- Keep issues small enough to implement in one PR
- No clipboard, no JSON payload, no external scripts
