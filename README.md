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

## Components

### Agents (11)

| Agent | Model | Specialization |
|-------|-------|----------------|
| **orchestrator** | opus | Meta-agent for task delegation and multi-agent coordination |
| **code-reviewer** | sonnet | Code quality, security, PR reviews, review-loop automation |
| **gitops-devex** | opus | Git worktrees, branches, commits, PRs, CI/CD (all git ops) |
| **tech-writer** | sonnet | Documentation, PR descriptions, READMEs, runbooks |
| **system-admin** | sonnet | Complex system projects, script development, audits |
| **system-ops** | sonnet | Quick local machine operations, running existing scripts |
| **repo-topology** | sonnet | Codebase architecture analysis and dependency mapping |
| **doc-quality-reviewer** | sonnet | Audit documentation for accuracy and maintainability |
| **git-workflow-guardian** | haiku | Proactive merge conflict and branch health monitoring |
| **linear-project-manager** | sonnet | Linear integration for project tracking |
| **plugin-manager** | haiku | Classify and sync components between plugin repos |

### Skills (17)

| Category | Skills |
|----------|--------|
| **Git Workflow** | worktree-create, worktree-list, worktree-cleanup, branch-health |
| **Quality Gates** | pre-commit-gate, pre-push-gate, ci-status |
| **Code Review** | review-classify, review-loop, review-response, latest-comment |
| **Orchestration** | info-hub, workflow-map, conversation-compression |
| **Documentation** | repo-documentation, landscape |
| **Plugin Mgmt** | sync-agents |

### Commands (1)

| Command | Purpose |
|---------|---------|
| **/document** | Create or update repository documentation |

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
