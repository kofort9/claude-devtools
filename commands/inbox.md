---
name: inbox
description: Process items in the Obsidian inbox folder (classify, extract, route)
arguments:
  - name: action
    description: "Action: list, process, clear, or status"
    required: false
---

# Inbox Processing

Process items captured in `~/Documents/Obsidian/inbox/`.

## Actions

| Action | Description |
|--------|-------------|
| `list` (default) | Show current inbox contents |
| `process` | Process all items (classify → extract → route) |
| `process [filename]` | Process specific item |
| `status` | Show inbox stats and last processed date |
| `clear` | Remove processed items (confirm first) |

## Processing Workflow

For each item in inbox/:

### 1. Detect Content Type

Use the **Type Registry** below to identify content. Match against patterns in order of specificity.

### 2. Route to Handler

Each type has a handler that defines: destination, processing steps, and validation.

### 3. Report Results

Summary of items processed, insights extracted, manual review needed.

---

## Type Registry

Extensible registry of content types. Add new types here as needed.

### Registered Types

| Type ID | Detection Pattern | Handler |
|---------|-------------------|---------|
| `chatgpt-export` | YAML frontmatter with `create_time`, `update_time` | `handler:chatgpt` |
| `claude-export` | TBD - future | `handler:claude` |
| `web-clipping` | URL in frontmatter or `source:` field | `handler:web-source` |
| `insight-draft` | Has `## Key Insight` or similar headers | `handler:insight` |
| `thinking-partner` | Dialogue/conversation needing synthesis | `handler:thinking-partner` |
| `project-note` | Has `project:` in frontmatter | `handler:project` |
| `reference` | PDF, image, or `type: reference` | `handler:reference` |
| `notion-export` | Notion-specific metadata | `handler:notion` |
| `linear-export` | Linear issue/project format | `handler:linear` |
| `apple-notes` | Apple Notes export format | `handler:apple-notes` |
| `unknown` | No pattern match | `handler:manual-review` |

### Adding New Types

To register a new content type, add a row to the table above and define its handler below.

---

## Handler Definitions

### handler:chatgpt

**Destination**: `sources/chatgpt/` → then insights extraction

**Steps**:
1. Move to `sources/chatgpt/YYYY/MM/` (by create_time date)
2. Run classifier for value assessment
3. If high-value: run insight-extractor
4. If insights created: run concept-linker
5. Add to validation queue

**Validation**: Requires `create_time` in frontmatter

---

### handler:web-source

**Destination**: `sources/web/`

**Steps**:
1. Extract/validate source URL
2. Add `fetched:` date if not present
3. Move to `sources/web/YYYY-MM-DD-title.md`
4. Run classifier for value assessment
5. If high-value: extract insights

**Validation**: Must have valid URL

---

### handler:insight

**Destination**: `insights/`

**Steps**:
1. Validate unique (check for duplicates via title/content similarity)
2. Ensure required fields: title, source, tags
3. Format as proper insight file
4. Move to `insights/`
5. Run concept-linker
6. Add to validation queue

**Validation**: Must be atomic, reusable insight

---

### handler:thinking-partner

**Destination**: `insights/` (synthesized) + `sources/dialogues/` (original)

**Purpose**: Process exploratory dialogues/conversations where ideas were developed through back-and-forth discussion. Extract the synthesized conclusions, not just raw exchanges.

**Steps**:
1. Archive original dialogue to `sources/dialogues/`
2. Identify key decision points and conclusions in the conversation
3. Synthesize into atomic insights (what was decided, learned, or clarified)
4. For each synthesized insight:
   - Create insight file with source link to dialogue
   - Tag with `thinking-partner` for provenance
5. Run concept-linker on new insights
6. Add to validation queue

**Detection Patterns**:
- Multiple speakers/turns
- Exploratory language ("what if", "I'm thinking", "let's consider")
- Conclusion markers ("so we decided", "the answer is", "key takeaway")

**Validation**: Must contain synthesizable conclusions (not pure brainstorming without resolution)

**Source**: Inspired by thinking partner agent pattern - see [[sources/web/thinking-partner-agent-pattern]]

---

### handler:project

**Destination**: `projects/[project-name]/`

**Steps**:
1. Extract project name from frontmatter
2. Create project folder if needed
3. Move file to project folder
4. Update project index if exists

**Validation**: Must have `project:` field

---

### handler:reference

**Destination**: `sources/references/`

**Steps**:
1. Move to `sources/references/`
2. Add metadata if missing (date, type)
3. No extraction - reference only

**Validation**: None required

---

### handler:notion (PLANNED)

**Status**: Not yet implemented

**Destination**: `sources/notion/`

**Steps**: TBD when Notion export available

---

### handler:linear (PLANNED)

**Status**: Not yet implemented

**Destination**: `sources/linear/`

**Steps**: TBD

---

### handler:apple-notes (PLANNED)

**Status**: Not yet implemented

**Destination**: `sources/apple-notes/`

**Steps**: TBD

---

### handler:claude (PLANNED)

**Status**: Not yet implemented

**Destination**: `sources/claude/`

**Steps**: Similar to chatgpt handler

---

### handler:manual-review

**Destination**: Stays in `inbox/`

**Steps**:
1. Flag for manual review
2. Add `needs-review:` tag
3. Report in processing summary

**Validation**: User must manually classify

---

## Inbox Rules

- **Inbox should be empty** after processing
- Items sitting >7 days: flag in weekly review
- Don't process during active work - batch at session end
- Unknown types stay in inbox until manually classified

## File Naming on Move

When moving files:
- No spaces (use hyphens)
- Lowercase
- Date prefix if not present: `YYYY-MM-DD-`
- Preserve meaningful original name

## Example Run

```
/inbox process

Processing inbox/ (5 items)...

1. "Career Strategy Chat.md"
   → Type: chatgpt-export (detected: create_time field)
   → Handler: handler:chatgpt
   → Moved to: sources/chatgpt/2025/05/career-strategy-chat.md
   → Value: high → extracting insights...
   → Insights: 2 created
   → Concepts: 3 linked

2. "interesting-article.md"
   → Type: web-clipping (detected: source URL)
   → Handler: handler:web-source
   → Moved to: sources/web/2025-12-21-interesting-article.md
   → Value: medium (no extraction)

3. "startup-pricing.md"
   → Type: insight-draft (detected: ## Key Insight header)
   → Handler: handler:insight
   → Validated: unique
   → Moved to: insights/startup-pricing-strategy.md
   → Concepts: 2 linked

4. "brainstorm-with-claude.md"
   → Type: thinking-partner (detected: dialogue + conclusions)
   → Handler: handler:thinking-partner
   → Archived to: sources/dialogues/2025-12-21-brainstorm-with-claude.md
   → Synthesized: 3 insights extracted
   → Concepts: 4 linked

5. "random-stuff.md"
   → Type: unknown
   → Handler: handler:manual-review
   → Action: MANUAL REVIEW NEEDED (staying in inbox)

Summary:
├── Processed: 4/5
├── Insights created: 6
├── Concepts linked: 9
├── Manual review: 1 item
└── Inbox remaining: 1
```

## Integration Points

- `conversation-classifier` agent: value assessment
- `insight-extractor` agent: high-value extraction
- `concept-linker` agent: concept mapping
- `knowledge-validator` agent: validation queue
- `meta/validation-queue.jsonl`: artifact tracking
