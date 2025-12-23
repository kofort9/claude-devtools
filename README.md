<!-- COMPONENTS:START - Auto-generated, do not edit manually -->
## Components

### Agents (13)

| Agent | Model | Purpose |
|-------|-------|---------|
| **code-reviewer** | sonnet | Use this agent when: |
| **doc-quality-reviewer** | ? | Use this agent to audit documentation for accuracy, com |
| **git-workflow-guardian** | haiku | Use this agent proactively when the user is about to ma |
| **gitops-devex** | opus | Unified Git workflow authority with worktree-based work |
| **linear-project-manager** | sonnet | Use this agent when you need to plan, organize, or coor |
| **orchestrator** | opus | Use this agent when:\n\n1. A new task arrives that need |
| **pattern-learner** | opus | Observes decisions across all domains, learns patterns, |
| **plugin-manager** | haiku | Use this agent to manage Claude Code plugin repositorie |
| **repo-topology** | ? | Use this agent when you need to understand codebase str |
| **scout** | sonnet | Reconnaissance agent for content triage and routing. Us |
| **system-admin** | sonnet | Use this agent for complex system projects, script deve |
| **system-ops** | sonnet | Use this agent for quick local machine operations and d |
| **tech-writer** | sonnet | Use this agent when documentation needs to be created,  |

### Skills (21)

| Skill | Purpose |
|-------|---------|
| / |  |
| / |  |
| / |  |
| /back | AFK return command - shows time gap, runs stale check,  |
| /branch-health | Check git branch health - detect conflicts, staleness,  |
| /ci-status | Get CI job status for a PR or current branch |
| /conversation-compression | Compress long conversations using RAPTOR-inspired hiera |
| /info-hub | Capability registry for agent ecosystem. Query what age |
| /landscape | Show current work landscape - branch, repo, PRs, latest |
| /latest-comment | Get just the latest review comment on a PR |
| /pre-commit-gate | Hard gate before any commit - runs lint, typecheck, and |
| /pre-push-gate | Hard gate before any push - runs full build, all tests, |
| /repo-documentation | Multi-agent skill for maintaining high-quality, situati |
| /review-classify | Parse code-reviewer output and produce an action plan w |
| /review-loop | Autonomous PR review workflow - address latest comment, |
| /review-response | Respond to PR review comments, addressing false positiv |
| /sync-agents | Sync public agents and skills to claude-devtools repo,  |
| /workflow-map | Generate a visual dashboard of all coding/GitHub workfl |
| /worktree-cleanup | Remove worktrees for merged/closed PRs and their associ |
| /worktree-create | Create an isolated git worktree for a new feature/fix.  |
| /worktree-list | Show all active git worktrees with their status (branch |

### Commands (10)

| Command | Purpose |
|---------|---------|
| /afk | AFK return command - shows time gap, runs stale check,  |
| /backlog | Manage knowledge system backlog (add, view, promote, co |
| /document | Create or update repository documentation using multi-a |
| /inbox | Process items in the Obsidian inbox folder (classify, e |
| /log | Add manual entry to current session log |
| /stale | Check if agents/skills modified since session start, re |
| /status | Get knowledge system status report |
| /sweep | Audit and sync skills/agents across all Claude config l |
| /timer | Timer Skill |
| /timestamp | Timestamp Validation Skill |

*Auto-generated from frontmatter. See [/workflow-map](skills/workflow-map.md) for relationships.*
<!-- COMPONENTS:END -->
