# /issues - Browse and filter issues by project

## Usage

```
/issues                  # Browse active projects, then issues in selected project
/issues <project name>   # Jump directly to a named project
```

## Flow

### 1. Fetch projects

Use `mcp__claude_ai_Linear__list_projects` to list all projects.

Display only projects with status Backlog, Planned, or In Progress as a numbered list:

```
#   Name              Status       Issues
1   Phase 1           In Progress  12 open
2   Auth Redesign     Planned      5 open
3   Mobile App        Backlog      8 open
```

If no active projects are found: say "No active projects found. Use `/plan` to create one."

### 2. User selects a project

By number from the list. If a project name was passed directly (e.g., `/issues Phase 1`), skip the list.

### 3. Fetch and display issues

Use `mcp__claude_ai_Linear__list_issues` to get issues in the selected project.

Default: all non-completed issues. Display as a table:

```
ID       Pri   Status          Title                        Assignee    Est
FIN-42   🔴    In Progress     Fix auth token expiry        @alice      S
FIN-38   🟠    Ready for build Add retry logic              —           M
FIN-31   🟡    Backlog         Improve error messages       —           XS
```

Priority symbols: 🔴 Urgent, 🟠 High, 🟡 Medium, 🔵 Low, ⚪ None

If the project has no open issues: say "No open issues in <project>. Use `/plan` to add some."

### 4. Offer filter options

After showing the default list, offer:

```
Filter by status:   [1] Ready for build  [2] In Progress  [3] Backlog  [4] All
Filter by assignee: [m] Mine  [u] Unassigned  [a] All
```

Apply filters and redisplay if selected.

### 5. Issue detail on demand

If the user types an issue ID (e.g., `FIN-42`):
- Fetch full details via `mcp__claude_ai_Linear__get_issue`
- Display: title, description, acceptance criteria, assignee, estimate, labels, blockers, comments
- Offer: `/next FIN-42` to start work, or press Enter to return to the list

## Error Handling

- API error → "Could not reach Linear. Check your MCP connection with `/mcp`."
- Project not found by name → show the full project list and ask to select

## Notes

- Always show issue IDs so the user can run `/next <ID>` directly
- Blocked issues show the blocking issue ID next to them: `⊘ blocked by FIN-10`
