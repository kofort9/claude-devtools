---
name: sync-agents
description: Sync local agents and skills to claude-devtools repo
argument-hint: "[--dry-run] [--push]"
---

# Agent Sync Skill

Sync local agent definitions and skills from `~/.claude/` to the claude-devtools repo.

## Usage

```bash
/sync-agents              # Preview changes (dry-run)
/sync-agents --push       # Sync and push to GitHub
/sync-agents --diff       # Show detailed diff
```

## Workflow

### 1. Preview Mode (default)

```bash
# Show what would be synced
echo "=== AGENTS ==="
diff -rq ~/.claude/agents/ /Users/kofifort/Repos/claude-devtools/agents/ \
  --exclude='archive' --exclude='*.backup*' 2>/dev/null || echo "Differences found"

echo "=== SKILLS ==="
diff -rq ~/.claude/skills/ /Users/kofifort/Repos/claude-devtools/skills/ 2>/dev/null || echo "Differences found"
```

### 2. Sync Mode (--push)

```bash
# Sync agents (exclude archive and backups)
rsync -av --delete \
  --exclude='archive/' \
  --exclude='*.backup*' \
  ~/.claude/agents/*.md \
  /Users/kofifort/Repos/claude-devtools/agents/

# Sync skills
rsync -av --delete \
  ~/.claude/skills/*.md \
  /Users/kofifort/Repos/claude-devtools/skills/

# Commit and push
cd /Users/kofifort/Repos/claude-devtools
git add agents/ skills/
git status
git commit -m "sync: update agents and skills from local $(date +%Y-%m-%d)"
git push origin main
```

## What Gets Synced

### Included
- `~/.claude/agents/*.md` → `claude-devtools/agents/`
- `~/.claude/skills/*.md` → `claude-devtools/skills/`

### Excluded
- `~/.claude/agents/archive/` - Archived agents stay local
- `*.backup*` files - Old versions stay local
- Commands - Managed separately

## Current State

Run `/sync-agents` to see current diff between local and repo.

## Safety

- Default is dry-run (preview only)
- `--push` required to actually sync
- Excludes archive to prevent accidentally publishing deprecated agents
- Uses `rsync --delete` to remove agents from repo that were archived locally
