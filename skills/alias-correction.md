---
name: alias-correction
description: Apply speech-to-text alias corrections during conversation processing
---

# Alias Correction Skill

Handles speech-to-text transcription errors in ChatGPT conversation content.

## Usage

```
/alias-correction [add|list|apply]
```

## Alias Configuration

**Location**: `~/Documents/Obsidian/analysis_results/aliases.yaml`

## Application Strategy: Both Pre + Post

### Pre-Processing (Before Classification)
- Correct titles and content for accurate topic detection
- Ensures classifier sees "Phila" not "Fila" when categorizing

### Post-Processing (Recording)
- Record what aliases were applied in `aliases_applied` field
- Preserves provenance: "Original said X, corrected to Y"

## Output Format

```json
{
  "file": "2024/08/Fila Backend Design.md",
  "original_title": "Fila Backend Design",
  "corrected_title": "Phila Backend Design",
  "topic": "career/projects",
  "aliases_applied": ["fila -> Phila"],
  "notes": "Title corrected from speech-to-text transcription"
}
```

## Commands

### Add New Alias
```
/alias-correction add "error form" "Correct Form"
```

### List All Aliases
```
/alias-correction list
```

### Apply to Existing Classifications
```
/alias-correction apply
```

## Integration

- Classifier loads `aliases.yaml` before processing
- Corrections applied to both title and content analysis
- All corrections recorded for transparency
