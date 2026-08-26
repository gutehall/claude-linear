# /issues - Browse and filter issues by epic

## Usage

```
/issues                  # Browse active epics, then issues in selected epic
/issues <epic name>      # Jump directly to a named epic
/issues sprint           # Browse by active sprint instead
```

## Flow

### 1. Fetch epics

```bash
jira epic list --plain
```

Display active epics as a numbered list:

```
#   Key         Name              Status
1   PROJ-100    Phase 1           In Progress
2   PROJ-110    Auth Redesign     To Do
```

If no active epics are found: say "No active epics found. Use `/plan` to create one."

### 2. User selects an epic

By number from the list. If an epic key or name was passed directly (e.g., `/issues Phase 1`), skip the list.
If `/issues sprint` was passed: skip to the sprint view (see below).

### 3. Fetch and display issues

```bash
jira issue list --epics PROJ-100 --resolution Unresolved --plain
```

Display as a table:

```
ID         Pri       Status          Type    Title                          Assignee    Est
PROJ-42    High      In Progress     Story   Fix auth token expiry          @alice      S
PROJ-38    Medium    To Do           Task    Add retry logic                —           M
PROJ-31    Low       To Do           Bug     Improve error messages         —           XS
```

Priority symbols: 🔴 Highest, 🟠 High, 🟡 Medium, 🔵 Low, ⚪ Lowest

If the epic has no open issues: say "No open issues in <epic>. Use `/plan` to add some."

### 3b. Sprint view (if `/issues sprint` was used)

```bash
jira board list --plain
jira sprint list --board-id <id> --plain
```

Show a numbered sprint list, user selects one, then:

```bash
jira issue list --sprint "Sprint 12" --resolution Unresolved --plain
```

### 4. Offer filter options

After showing the default list, offer:

```
Filter by status:   [1] To Do  [2] In Progress  [3] In Review  [4] All
Filter by assignee: [m] Mine  [u] Unassigned  [a] All
```

Apply filters and redisplay if selected.

### 5. Issue detail on demand

If the user types an issue key (e.g., `PROJ-42`):
- Run `jira issue view PROJ-42`
- Display: summary, description, assignee, type, priority, estimate, labels, linked issues, comments
- Offer: `/next PROJ-42` to start work, or press Enter to return to the list

## Error Handling

- CLI error → "Could not reach Jira. Check your configuration with `jira init`."
- Epic/sprint not found by name → show the full list and ask to select

## Notes

- Always show issue keys so the user can run `/next <ID>` directly
- Blocked issues show the blocking issue key next to them (from `jira issue view` linked issues): `⊘ blocked by PROJ-10`
