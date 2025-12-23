---
description: Get knowledge system status report
---

# Knowledge System Status

Generate a comprehensive status report of the Obsidian knowledge system.

## Data to Gather

### 1. Processing Pipeline State
```bash
# Total classified
wc -l ~/Documents/Obsidian/analysis_results/classifications-master.jsonl

# High-value count
grep -c '"value":"high"' ~/Documents/Obsidian/analysis_results/classifications-master.jsonl

# Insights created
ls ~/Documents/Obsidian/insights/*.md 2>/dev/null | wc -l

# Concepts created
ls ~/Documents/Obsidian/concepts/*.md 2>/dev/null | wc -l
```

### 2. Inbox State
```bash
ls ~/Documents/Obsidian/inbox/ 2>/dev/null | wc -l
```

### 3. Backlog Summary
Read `~/Documents/Obsidian/meta/backlog.md` and count items in each section.

### 4. Review Cadence
```bash
# Last review
ls -t ~/Documents/Obsidian/meta/reviews/*.md 2>/dev/null | head -1
```

### 5. Today's Session Log
```bash
ls ~/Documents/Obsidian/meta/session-logs/$(date +%Y-%m-%d)*.md 2>/dev/null
```

## Output Format

```markdown
## Knowledge System Status

**As of**: [current datetime]

### Processing Pipeline
| Stage | Count | Status |
|-------|-------|--------|
| Conversations Classified | 1,188 | ✅ Complete |
| High-Value Identified | 53 | ✅ Complete |
| Insights Extracted | [N] | [N] remaining |
| Concepts Created | [N] | - |

### Health Indicators
| Indicator | Value | Status |
|-----------|-------|--------|
| Inbox | [N] items | [✅ or ⚠️] |
| Weekly Review | [N] days ago | [✅ or ⚠️] |
| Today's Session | [exists/none] | - |

### Backlog Summary
| Section | Count |
|---------|-------|
| 🔴 Active | [N] |
| 🟡 Queued | [N] |
| 🔵 Ideas | [N] |
| ⚪ Blocked | [N] |

### Recommended Next Steps
1. [Top priority based on state]
2. [Secondary action]
3. [Tertiary action]
```

## Priority Logic

1. Inbox > 0 → "Process inbox"
2. Active backlog items → "Work on: [item]"
3. Insights < High-value → "Extract more insights"
4. Weekly review > 7 days → "Weekly review due"
5. All clear → "System healthy, consider promoting from Queued"
