# Claude Devtools

A Claude Code plugin providing a **multi-agent developer workflow toolkit** - code review, git operations, documentation, and codebase analysis with intelligent task delegation.

## Architecture Overview

This plugin demonstrates a **supervisor-worker agent architecture** where specialized agents handle domain-specific tasks while an orchestrator manages delegation and coordination.

```
                          ┌─────────────────────────────────────────┐
                          │            USER REQUEST                 │
                          └────────────────┬────────────────────────┘
                                           ▼
                          ┌─────────────────────────────────────────┐
                          │           ORCHESTRATOR                  │
                          │   (Task Analysis & Agent Routing)       │
                          │                                         │
                          │  • Analyzes task complexity             │
                          │  • Selects appropriate agent(s)         │
                          │  • Designs multi-agent workflows        │
                          │  • Coordinates sequential/parallel work │
                          └────────────────┬────────────────────────┘
                                           │
           ┌───────────────┬───────────────┼───────────────┬───────────────┐
           ▼               ▼               ▼               ▼               ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │   CODE      │ │   GIT       │ │   DOCS      │ │  SYSTEM     │ │  ANALYSIS   │
    │  REVIEWER   │ │  OPS        │ │  WRITER     │ │   OPS       │ │             │
    ├─────────────┤ ├─────────────┤ ├─────────────┤ ├─────────────┤ ├─────────────┤
    │ • Security  │ │ • Worktrees │ │ • READMEs   │ │ • Scripts   │ │ • Topology  │
    │ • Quality   │ │ • Branches  │ │ • API docs  │ │ • Configs   │ │ • Deps      │
    │ • PR review │ │ • PRs       │ │ • Runbooks  │ │ • Env setup │ │ • Structure │
    └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
           │               │               │               │               │
           └───────────────┴───────────────┴───────────────┴───────────────┘
                                           │
                          ┌────────────────┴────────────────┐
                          │      SHARED SKILL LIBRARY       │
                          │  (Reusable capabilities across  │
                          │         all agents)             │
                          └─────────────────────────────────┘
```

## Agent Workflow Patterns

### Feature Development (Worktree Isolation)

```
┌──────────────────────────────────────────────────────────────────────────┐
│ /worktree-create oauth-refresh                                           │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. gitops-devex ──▶ Creates isolated worktree                          │
│                      ~/Repos/project-worktrees/oauth-refresh/           │
│                      Branch: feature/oauth-refresh                       │
│                                                                          │
│  2. [Developer works in worktree]                                        │
│                                                                          │
│  3. /pre-commit-gate ──▶ Hard gate: lint, typecheck, tests              │
│         │                                                                │
│         ├─ PASS ──▶ Commit allowed                                      │
│         └─ FAIL ──▶ Block commit, show fixes                            │
│                                                                          │
│  4. code-reviewer ──▶ Automated quality review                          │
│                                                                          │
│  5. /pre-push-gate ──▶ Hard gate: build, full tests, health            │
│         │                                                                │
│         ├─ PASS ──▶ Push allowed                                        │
│         └─ FAIL ──▶ Block push, show issues                             │
│                                                                          │
│  6. gitops-devex ──▶ Create PR                                          │
│         │                                                                │
│         └──▶ tech-writer (spawned) ──▶ Generate PR description          │
│                                                                          │
│  7. /worktree-cleanup ──▶ Remove after merge                            │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Review Loop Automation

```
┌──────────────────────────────────────────────────────────────────────────┐
│ PR Review Handling                                                       │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Review Comment ──▶ /review-classify ──▶ Categorize:                    │
│                                                                          │
│    ┌─ [ACTIONABLE] ──▶ Auto-fix in worktree                             │
│    │                                                                     │
│    ├─ [STYLE] ──▶ Apply style fix                                       │
│    │                                                                     │
│    ├─ [TRADEOFF] ──▶ Escalate to developer                              │
│    │                                                                     │
│    ├─ [QUESTION] ──▶ Generate clarifying reply                          │
│    │                                                                     │
│    └─ [FALSE_POSITIVE] ──▶ Reply with rationale                         │
│                                                                          │
│  /review-response ──▶ Generate contextual response                      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

## Advanced Design Patterns

### Pattern 1: Capability Registry (Loose Coupling)

**Problem**: Agents hardcoding knowledge of other agents creates tight coupling. When responsibilities change, every agent needs updating.

