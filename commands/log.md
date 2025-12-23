---
name: log
description: Add manual entry to current session log
arguments:
  - name: entry
    description: "The log entry text to add"
    required: false
---

# Manual Session Log Entry

Add a timestamped entry to the current session log.

## Workflow

1. Find today's session log(s) in `~/Documents/Obsidian/meta/session-logs/`
2. If multiple logs exist for today, ask which one to use
3. If no log exists, offer to create one
4. Add timestamped entry to the Session Log section
5. If entry mentions a task, offer to add to backlog

## Entry Format

```markdown
### HH:MM - [Brief description from user or inferred]
- [Entry content]
```

## Session Log Location

```bash
ls ~/Documents/Obsidian/meta/session-logs/$(date +%Y-%m-%d)*.md
```

## If No Session Log Exists

Ask user:
1. What topic is this session about?
2. Create new session log with standard template
3. Then add the entry

## Task Detection

If entry contains task-like language, ask:
- "This sounds like a task. Should I add it to the backlog?"

Patterns to detect:
- "need to..."
- "should..."
- "TODO"
- "don't forget..."
- "remember to..."

## After Adding

Confirm: "Added entry to [session-log-name] at HH:MM"
