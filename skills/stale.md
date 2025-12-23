# Session Stale Check Skill

Checks if agents or skills have been modified since session start, indicating a restart may be needed to pick up changes.

## Usage

```
/stale
```

## Behavior

1. **Record session start**: On first invocation, capture current timestamp
2. **Check modifications**: Compare file mtimes in agent/skill directories against session start
3. **Report**: List any files modified after session started

## Directories Monitored

- `~/.claude/agents/` - Agent definitions
- `~/.claude/skills/` - Skill definitions
- `~/.claude/commands/` - Command definitions
- `~/.claude/settings.json` - Global settings
- `~/.claude/settings.local.json` - Local settings

## Output Format

### No Changes
```
Session is current. No agent/skill changes detected.
Started: 2025-12-23 00:15
```

### Changes Detected
```
Session may be stale. Modified since session start:

Agents (2):
  - gitops-devex.md (00:45)
  - orchestrator.md (00:32)

Skills (1):
  - review-classify.md (00:40)

Recommendation: Restart session to pick up changes.
Started: 2025-12-23 00:15
Current: 2025-12-23 00:50
```

## Implementation

```bash
# Get session start time (store in env or temp file on first run)
SESSION_START=${SESSION_START:-$(date +%s)}

# Find files modified after session start
find ~/.claude/agents ~/.claude/skills -type f -name "*.md" -newermt "@$SESSION_START" 2>/dev/null

# Check settings files
for f in ~/.claude/settings.json ~/.claude/settings.local.json; do
  if [ -f "$f" ] && [ $(stat -f %m "$f") -gt $SESSION_START ]; then
    echo "Modified: $f"
  fi
done
```

## Session Start Tracking

The skill needs to know when the current session started. Options:

1. **First invocation**: Store timestamp on first `/stale` call
2. **Environment variable**: Set `CLAUDE_SESSION_START` at session init
3. **Temp file**: Write to `~/.claude/.session-start`

Recommended: Use temp file approach for persistence across tool calls.

```bash
# On first call, create session marker
SESSION_FILE=~/.claude/.session-start
if [ ! -f "$SESSION_FILE" ]; then
  date +%s > "$SESSION_FILE"
fi
SESSION_START=$(cat "$SESSION_FILE")
```

## Integration Points

- **SessionEnd hook**: Could delete `.session-start` file on session end
- **Orchestrator**: Could proactively call `/stale` before delegating to agents
- **/sync-agents**: After syncing, remind user session may need restart

## Proactive Use

Consider calling `/stale` automatically:
- Before spawning agents for important tasks
- After running `/sync-agents`
- When user asks "why isn't X working?"

## Notes

- This skill doesn't force restart, just informs
- Changes to agents/skills require session restart to take effect
- Settings changes may or may not require restart depending on the setting
