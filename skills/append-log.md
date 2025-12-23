---
name: append-log
description: Add a timestamped entry to the current session log in Obsidian
---

# Append Log Skill

Adds a timestamped entry to an existing session log.

## Usage

```
/append-log [content]
```

**Examples**:
- `/append-log Explored the codebase structure`
- `/append-log decision: Use PostgreSQL for better JSON support`
- `/append-log question: How should we handle auth tokens?`

## Workflow

1. Get current time: `date "+%H:%M"`

2. Find today's most recent session log:
   ```bash
   ls -t ~/Documents/Obsidian/meta/session-logs/$(date "+%Y-%m-%d")*.md 2>/dev/null | head -1
   ```

3. If no session exists: suggest `/new-session` first

4. Append timestamped entry to Session Log section:
   ```markdown
   ### HH:MM - [Entry]
   - [Content]
   ```

5. If content starts with `decision:`, `question:`, or `next:`, also add to the appropriate section

## Vault Path

`~/Documents/Obsidian/meta/session-logs/`

## Evolution Notes

This skill will learn common patterns as it's used:
- Frequently used entry types
- Preferred formatting
- Section placement rules

*Patterns discovered through use will be added here.*
