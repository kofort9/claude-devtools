---
name: project-pulse
description: Aggregate project status from git, Linear, and session logs into a unified view
arguments: project-name (optional - infers from git if in repo)
---

# Project Pulse Skill

Aggregates high-value signals from multiple sources to provide project status. One-way extraction pattern - reads from external sources, synthesizes into Obsidian.

## Usage

```
/project-pulse [project-name]
```

**Examples**:
- `/project-pulse` - Infer project from current git repo
- `/project-pulse trakt-tv-mcp` - Explicit project name
- `/project-pulse --full` - Include Linear issues detail

## Sources

### 1. Git (local commits, branch state)
Primary source of "what actually got done"

### 2. Linear (issues, blockers, decisions)
Use Linear MCP tools. Extract high-value signals only:
- Issue state changes
- Comments with decisions
- Blockers

### 3. Session Logs (explicit documentation)
Search recent session logs for project mentions

## Execution Steps

### Step 1: Determine Project Context

```bash
# If no project specified, infer from git
PROJECT_NAME=${1:-$(basename $(git rev-parse --show-toplevel 2>/dev/null) 2>/dev/null)}

# Validate we have a project name
if [ -z "$PROJECT_NAME" ]; then
  echo "Error: No project specified and not in a git repo"
  exit 1
fi
```

### Step 2: Git Status (Recent Activity)

```bash
# Recent commits (last 7 days or last 10, whichever is more relevant)
git log --oneline --since="7 days ago" --format="%h %s (%ar)" | head -10

# Current branch
git branch --show-current

# Uncommitted changes
git status --porcelain | wc -l
```

### Step 3: Linear Status (if project has Linear issues)

Use Linear MCP to query:

```
mcp__linear-server__list_issues with:
  - project: <project-name> OR
  - label: <project-name>
  - limit: 10
  - orderBy: updatedAt
```

Extract:
- Issues in "In Progress" state
- Issues with "blocked" label or state
- Recent comments with decisions

### Step 4: Session Log Search

```bash
# Search recent session logs for project mentions
grep -l -i "$PROJECT_NAME" ~/Documents/Obsidian/meta/session-logs/*.md 2>/dev/null | head -5
```

For each matching file, extract:
- Date
- Key entries mentioning the project

### Step 5: Synthesize Output

```
=== PROJECT PULSE: [project-name] ===
Generated: YYYY-MM-DD HH:MM

## Git Activity (Last 7 Days)
Branch: <current-branch>
Commits: <count>
  - <hash> <message> (<relative-time>)
  - ...

Uncommitted: <count> files

## Linear Status
In Progress:
  - [PROJ-123] Issue title
  - ...

Blocked:
  - [PROJ-456] Issue title (reason: <blocked-reason>)

Recent Decisions:
  - [PROJ-789] <decision from comment>

## Session Log Mentions
- 2025-12-22: [session-log-name] - <relevant excerpt>
- ...

## Summary
<2-3 sentence synthesis of project state>
```

## Optional: Write to Projects Folder

If `--save` flag provided:

```bash
# Create project folder if doesn't exist
mkdir -p ~/Documents/Obsidian/projects/$PROJECT_NAME/status

# Write pulse snapshot
FILENAME="$(date +%Y-%m-%d).md"
# Write output to ~/Documents/Obsidian/projects/$PROJECT_NAME/status/$FILENAME
```

## High-Value Signal Filters

**Include:**
- State transitions (todo → in_progress → done)
- Blockers and their resolution
- Decisions documented in comments
- Commits with meaningful messages
- Session log entries with context

**Exclude:**
- Routine status updates
- Automated messages
- File-level change details
- Mechanical task completion logs

## Integration Notes

- Works standalone or within session workflow
- Output format suitable for session log entry
- Can be piped to `/checkpoint` for full snapshot
- Linear queries are optional - gracefully handles no Linear project
