---
name: backlog
description: Manage knowledge system backlog (add, view, promote, complete tasks)
arguments:
  - name: action
    description: "Action: add, view, promote, complete, idea, or status"
    required: false
  - name: item
    description: "Task or idea description (for add/idea actions)"
    required: false
---

# Backlog Management

Manage the knowledge system backlog at `~/Documents/Obsidian/meta/backlog.md`.

## Actions

| Action | Description |
|--------|-------------|
| `view` (default) | Show current backlog summary |
| `add [task]` | Add task to Active section |
| `idea [idea]` | Add to Ideas/Someday section |
| `promote [item]` | Move from Queued → Active |
| `complete [item]` | Move to Archive as completed |
| `queue [task]` | Add to Queued section |

## Workflow

1. Read current backlog from `~/Documents/Obsidian/meta/backlog.md`
2. Perform requested action
3. Update backlog file
4. Confirm change to user

## When Adding Items

Always capture:
- Task/idea description
- Source: Link to current session log or context
- Created date: Today's date
- Context: Why this matters (1 line)

## Format for New Items

```markdown
- [ ] [Task description]
  - Source: [[session-logs/YYYY-MM-DD-topic]]
  - Created: YYYY-MM-DD
  - Context: [Why this matters]
```

## For Ideas

```markdown
- **[Idea title]**
  - Context: [What problem this solves]
  - Source: [[session-logs/YYYY-MM-DD-topic]]
```

## After Changes

Update the "Last Updated" date and Backlog Stats section at the bottom.
