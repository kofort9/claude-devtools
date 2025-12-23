---
name: sync-agents
description: Sync public agents and skills to claude-devtools repo
argument-hint: "[--dry-run] [--push]"
---

# Agent Sync Skill

Sync **public** agent definitions and skills from `~/.claude/` to the claude-devtools repo.

Private/personal agents (Obsidian, PKM workflows) are excluded automatically.

## Usage

```bash
/sync-agents              # Preview changes (dry-run)
/sync-agents --push       # Sync and push to GitHub
/sync-agents --diff       # Show detailed diff
```

## Workflow

### 1. Preview Mode (default)

```bash
# Show what would be synced (public agents only)
REPO_DIR="$HOME/Repos/claude-devtools"

echo "=== AGENTS ==="
diff -rq ~/.claude/agents/ "$REPO_DIR/agents/" \
  --exclude='archive' --exclude='*.backup*' \
  --exclude='obsidian-writer.md' \
  --exclude='conversation-classifier.md' \
  --exclude='knowledge-validator.md' \
  --exclude='conversation-compressor.md' \
  --exclude='session-logger.md' \
  --exclude='weekly-reviewer.md' \
  --exclude='project-researcher.md' \
  --exclude='concept-linker.md' \
  --exclude='insight-extractor.md' \
  --exclude='web-clipper.md' \
  --exclude='pdf-ingester.md' \
  --exclude='rag-optimizer.md' \
  --exclude='macos-sysadmin.md' \
  --exclude='agent-sentinel.md' \
  2>/dev/null || echo "Differences found"

echo "=== SKILLS ==="
diff -rq ~/.claude/skills/ "$REPO_DIR/skills/" \
  --exclude='browser-scrape.md' \
  --exclude='checkpoint.md' \
  --exclude='project-pulse.md' \
  --exclude='temporal-extraction.md' \
  --exclude='agent-output-recovery.md' \
  --exclude='alias-correction.md' \
  --exclude='new-session.md' \
  --exclude='vault-search.md' \
  --exclude='append-log.md' \
  --exclude='log-media.md' \
  2>/dev/null || echo "Differences found"
```

### 2. Sync Mode (--push)

```bash
REPO_DIR="$HOME/Repos/claude-devtools"

# Sync public agents only
rsync -av --delete \
  --exclude='archive/' \
  --exclude='*.backup*' \
  --exclude='obsidian-writer.md' \
  --exclude='conversation-classifier.md' \
  --exclude='knowledge-validator.md' \
  --exclude='conversation-compressor.md' \
  --exclude='session-logger.md' \
  --exclude='weekly-reviewer.md' \
  --exclude='project-researcher.md' \
  --exclude='concept-linker.md' \
  --exclude='insight-extractor.md' \
  --exclude='web-clipper.md' \
  --exclude='pdf-ingester.md' \
  --exclude='rag-optimizer.md' \
  --exclude='macos-sysadmin.md' \
  --exclude='agent-sentinel.md' \
  ~/.claude/agents/*.md \
  "$REPO_DIR/agents/"

# Sync public skills only
rsync -av --delete \
  --exclude='browser-scrape.md' \
  --exclude='checkpoint.md' \
  --exclude='project-pulse.md' \
  --exclude='temporal-extraction.md' \
  --exclude='agent-output-recovery.md' \
  --exclude='alias-correction.md' \
  --exclude='new-session.md' \
  --exclude='vault-search.md' \
  --exclude='append-log.md' \
  --exclude='log-media.md' \
  ~/.claude/skills/*.md \
  "$REPO_DIR/skills/"

# Commit and push
cd "$REPO_DIR"
git add agents/ skills/
git status
git commit -m "sync: update agents and skills $(date +%Y-%m-%d)"
git push origin main
```

## What Gets Synced

### Public (Included)
- Generic dev tools: code-reviewer, gitops-devex, tech-writer, system-admin
- Workflow skills: worktree-*, pre-commit-gate, review-*, etc.

### Private (Excluded)
- Obsidian/PKM agents: obsidian-writer, concept-linker, insight-extractor, etc.
- Personal workflow skills: vault-search, checkpoint, project-pulse, etc.
- Security agents: agent-sentinel (contains sensitive detection patterns)

## Safety

- Default is dry-run (preview only)
- `--push` required to actually sync
- Excludes archive to prevent publishing deprecated agents
- Excludes private/personal agents automatically
- Uses `$HOME` instead of hardcoded paths
