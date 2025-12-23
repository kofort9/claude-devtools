---
name: recover-agent-output
description: |
  Recover and persist output from agents that hit permission/sandbox issues.
  Use when:
  - Background agent completed but couldn't write files
  - Agent returned structured data (JSONL, markdown) in response text
  - Need to extract embedded content and save to intended destination
allowed_tools: Read, Write, Edit, Bash, TaskOutput
---

# Agent Output Recovery Skill

When background agents hit sandbox restrictions, they often complete their task conceptually but can't persist the results. This skill recovers that output.

## Trigger Conditions

1. Agent completed with message like "permission issues" or "cannot write"
2. Agent response contains structured data (JSONL, code blocks, markdown)
3. Agent specified an intended output path

## Recovery Workflow

### Step 1: Check Agent Output

```bash
# View agent's full output
cat /tmp/claude/-Users-kofifort-Documents/tasks/{agent_id}.output
```

### Step 2: Identify Structured Content

Look for:
- JSONL blocks (one JSON per line)
- Markdown files with frontmatter
- Code blocks with file paths mentioned
- Patterns like "I would have written to..."

### Step 3: Extract and Write

For JSONL output:
```python
# The agent typically embeds the intended content in its response
# Extract the JSON lines and write to destination
```

For markdown files (insights):
```markdown
# Look for files with frontmatter like:
---
doc_type: insight
created: YYYY-MM-DD
---
```

### Step 4: Verify and Report

- Confirm files were created
- Count records/files recovered
- Update any tracking systems

## Common Scenarios

### Scenario A: insight-extractor can't write to insights/

1. Agent reads conversations and generates insight notes
2. Agent can't write to `~/Documents/Obsidian/insights/`
3. Agent includes full insight markdown in response
4. **Action**: Extract markdown blocks, write each to insights/ folder

### Scenario B: conversation-compressor can't write JSONL

1. Agent analyzes conversations for Level 2 summaries
2. Agent can't write to `analysis_results/`
3. Agent includes JSONL content in response text
4. **Action**: Extract JSONL, write to intended file

### Scenario C: classifier agent times out

1. Agent processed most files but hit token limit
2. Partial JSONL output in response
3. **Action**: Extract partial output, note where to resume

## Prevention Tips

For future agent invocations:
1. Pre-create destination folders: `mkdir -p ~/Documents/Obsidian/insights`
2. Use smaller batches for write-heavy tasks
3. Consider using main session for writes, agent for analysis only

## Integration

After recovery, update:
- Session log with what was recovered
- Any TODO items related to the agent's task
- Evolution notes if this is a recurring pattern
