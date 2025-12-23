---
name: temporal-extraction
description: Extract and parse temporal metadata (dates, timestamps) from source files for provenance tracking
---

# Temporal Extraction Skill

Extract date/timestamp information from source files to enable temporal provenance in knowledge artifacts.

## Usage

```
/temporal-extraction [source_path]
/temporal-extraction --bulk [glob_pattern]
```

**Examples**:
- `/temporal-extraction "Nexus AI Chat Imports/2025/05/Some Conversation.md"`
- `/temporal-extraction --bulk "Nexus AI Chat Imports/2024/**/*.md"`

## Supported Sources

### ChatGPT Exports (Nexus AI Chat Imports)

**Location**: `~/Documents/Obsidian/Nexus AI Chat Imports/`

**Date Fields Available**:
| Field | Format | Granularity | Location |
|-------|--------|-------------|----------|
| `create_time` | `MM/DD/YYYY at H:MM AM/PM` | Day + Time | YAML frontmatter |
| `update_time` | `MM/DD/YYYY at H:MM AM/PM` | Day + Time | YAML frontmatter |
| Message timestamps | `HH:MM:SS` | Time only | Per-message in body |

**Parsing Logic**:
```python
# Extract from frontmatter
import re
from datetime import datetime

def parse_chatgpt_date(create_time_str):
    """Parse '12/21/2025 at 3:45 PM' to ISO-8601"""
    pattern = r"(\d{1,2})/(\d{1,2})/(\d{4}) at (\d{1,2}):(\d{2}) (AM|PM)"
    match = re.match(pattern, create_time_str)
    if match:
        month, day, year, hour, minute, ampm = match.groups()
        hour = int(hour)
        if ampm == "PM" and hour != 12:
            hour += 12
        elif ampm == "AM" and hour == 12:
            hour = 0
        return f"{year}-{int(month):02d}-{int(day):02d}"
    return None
```

**Output Format** (ISO-8601):
- Date only: `YYYY-MM-DD` (e.g., `2025-05-15`)
- Full timestamp: `YYYY-MM-DDTHH:MM:SS` (e.g., `2025-05-15T15:45:00`)

## Integration with Insights

When creating or updating insights, add `source_date` to frontmatter:

```yaml
---
title: Example Insight
source: "[[Nexus AI Chat Imports/2025/05/Career Discussion.md]]"
source_date: 2025-05-15
extracted: 2025-12-21
tags:
  - career
  - strategy
---
```

## Bulk Migration Workflow

For adding `source_date` to existing insights:

1. **Read insight** → Get source path from frontmatter
2. **Read source** → Extract `create_time` from source frontmatter
3. **Parse date** → Convert to ISO-8601 format
4. **Update insight** → Add `source_date` field to frontmatter

## Vault Paths

- **Insights**: `~/Documents/Obsidian/insights/`
- **ChatGPT Sources**: `~/Documents/Obsidian/Nexus AI Chat Imports/`

## Future Source Types

| Platform | Status | Date Granularity |
|----------|--------|------------------|
| ChatGPT Exports | Supported | Day + Time |
| Claude Conversations | Planned | TBD |
| Notion | Planned | TBD |
| Apple Notes | Planned | TBD |
| Linear | Planned | TBD |

## Evolution Notes

*Created 2025-12-21 based on exploration of ChatGPT export structure.*
*100% of 1,191 ChatGPT files have consistent `create_time` format.*
