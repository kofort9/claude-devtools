---
name: checkpoint
description: |
  Write a structured checkpoint for long-running task continuity.
  Use after completing a task, before context compaction risk, or at session end.
  Enables recovery after compaction or session restart.
---

# Checkpoint Skill

Write structured state for continuity across compaction and sessions.

## When to Use

- After completing a task/phase
- Before risky operations (large refactors)
- When context is getting long (compaction risk)
- At session end
- After PR creation

## Checkpoint Locations

1. **PROJECT_STATUS.md** - Canonical task state (in repo)
2. **Session Log** - Detailed decisions/context (in Obsidian)

## Checkpoint Format

### PROJECT_STATUS.md Update

Add to Session Log section:

```markdown
### YYYY-MM-DD (Session N)
- Completed: [task ID and description]
- PR: #[number] ([status: open/merged])
- Branch: [branch name]
- Key decisions: [brief list]
- Tests: [pass/fail, count]
- Next: [next task ID]
- Recovery: Read session log at [path] for full context
```

### Session Log Entry (Obsidian)

```markdown
## HH:MM - Checkpoint: [Task ID]

### State
- **Task**: [Phase X.Y - Description]
- **Branch**: [branch name]
- **PR**: [#number or "not yet created"]
- **Status**: [implementing | reviewing | testing | complete]

### Progress
- [x] Implementation complete
- [x] Tests written (N tests, all passing)
- [x] Code review passed (iteration 2)
- [ ] QA validation
- [ ] PR merged

### Loop State
- Code review iterations: 2
- Issues fixed: 3 (1 critical, 2 important)
- Current blocker: None

### Key Decisions This Session
1. [Decision]: [Rationale]
2. [Decision]: [Rationale]

### Context for Recovery
If resuming after compaction or new session:
1. Read PROJECT_STATUS.md for task list
2. Check git status: `git status && git log -3 --oneline`
3. Current task: [Task ID]
4. Next action: [specific next step]

### Open Questions
- [ ] [Question needing resolution]
```

## Compaction Recovery Protocol

When context is compacted, the model loses conversation history. To recover:

### Automatic Recovery (if session continues)
1. Read PROJECT_STATUS.md → identify current phase/task
2. Read latest session log → get loop state, decisions
3. Check git state → confirm branch, uncommitted changes
4. Resume from "Next action" in checkpoint

### Manual Recovery (new session)
1. User says "continue working on trakt.tv-mcp"
2. Read PROJECT_STATUS.md header → "Next Session Focus"
3. Read latest session log checkpoint
4. Summarize state to user, confirm before proceeding

## Integration with Workflow

```
Implement → Test → Review → [CHECKPOINT] → Next Task
                      ↓
              (if review fails)
                      ↓
            Fix → Test → Review → ...
                           ↓
                    (after N iterations)
                           ↓
                   [CHECKPOINT - loop state]
```

## Example Checkpoint

```markdown
## 15:45 - Checkpoint: Phase 0.2 Complete

### State
- **Task**: Phase 0.2 - Fix _retryCount crash
- **Branch**: fix/retry-count-init
- **PR**: #42 (open, awaiting merge)
- **Status**: complete

### Progress
- [x] Implementation complete
- [x] Tests written (2 tests, all passing)
- [x] Code review passed (iteration 1)
- [x] QA validation passed
- [x] PR created

### Loop State
- Code review iterations: 1
- Issues fixed: 0 (clean first pass)
- Current blocker: None

### Key Decisions This Session
1. Initialize _retryCount to 0 in axios interceptor: Defensive, no behavior change for existing code
2. Added unit test for undefined case: Reproduces exact crash scenario

### Context for Recovery
If resuming after compaction or new session:
1. PR #42 is open for Phase 0.2
2. Next task: Phase 0.3 - Search-first type inference
3. Branch for 0.3: feat/search-first-type
4. First step: Run pr-scoper for 0.3+0.4 batching decision

### Open Questions
- [ ] Should 0.3 and 0.4 be batched into one PR?
```

## Checkpoint Triggers

| Trigger | Action |
|---------|--------|
| Task complete | Full checkpoint |
| PR created | Checkpoint with PR number |
| Review iteration | Update loop state only |
| Session ending | Full checkpoint + "Next action" |
| Context > 50k tokens | Preemptive checkpoint |

## Anti-Patterns

- **Don't checkpoint every minor action** - Noise reduces signal
- **Don't skip checkpoints at task boundaries** - Critical for recovery
- **Don't write vague "next actions"** - Be specific enough to resume
