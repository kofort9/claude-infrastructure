# Claude Code Infrastructure

Personal Claude Code setup showcasing agents, skills, and automation patterns.

## What's Here

- **agents/** - Specialized agents for code review, exploration, documentation
- **commands/** - Slash commands for common workflows
- **skills/** - Reusable workflow definitions
- **docs/** - Architecture and pattern documentation

## Quick Start

1. Copy agents to `~/.claude/agents/`
2. Copy commands to `~/.claude/commands/`
3. Copy skills to `~/.claude/skills/`
4. Restart Claude Code

## Architecture

See [docs/architecture.md](docs/architecture.md) for how these components work together.

## Key Concepts

### Agents
Specialized personas with focused capabilities. Each agent has:
- System prompt defining behavior
- Tool allowlist for its domain
- Usage triggers (when to invoke)

### Commands (Slash Commands)
User-invoked workflows via `/command-name`. Define:
- Arguments and options
- Step-by-step workflow
- Expected outputs

### Skills
Reusable capabilities that agents can reference. Provide:
- Domain knowledge
- Workflow patterns
- Best practices

<!-- COMPONENTS:START - Auto-generated, do not edit manually -->
## Components

### Agents (18)

| Agent | Model | Purpose |
|-------|-------|---------|
| **code-reviewer** | sonnet | Code review and quality validation before commits, PRs, or merging |
| **concept-linker** | haiku | Build and maintain the concept graph, link insights to concepts |
| **conversation-compressor** | sonnet | Compress long conversations that exceed token limits |
| **gitops-devex** | opus | Unified Git workflow authority with worktree-based workspace isolation |
| **insight-extractor** | sonnet | Extract insights from conversations and sources with provenance tracking |
| **knowledge-citadel** | sonnet | Utility agent for knowledge base retrieval from Obsidian vault |
| **ml-specialist** | opus | Reviews ML/statistical approaches in pattern systems |
| **orchestrator** | opus | Route tasks to appropriate agents, coordinate multi-agent workflows |
| **pattern-learner** | opus | Observe decisions across domains, learn patterns, suggest automation |
| **pdf-ingester** | sonnet | Import PDFs into the Obsidian knowledge base |
| **plugin-manager** | haiku | Manage Claude Code plugin repositories |
| **project-researcher** | sonnet | Research your vault for project-relevant knowledge |
| **repo-topology** | sonnet | Understand codebase structure, map module relationships, trace dependencies |
| **scout** | sonnet | Reconnaissance agent for content triage and routing |
| **system-admin** | sonnet | Complex system projects, script development, configuration management |
| **system-ops** | sonnet | Quick local machine operations and diagnostics |
| **tech-writer** | sonnet | Create, review, or update documentation |
| **web-clipper** | sonnet | Save web content to the Obsidian vault |

### Skills (21)

| Skill | Purpose |
|-------|---------|
| /ai-capture | Auto-detect platform from URL and route to appropriate capture command |
| /branch-health | Check git branch health - detect conflicts, staleness, divergence |
| /browser-scrape | Use Playwright MCP to screen-scrape external UIs for data extraction |
| /ci-status | Get CI job status for a PR or current branch |
| /conversation-compression | Compress long conversations using RAPTOR-inspired hierarchical summarization |
| /info-hub | Capability registry for agent ecosystem - query what agents can do |
| /landscape | Show current work landscape - branch, repo, PRs, latest commits |
| /latest-comment | Get just the latest review comment on a PR |
| /pre-commit-gate | Hard gate before any commit - runs lint, typecheck, and unit tests |
| /pre-push-gate | Hard gate before any push - runs full build, all tests, branch health |
| /repo-documentation | Multi-agent skill for maintaining high-quality, situational documentation |
| /review-classify | Parse code-reviewer output and produce an action plan with classifications |
| /review-loop | Autonomous PR review workflow - address comments, iterate until approved |
| /review-response | Respond to PR review comments, addressing false positives and tradeoffs |
| /tasks | Monitor and manage background task queue |
| /terminal-ui-design | Create distinctive, production-grade terminal user interfaces |
| /vault-stats | Display quick vault health stats showing file counts, quality metrics |
| /workflow-map | Generate a visual dashboard of all coding/GitHub workflows and agents |
| /worktree-cleanup | Remove worktrees for merged/closed PRs and their associated branches |
| /worktree-create | Create an isolated git worktree for a new feature/fix |
| /worktree-list | Show all active git worktrees with their status (branch, clean/dirty) |

### Commands (14)

| Command | Purpose |
|---------|---------|
| /afk | AFK return command - shows time gap, runs stale check, summarizes background |
| /chatgpt-single | Capture ChatGPT share conversations via Playwright |
| /claude-web | Capture Claude.ai share conversations via Playwright |
| /document | Create or update repository documentation using multi-agent workflow |
| /extract-patterns | Extract patterns from session logs (auto or deep mode) |
| /gemini | Capture Gemini AI share conversations via Playwright |
| /patterns-improve | Review and apply improvement suggestions from pattern analysis |
| /patterns-recurrence | View pattern recurrence clusters and escalation status |
| /patterns-verify | Check verification status of applied pattern fixes |
| /perplexity | Capture Perplexity AI search/chat shares via Playwright |
| /stale | Check if agents/skills modified since session start, recommend restart |
| /surface-patterns | Surface relevant patterns for current context |
| /timer | Time tracking for tasks and pattern analysis |
| /timestamp | Validated timestamps with midnight rollover detection |

*Auto-generated from frontmatter. See [/workflow-map](skills/workflow-map.md) for relationships.*
<!-- COMPONENTS:END -->

## Related

- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [MCP (Model Context Protocol)](https://modelcontextprotocol.io/)

## License

MIT - Feel free to use and adapt for your own Claude Code setup.
