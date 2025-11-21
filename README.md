# Claude Devtools

Developer tools and utilities for Claude Code.

## Overview

This repository contains a collection of developer tools and utilities designed to enhance the Claude Code development experience. These tools help manage configurations, automate common tasks, and improve workflows when working with Claude Code and MCP (Model Context Protocol) servers.

## Tools

### MCP Sync

MCP configuration consolidation tool for Claude Code.

**Location:** `mcp-sync/`

**Purpose:** Consolidates all Claude MCP server configurations from global and project-level settings files into a single canonical list.

**Features:**
- Scans all .claude/settings*.json files
- Deduplicates MCP server definitions
- Consolidates into global settings (~/.claude/settings.json)
- Dry-run mode for safe preview
- Timestamped backups before modifications
- JSON validation before writing
- Color-coded terminal output

**Documentation:** See [mcp-sync/README.md](mcp-sync/README.md) for detailed usage instructions.

## Installation

Each tool can be installed independently. See the tool's README for specific installation instructions.

For mcp-sync:
```bash
# Copy to your local bin directory
cp mcp-sync/claude-mcp-sync ~/bin/

# Make executable
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