**Solution**: The `/info-hub` skill acts as a capability registry - agents query it instead of hardcoding routing logic.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ BEFORE: Tight Coupling (Maintenance Nightmare)                           │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────┐     knows about      ┌─────────────┐                   │
│  │ orchestrator│ ───────────────────▶ │ tech-writer │                   │
│  └─────────────┘                      └─────────────┘                   │
│         │                                    │                           │
│         │ knows about                        │ knows about               │
│         ▼                                    ▼                           │
│  ┌─────────────┐     knows about      ┌─────────────┐                   │
│  │code-reviewer│ ◀─────────────────── │gitops-devex │                   │
│  └─────────────┘                      └─────────────┘                   │
│                                                                          │
│  Every agent knows about every other agent = N² connections             │
│  Change one agent = update all others                                    │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│ AFTER: Loose Coupling via Capability Registry                            │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────┐                      ┌─────────────┐                   │
│  │ orchestrator│ ──┐                  │ tech-writer │                   │
│  └─────────────┘   │                  └─────────────┘                   │
│                    │                         ▲                           │
│  ┌─────────────┐   │  ┌─────────────┐        │                           │
│  │code-reviewer│ ──┼─▶│  INFO-HUB   │────────┘  "who writes docs?"      │
│  └─────────────┘   │  │  (registry) │                                   │
│                    │  └─────────────┘        ┌─────────────┐             │
│  ┌─────────────┐   │         │               │gitops-devex │             │
│  │gitops-devex │ ──┘         └──────────────▶└─────────────┘             │
│  └─────────────┘                 "who handles git?"                      │
│                                                                          │
│  Agents query registry = N connections                                   │
│  Change one agent = update only registry                                 │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**Implementation**: Any agent can query `/info-hub <capability>` and get back the correct routing without knowing about other agents directly. This is dependency injection for multi-agent systems.

---

### Pattern 2: Hard Gates (No Bypass Design)

**Problem**: Soft warnings get ignored. "We'll fix it later" becomes technical debt. CI catches issues too late in the feedback loop.

**Solution**: Gates that **block** operations on failure with **no escape hatch**. Not even `--force`.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ SOFT WARNING (What most tools do)                                        │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Code Change ──▶ Lint Warning ──▶ "Do you want to continue? [Y/n]"      │
│                                              │                           │
│                                              ▼                           │
│                                         User hits Y                      │
│                                              │                           │
│                                              ▼                           │
│                                    Commit goes through                   │
│                                              │                           │
│                                              ▼                           │
│                                      CI fails later                      │
│                                              │                           │
│                                              ▼                           │
│                                    Context switch to fix                 │
│                                                                          │
│  Result: Warnings become noise. Everyone hits Y. Debt accumulates.      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│ HARD GATE (What this toolkit does)                                       │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Code Change ──▶ /pre-commit-gate                                       │
│                        │                                                 │
│           ┌────────────┴────────────┐                                   │
│           │                         │                                    │
│           ▼                         ▼                                    │
│     ┌─────────┐               ┌─────────┐                               │
│     │  PASS   │               │  FAIL   │                               │
│     └────┬────┘               └────┬────┘                               │
│          │                         │                                     │
│          ▼                         ▼                                     │
│   Commit allowed            ┌─────────────┐                             │
│                             │   BLOCKED   │                             │
│                             │             │                             │
│                             │ No --force  │                             │
│                             │ No bypass   │                             │
│                             │ Fix or stop │                             │
│                             └─────────────┘                             │
│                                    │                                     │
│                                    ▼                                     │
│                              Fix the issue                               │
│                                    │                                     │
│                                    ▼                                     │
│                              Re-run gate                                 │
│                                                                          │
│  Result: Problems fixed at the source. No debt. No CI failures.         │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**The Opinionated Part**: There's no `--force` flag. No `--skip-hooks`. If the gate fails, the only path forward is fixing the code. This is intentional friction that prevents quality erosion.

**Why No Bypass?** If you're frequently wanting to bypass, that's a signal the checks are too strict or too slow - the answer is to adjust the checks, not add escape hatches.

---

## Skill-to-Agent Mapping

Skills are reusable capabilities that agents invoke. Here's how they map:

