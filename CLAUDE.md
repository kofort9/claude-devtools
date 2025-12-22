# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project Overview

**claude-devtools** - Public/shareable developer workflow toolkit for Claude Code.

Contains generic developer tools (code review, git operations, documentation, system admin) that can be shared with teams or open-sourced.

## Git Workflow Rules

### Branch-First Development (MANDATORY)

**Before making any commits to this repository:**

1. **Create a feature branch** from `main`:
   ```bash
   git checkout -b <type>/<description>
   # Examples:
   # feat/add-new-agent
   # fix/agent-tool-permissions
   # docs/update-readme
   ```

2. **Make commits** on the feature branch (not `main`)

3. **Create a Pull Request** before merging to `main`

4. **Update documentation** (README, etc.) before pushing

**Branch Naming Convention:**
| Prefix | Use Case |
|--------|----------|
| `feat/` | New agents, skills, commands |
| `fix/` | Bug fixes |
| `docs/` | Documentation only |
| `refactor/` | Code restructuring |
| `chore/` | Maintenance, dependencies |

**Exception:** Direct commits to `main` allowed only for:
- Typo fixes (< 3 lines changed)
- Emergency hotfixes (document afterward)

### Pre-Push Checklist

Before pushing any branch:
- [ ] Run `/document` if new agents/skills/commands were added
- [ ] Update README.md component counts
- [ ] Verify agent/skill descriptions are clear
- [ ] Check for hardcoded paths (should be none in devtools)

## Component Guidelines

### Agents
- Must have clear `description` with usage examples
- Specify appropriate `model` (haiku for fast, sonnet for complex, opus for orchestration)
- List all required `tools`

### Skills
- Single-purpose, composable
- Invokable via `/skill-name`
- No personal paths or credentials

### Commands
- User-facing entry points
- Map to skills or agent workflows

## Repository Structure

```
claude-devtools/
├── plugin.json          # Plugin manifest
├── agents/              # 12 agents
├── skills/              # 3 skills
├── commands/            # 1 command
└── mcp-sync/            # Standalone CLI tool
```

## Related Repositories

- `claude-personal` - Private PKM workflows (separate repo)
- `~/.claude/` - Global runtime installation

---

**Last Updated:** 2025-12-21
