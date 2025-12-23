---
name: vault-search
description: Quick search across the Obsidian vault for relevant knowledge
---

# Vault Search Skill

Search the Obsidian vault for relevant notes, insights, and concepts.

## Usage

```
/vault-search [query]
```

**Examples**:
- `/vault-search authentication patterns`
- `/vault-search rate limiting`
- `/vault-search what did I learn about postgres`

## Search Locations (Priority)

1. `insights/` - Extracted insights (highest signal)
2. `concepts/` - Concept notes
3. `meta/session-logs/` - Session logs
4. `sources/notes/` - PDF and web notes
5. `Nexus AI Chat Imports/` - Raw conversations (last resort)

## Workflow

1. Parse query for key terms
2. Search with Grep across vault
3. Rank results by location priority
4. Return top matches with context

## Vault Path

`~/Documents/Obsidian`

## Evolution Notes

*Search patterns will be refined through use.*