| Skill | Primary Agent | Purpose |
|-------|---------------|---------|
| `/worktree-create` | gitops-devex | Create isolated workspace for feature |
| `/worktree-list` | gitops-devex | Show all active worktrees with health |
| `/worktree-cleanup` | gitops-devex | Remove merged worktrees |
| `/pre-commit-gate` | gitops-devex | Hard validation before commit |
| `/pre-push-gate` | gitops-devex | Hard validation before push |
| `/branch-health` | gitops-devex | Check staleness, conflicts, PR overlaps |
| `/review-classify` | code-reviewer | Categorize review comments |
| `/review-loop` | code-reviewer | Continuous review-fix cycle |
| `/review-response` | code-reviewer | Generate review responses |
| `/ci-status` | gitops-devex | Check GitHub Actions status |
| `/latest-comment` | code-reviewer | Get latest PR review comment |
| `/info-hub` | orchestrator | Query capability registry |
| `/workflow-map` | orchestrator | Agent/skill dashboard |
| `/landscape` | repo-topology | Codebase structure overview |
| `/repo-documentation` | tech-writer | Multi-agent documentation workflow |
| `/conversation-compression` | orchestrator | Context compaction |
| `/sync-agents` | plugin-manager | Sync to plugin repos |

## Installation

### As Claude Code Plugin

```bash
claude plugin add ~/Repos/claude-devtools
# Or from GitHub:
claude plugin add https://github.com/kofort9/claude-devtools
```

### Standalone Tools

Each CLI tool can be installed independently. See the tool's README for specific instructions.

<!-- COMPONENTS:START - Auto-generated, do not edit manually -->
## Components

### Agents (13)

| Agent | Model | Purpose |
|-------|-------|---------|
| **code-reviewer** | sonnet | A logical chunk of code has been written and needs revi |
| **doc-quality-reviewer** | sonnet | audit documentation for accuracy, completeness, and mai |
| **git-workflow-guardian** | haiku | Use this agent proactively when the user is about to ma |
| **gitops-devex** | opus | Unified Git workflow authority with worktree-based work |
| **linear-project-manager** | sonnet | you need to plan, organize, or coordinate multi-phase w |
| **orchestrator** | opus | A new task arrives that needs to be routed to the appro |
| **pattern-learner** | opus | Observes decisions across all domains, learns patterns, |
| **plugin-manager** | haiku | manage Claude Code plugin repositories. |
| **repo-topology** | sonnet | trace dependency graphs, or identify architectural patt |
| **scout** | sonnet | Reconnaissance agent for content triage and routing. Us |
| **system-admin** | sonnet | Use this agent for complex system projects, script deve |
| **system-ops** | sonnet | Use this agent for quick local machine operations and d |
| **tech-writer** | sonnet | Creating or updating README files, API docs, architectu |

### Skills (18)

| Skill | Purpose |
|-------|---------|
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
### CLI Tools

#### MCP Sync

MCP configuration consolidation tool for Claude Code.

**Location:** `mcp-sync/`

**Purpose:** Consolidates all Claude MCP server configurations from global and project-level settings files into a single canonical list.

**Features:**
- Scans all .claude/settings*.json files
- Deduplicates MCP server definitions
- Consolidates into global settings (~/.claude/settings.json)
- Dry-run mode for safe preview
- Timestamped backups before modifications

**Documentation:** See [mcp-sync/README.md](mcp-sync/README.md) for usage.

## Design Principles

### Hard Gates Over Soft Warnings
Quality gates (pre-commit, pre-push) **block** operations on failure rather than just warning. This ensures code quality is enforced, not suggested.

### Worktree Isolation
Each feature gets its own worktree - no stashing, no context switching, no branch collisions. Parallel development becomes trivial.

### Agent Autonomy with Coordination
Agents are specialized and autonomous within their domain. The orchestrator routes tasks but doesn't micromanage execution.

### Skills as Shared Capabilities
Skills are composable building blocks that multiple agents can invoke, preventing duplication and ensuring consistency.

## Requirements

- bash 3.2+ (compatible with macOS and Linux)
- jq (for JSON manipulation)
- Claude Code installed with .claude directory
- GitHub CLI (`gh`) for PR operations

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Related Projects

- [Claude Code](https://claude.com/claude-code) - Anthropic's official CLI for Claude
- [Model Context Protocol](https://modelcontextprotocol.io/) - Protocol for connecting AI assistants to data sources

## Support

For issues, questions, or feature requests, please open an issue on GitHub.
