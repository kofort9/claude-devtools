# claude-mcp-sync

MCP configuration consolidation tool for Claude Code.

## Overview

`claude-mcp-sync` is a bash script that consolidates all Claude MCP (Model Context Protocol) server configurations from global and project-level settings files into a single canonical list in your global Claude settings.

When working with multiple Claude Code projects, you may accumulate MCP server definitions across different project-level settings files. This tool helps you maintain a clean, deduplicated list of all your MCP servers in one place.

## Features

- **Automatic Discovery:** Scans all `.claude/settings*.json` files in your home directory
- **Deduplication:** Intelligently combines MCP server definitions, removing duplicates
- **Safe Operations:** Dry-run mode lets you preview changes before applying them
- **Backup Management:** Creates timestamped backups before modifying settings
- **JSON Validation:** Validates JSON structure before writing files
- **Visual Feedback:** Color-coded terminal output for easy reading
- **Robust Error Handling:** Comprehensive error checking and recovery

## Installation

### Quick Install

```bash
# Copy to your local bin directory
cp claude-mcp-sync ~/bin/

# Make executable
chmod +x ~/bin/claude-mcp-sync

# Add to PATH if needed (add to ~/.bashrc or ~/.zshrc)
export PATH="$HOME/bin:$PATH"
```

### Verify Installation

```bash
claude-mcp-sync --help
```

## Usage

### Basic Usage

Run the consolidation (dry-run mode by default):
```bash
claude-mcp-sync
```

This will show you what changes would be made without actually modifying any files.

### Apply Changes

To actually consolidate the configurations:
```bash
claude-mcp-sync --apply
```

### Command-line Options

```
Usage: claude-mcp-sync [OPTIONS]

Options:
  --apply      Actually apply changes (default is dry-run)
  --help       Show this help message
  --version    Show version information
```

## How It Works

1. **Discovery Phase:**
   - Scans `~/.claude/` for all `settings*.json` files
   - Finds both global settings and project-level settings

2. **Extraction Phase:**
   - Extracts all MCP server definitions from each file
   - Maintains command, args, and env properties for each server

3. **Consolidation Phase:**
   - Deduplicates servers by name (keeps the first occurrence)
   - Builds a consolidated list of unique MCP servers

4. **Validation Phase:**
   - Validates the consolidated configuration
   - Checks JSON structure and required fields

5. **Application Phase (if --apply is used):**
   - Creates timestamped backup of global settings
   - Writes consolidated configuration to global settings
   - Verifies the write was successful

## Example Output

```
===========================================
Claude MCP Settings Consolidation Tool
===========================================

[INFO] Global settings file: /Users/username/.claude/settings.json
[INFO] Settings files to consolidate: 5

[FOUND] settings_project1.json
[FOUND] settings_project2.json
[FOUND] settings_project3.json
[FOUND] settings.json

[INFO] Consolidating MCP servers...
[FOUND] Server: trakt.tv-mcp
[FOUND] Server: github-mcp
[FOUND] Server: linear-mcp

[SUCCESS] Found 3 unique MCP servers across all settings files

[DRY RUN] The following changes would be made:

Consolidated MCP Servers (3 total):
├─ trakt.tv-mcp
├─ github-mcp
└─ linear-mcp

[INFO] No changes applied. Use --apply to actually consolidate settings.
```

## Requirements

- **bash:** Version 3.2 or higher (compatible with macOS default bash)
- **jq:** Required for JSON manipulation
  ```bash
  # Install on macOS
  brew install jq

  # Install on Ubuntu/Debian
  sudo apt-get install jq
  ```
- **Claude Code:** Must be installed with `.claude` directory in home folder

## Technical Details

- **Implementation:** 678 lines of bash script
- **Compatibility:** bash 3.2+ (macOS and Linux)
- **Dependencies:** jq for JSON manipulation
- **Backup Format:** `settings.json.backup.YYYYMMDD_HHMMSS`

## Safety Features

1. **Dry-run by Default:** Shows what would change without modifying files
2. **Timestamped Backups:** Original settings are preserved before any changes
3. **JSON Validation:** Ensures valid JSON before writing files
4. **Error Recovery:** Comprehensive error checking at each step
5. **Verbose Logging:** Clear messages about what's happening

## Troubleshooting

### "jq: command not found"

Install jq:
```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq
```

### "No .claude directory found"

Make sure Claude Code is installed and you've run it at least once to create the `.claude` directory.

### "Invalid JSON in settings file"

The tool will skip files with invalid JSON and show a warning. Check your settings files for syntax errors.

## Development

### Testing

```bash
# Run in dry-run mode to test
claude-mcp-sync

# Test with verbose output
bash -x claude-mcp-sync
```

### Debugging

The script includes extensive logging. Use the output to diagnose issues:
- `[INFO]` - Informational messages
- `[FOUND]` - Discovery messages
- `[SUCCESS]` - Successful operations
- `[WARNING]` - Non-critical issues
- `[ERROR]` - Critical errors

## License

MIT License - see [LICENSE](../LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Credits

Created as part of the Claude Code tooling ecosystem (KHQ-18).

Generated with Claude Code (https://claude.com/claude-code)
