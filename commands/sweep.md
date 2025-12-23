---
name: sweep
description: Audit and sync skills/agents across all Claude config locations
arguments:
  - name: action
    description: "sync | report | diff (default: report)"
    required: false
---

# Plugin Component Sweep

Audit skills, agents, and commands across all configuration locations and optionally sync them.

## Locations to Scan

1. **Global**: `~/.claude/` (agents/, skills/, commands/)
2. **Plugin: claude-devtools**: `~/Repos/claude-devtools/` (agents/, skills/, commands/)
3. **Plugin: claude-personal**: `~/Repos/claude-personal/` (agents/, skills/, commands/)
4. **Project: trakt.tv-mcp**: `~/Repos/trakt.tv-mcp/.claude/` (agents/, skills/)

## Actions

### `report` (default)
Generate a status report showing:
- Total count of each component type per location
- Components unique to each location (orphans)
- Components that exist in multiple locations (duplicates)
- Recently modified files (last 7 days)

### `diff`
For duplicate components, show file differences between locations.
Use `diff` command to compare content.

### `sync`
Interactive sync workflow:
1. Show orphaned components
2. Ask which direction to sync (global → repos OR repos → global)
3. Copy missing files to target location
4. Report what was synced

## Report Format

```markdown
## Plugin Component Sweep Report

**Generated**: [timestamp]

### Summary
| Location | Agents | Skills | Commands |
|----------|--------|--------|----------|
| ~/.claude/ | N | N | N |
| claude-devtools | N | N | N |
| claude-personal | N | N | N |
| trakt.tv-mcp | N | N | N |

### Orphaned Components (exist in only one location)

#### Global Only (~/.claude/)
- agents/macos-sysadmin.md
- agents/conversation-compressor.md
- ...

#### claude-devtools Only
- agents/system-admin.md
- agents/git-workflow-guardian.md

#### claude-personal Only
- (none)

### Recently Modified (last 7 days)
| File | Location | Modified |
|------|----------|----------|
| agent-name.md | ~/.claude/agents | 2 days ago |

### Recommended Actions
1. [Sync recommendation based on orphans]
2. [Review duplicates with differences]
```

## Implementation Steps

1. **Count components**:
```bash
# Global
ls ~/.claude/agents/*.md 2>/dev/null | wc -l
ls ~/.claude/skills/*.md 2>/dev/null | wc -l
ls ~/.claude/commands/*.md 2>/dev/null | wc -l

# claude-devtools
ls ~/Repos/claude-devtools/agents/*.md 2>/dev/null | wc -l
ls ~/Repos/claude-devtools/skills/*.md 2>/dev/null | wc -l
ls ~/Repos/claude-devtools/commands/*.md 2>/dev/null | wc -l

# claude-personal
ls ~/Repos/claude-personal/agents/*.md 2>/dev/null | wc -l
ls ~/Repos/claude-personal/skills/*.md 2>/dev/null | wc -l
ls ~/Repos/claude-personal/commands/*.md 2>/dev/null | wc -l

# trakt.tv-mcp
ls ~/Repos/trakt.tv-mcp/.claude/agents/*.md 2>/dev/null | wc -l
ls ~/Repos/trakt.tv-mcp/.claude/skills/*.md 2>/dev/null | wc -l
```

2. **Find orphans** (components in one location only):
```bash
# Get basenames from each location
ls ~/.claude/agents/*.md 2>/dev/null | xargs -n1 basename
ls ~/Repos/claude-devtools/agents/*.md 2>/dev/null | xargs -n1 basename
# Compare using comm or diff
```

3. **Find recently modified**:
```bash
find ~/.claude/agents ~/.claude/skills ~/.claude/commands \
  ~/Repos/claude-devtools/agents ~/Repos/claude-devtools/skills \
  ~/Repos/claude-personal/agents ~/Repos/claude-personal/skills \
  -name "*.md" -mtime -7 2>/dev/null
```

4. **For sync action**: Use `cp` to copy missing files, confirm before overwriting

## Sync Rules

- **Global → Repo**: Copy component to appropriate plugin based on domain:
  - Dev tools (code-reviewer, gitops, etc.) → claude-devtools
  - Personal tools (PKM, session, etc.) → claude-personal

- **Repo → Global**: Copy to ~/.claude/ for global availability

- **Never overwrite without confirmation** when content differs
