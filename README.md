# Claude Devtools

Developer tools, agents, and utilities for Claude Code.

## Overview

A Claude Code plugin providing portable developer workflow tools - code review, git operations, documentation, and codebase analysis. Plus standalone CLI utilities for configuration management.

## Installation

### As Claude Code Plugin

```bash
claude plugin add ~/Repos/claude-devtools
# Or from GitHub:
claude plugin add https://github.com/kofort9/claude-devtools
```

### Standalone Tools

Each CLI tool can be installed independently. See the tool's README for specific instructions.

## Components

### Agents (12)

| Agent | Model | Purpose |
|-------|-------|---------|
| **orchestrator** | opus | Meta-agent for task delegation and multi-agent coordination |
| **code-reviewer** | sonnet | Code quality analysis, security assessment, issue detection |
| **tech-writer** | sonnet | Documentation, PR descriptions, migration guides |
| **gitops-devex** | sonnet | Git workflow, branch operations, GitHub integration |
| **git-workflow-guardian** | haiku | Proactive merge conflict prevention |
| **repo-topology** | - | Codebase architecture analysis and dependency mapping |
| **rag-optimizer** | - | Optimize docs for RAG systems (embedding, chunking) |
| **doc-quality-reviewer** | - | Audit documentation for accuracy and maintainability |
| **system-admin** | sonnet | Complex system projects and script development |
| **system-ops** | sonnet | Quick local machine operations |
| **plugin-manager** | haiku | Manage Claude Code plugin repositories |
| **macos-sysadmin** | sonnet | macOS system administration and automation |

### Skills (3)

| Skill | Purpose |
|-------|---------|
| **repo-documentation** | Multi-agent documentation workflow (analysis → synthesis → validation) |
| **branch-health** | On-demand git branch health checks (staleness, conflicts, PR overlaps) |
| **alias-correction** | Correct and suggest command aliases |

### Commands (1)

| Command | Purpose |
|---------|---------|
| **/document** | Create or update repository documentation |

### CLI Tools

#### MCP Sync

MCP configuration consolidation tool for Claude Code.

**Location:** `mcp-sync/`

**Purpose:** Consolidates all Claude MCP server configurations from global and project-level settings files into a single canonical list.

**Features:**
- Scans all .claude/settings*.json files
- Deduplicates MCP server definitions
- Consolidates into global settings (~/.claude/settings.json)
- Dry-run mode for safe preview
- Timestamped backups before modifications

**Documentation:** See [mcp-sync/README.md](mcp-sync/README.md) for usage.

```bash
# Install
cp mcp-sync/claude-mcp-sync ~/bin/
chmod +x ~/bin/claude-mcp-sync

# Run
claude-mcp-sync
```

## Requirements

- bash 3.2+ (compatible with macOS and Linux)
- jq (for JSON manipulation)
- Claude Code installed with .claude directory

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Related Projects

- [Claude Code](https://claude.com/claude-code) - Anthropic's official CLI for Claude
- [Model Context Protocol](https://modelcontextprotocol.io/) - Protocol for connecting AI assistants to data sources

## Support

For issues, questions, or feature requests, please open an issue on GitHub.
