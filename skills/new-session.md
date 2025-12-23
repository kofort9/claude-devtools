---
name: new-session
description: Create a new session log in Obsidian for tracking knowledge work
---

# New Session Skill

Creates a new session log in the Obsidian vault for documenting a knowledge work session.

## Usage

```
/new-session [topic]
```

**Examples**:
- `/new-session` - Creates session, prompts for topic
- `/new-session api-refactoring` - Creates session with topic slug
- `/new-session "Knowledge System Migration"` - Creates session with full title

## Workflow

1. **Get current date/time**:
   ```bash
   date "+%Y-%m-%d %H:%M"
   ```

2. **Check for existing sessions today**:
   ```bash
   ls -la ~/Documents/Obsidian/meta/session-logs/$(date "+%Y-%m-%d")*.md 2>/dev/null
   ```

3. **Time-based decision logic**:
   - If existing session today with SAME topic → ask: "Continue existing or start fresh?"
   - If existing session today with DIFFERENT topic → create new session
   - If no session today → create new session
   - If existing session is >4 hours old → suggest new session even for same topic

4. **If topic not provided**: Ask user for session topic/goal

4. **Generate filename**: `YYYY-MM-DD-<topic-slug>.md`
   - Convert topic to lowercase
   - Replace spaces with hyphens
   - Remove special characters

5. **Create session log** at `~/Documents/Obsidian/meta/session-logs/`

## Session Template

```markdown
# [Topic Title] - Session Log

**Date**: YYYY-MM-DD
**Session Start**: HH:MM
**Goal**: [User-provided goal or derived from topic]

---

## Executive Summary

[To be filled as session progresses]

---

## Decisions Made

*None yet*

---

## Open Questions

*None yet*

---

## Future Tooling

*Ideas for automation or process improvements*

---

## Next Steps (Proposed)

*To be determined*

---

## Session Log

### HH:MM - Session started
- Topic: [Topic]
- Context: [Any initial context]
```

## After Creation

- Confirm the file was created
- Show the path to the new session log
- Remind user they can use `/append-log` to add entries

## Notes

- Always use `date` command for current date/time (never assume)
- One day can have multiple sessions (different topics)
- If a session with same topic exists today, ask if user wants to continue it instead
