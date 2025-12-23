---
name: log-media
description: Skill for logging watched media (TV episodes and movies) to Trakt.tv
---

# Log Media Skill

This skill enables the trakt-watch-companion agent to log watched media to Trakt.tv.

## Capabilities

- Log single episodes as watched
- Log single movies as watched
- Bulk log multiple episodes (using ranges like "1-5" or "1,3,5")
- Bulk log multiple movies
- Handle disambiguation when multiple matches found
- Maintain local watch history cache

## Local Watch History

**Database Agnostic Approach:** The skill maintains a local copy of watch history to avoid tight coupling to Trakt.tv as the sole data source.

Benefits:
- Faster queries (no API calls for history lookups)
- Works offline for viewing history
- Backup in case of API issues
- Can sync to other platforms in the future
- Reduces API rate limit pressure

Storage location: `.claude/data/watch-history.json`

Structure:
```json
{
  "movies": [
    {
      "title": "Dune",
      "year": 2021,
      "watched_at": "2025-12-08T20:30:00.000Z",
      "trakt_id": 12345,
      "imdb_id": "tt1160419"
    }
  ],
  "episodes": [
    {
      "show": "Breaking Bad",
      "season": 1,
      "episode": 1,
      "watched_at": "2025-12-08T19:00:00.000Z",
      "trakt_id": 67890,
      "show_trakt_id": 1388
    }
  ]
}
```

The skill should:
1. Log to Trakt.tv API first
2. On success, update local cache
3. If API fails but was logged before, note in local cache
4. Periodically sync local cache with Trakt.tv (reconciliation)

## Date Handling

**Important:** This skill receives ISO 8601 dates from Claude. Claude interprets natural language dates (like "yesterday", "last week") and converts them to ISO format before invoking this skill.

Accepted date formats:
- Date only: `2025-12-08`
- Full timestamp: `2025-12-08T20:30:00.000Z`
- Omit for current time

## Workflow

1. **Receive request** with media name, type, and optional date
2. **Search** for the media using `search_show` tool
3. **Handle disambiguation** if multiple matches (present options to user)
4. **Log the watch** using `log_watch` or `bulk_log` tool
5. **Update local cache** with logged entry
6. **Confirm** the logged item to user

## Tools Used

- `mcp__trakt__search_show` - Find media by name
- `mcp__trakt__search_episode` - Find specific episode
- `mcp__trakt__log_watch` - Log single item
- `mcp__trakt__bulk_log` - Log multiple items

## Examples

### Log single episode
User: "I watched Breaking Bad S1E1 yesterday"
→ Claude converts "yesterday" to "2025-12-08"
→ Call log_watch with type=episode, showName="Breaking Bad", season=1, episode=1, watchedAt="2025-12-08"
→ Update local cache

### Log movie
User: "Watched Dune last week"
→ Claude converts "last week" to "2025-12-02"
→ Call log_watch with type=movie, movieName="Dune", watchedAt="2025-12-02"
→ Update local cache

### Bulk log episodes
User: "Binged Stranger Things S4 episodes 1-5"
→ Call bulk_log with type=episodes, showName="Stranger Things", season=4, episodes="1-5"
→ Update local cache for all episodes
