---
name: sync-agents
description: Sync public agents and skills to claude-devtools repo, update architecture docs
argument-hint: "[--dry-run] [--push] [--docs-only]"
---

# Agent Sync Skill

Sync **public** agent definitions and skills from `~/.claude/` to the claude-devtools repo. Automatically updates README documentation with current component counts and architecture diagrams.

Private/personal agents (Obsidian, PKM workflows) are excluded automatically.

## Usage

```bash
/sync-agents              # Preview changes (dry-run)
/sync-agents --push       # Sync and push to GitHub
/sync-agents --diff       # Show detailed diff
/sync-agents --docs-only  # Only regenerate README docs
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

# Update README documentation (see step 3)
# ... doc generation ...

# Commit and push
cd "$REPO_DIR"
git add agents/ skills/ README.md
git status
git commit -m "sync: update agents and skills $(date +%Y-%m-%d)"
git push origin main
```

### 3. Documentation Generation

When syncing, automatically update README.md with:

1. **Component counts** - Scan directories for current agent/skill counts
2. **Agent table** - Extract name, model, description from frontmatter
3. **Skill categories** - Group skills by function
4. **Architecture diagrams** - Keep existing diagrams (manual updates only)

```bash
# Count components
AGENT_COUNT=$(ls -1 "$REPO_DIR/agents/"*.md 2>/dev/null | wc -l | tr -d ' ')
SKILL_COUNT=$(ls -1 "$REPO_DIR/skills/"*.md 2>/dev/null | wc -l | tr -d ' ')

echo "Agents: $AGENT_COUNT"
echo "Skills: $SKILL_COUNT"

# Update counts in README (sed in-place)
sed -i '' "s/### Agents ([0-9]*)/### Agents ($AGENT_COUNT)/" "$REPO_DIR/README.md"
sed -i '' "s/### Skills ([0-9]*)/### Skills ($SKILL_COUNT)/" "$REPO_DIR/README.md"
```

## What Gets Synced

### Public (Included)
- Generic dev tools: code-reviewer, gitops-devex, tech-writer, system-admin, system-ops
- Orchestration: orchestrator, plugin-manager
- Analysis: repo-topology, doc-quality-reviewer, git-workflow-guardian
- Project tracking: linear-project-manager
- Workflow skills: worktree-*, pre-commit-gate, review-*, branch-health, etc.

### Private (Excluded)

**Agents:**
- obsidian-writer, session-logger, weekly-reviewer
- conversation-classifier, conversation-compressor
- knowledge-validator, concept-linker, insight-extractor
- web-clipper, pdf-ingester, rag-optimizer
- project-researcher, macos-sysadmin, agent-sentinel

**Skills:**
- vault-search, checkpoint, project-pulse
- temporal-extraction, agent-output-recovery
- alias-correction, new-session
- append-log, log-media, browser-scrape

## Adding New Exclusions

When creating a new private agent/skill:

1. Add to exclusion lists in this skill (both preview and sync sections)
2. Add to `$REPO_DIR/skills/sync-agents.md` (the repo copy)
3. Component stays in `~/.claude/` only

## Safety

- Default is dry-run (preview only)
- `--push` required to actually sync
- Excludes archive to prevent publishing deprecated agents
- Excludes private/personal agents automatically
- Uses `$HOME` instead of hardcoded paths
