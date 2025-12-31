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

### Agents (14)

| Agent | Model | Purpose |
|-------|-------|---------|
| **code-reviewer** | sonnet | **Proactive code quality specialist.** Invoke automatically after writing signif |
| **conversation-compressor** | sonnet | compress long conversations that exceed token limits. |
| **gitops-devex** | opus | Unified Git workflow authority with worktree-based workspace isolation. This is  |
| **ml-specialist** | opus | Reviews ML/statistical approaches in pattern systems - similarity algorithms, gr |
| **orchestrator** | opus | **Recommended entry point for complex tasks.** This agent routes work to special |
| **pdf-ingester** | sonnet | import PDFs into the Obsidian knowledge base. |
| **plugin-manager** | haiku | manage Claude Code plugin repositories. |
| **project-researcher** | sonnet | research your vault for project-relevant knowledge. |
| **repo-topology** | sonnet | you need to understand codebase structure, map module relationships,\ntrace depe |
| **scout** | sonnet | Reconnaissance agent for content triage and routing. Use when: |
| **system-admin** | sonnet | Use this agent for complex system projects, script development, and systematic m |
| **system-ops** | sonnet | Use this agent for quick local machine operations and developer environment task |
| **tech-writer** | sonnet | documentation needs to be created, reviewed, or updated. Specific scenarios incl |
| **web-clipper** | sonnet | save web content to the Obsidian vault. |

### Skills (21)

| Skill | Purpose |
|-------|---------|
| /ai-capture | Auto-detect platform from URL and route to appropriate capture command |
| /branch-health | Check git branch health - detect conflicts, staleness, and coordination issues w |
| /browser-scrape | Use Playwright MCP to screen-scrape external UIs for data APIs don't expose (act |
| /ci-status | Get CI job status for a PR or current branch |
| /conversation-compression | Compress long conversations using RAPTOR-inspired hierarchical summarization |
| /info-hub | Capability registry for agent ecosystem. Query what agent handles a specific tas |
| /landscape | Show current work landscape - branch, repo, PRs, latest activity, git status |
| /latest-comment | Get just the latest review comment on a PR |
| /pre-commit-gate | Hard gate before any commit - runs lint, typecheck, and tests. Blocks commit if  |
| /pre-push-gate | Hard gate before any push - runs full build, all tests, checks branch health. Bl |
| /repo-documentation | Multi-agent skill for maintaining high-quality, situationally aware repository d |
| /review-classify | Parse code-reviewer output and produce an action plan with explicit next steps.  |
| /review-loop | Autonomous PR review workflow - address latest comment, push, loop until clean |
| /review-response | Respond to PR review comments, addressing false positives and documenting decisi |
| /tasks | Monitor and manage background task queue |
| /terminal-ui-design | Create distinctive, production-grade terminal user interfaces with high design q |
| /vault-stats | Display quick vault health stats showing file counts, queue status, and last act |
| /workflow-map | Generate a visual dashboard of all coding/GitHub workflow agents, skills, and co |
| /worktree-cleanup | Remove worktrees for merged/closed PRs and their associated branches |
| /worktree-create | Create an isolated git worktree for a new feature/fix. Returns the workspace pat |
| /worktree-list | Show all active git worktrees with their status (branch, commits behind, PR stat |

### Commands (14)

| Command | Purpose |
|---------|---------|
| /afk | AFK return command - shows time gap, runs stale check, summarizes background tas |
| /chatgpt-single | Capture ChatGPT share conversations via Playwright. Saves full chat to Obsidian, |
| /claude-web | Capture Claude.ai share conversations via Playwright. Saves full chat to Obsidia |
| /document | Create or update repository documentation using multi-agent workflow |
| /extract-patterns | "Extract patterns from session logs (auto or deep mode)" |
| /gemini | Capture Gemini AI share conversations via Playwright. Saves full chat to Obsidia |
| /patterns-improve | "Review and apply improvement suggestions from pattern analysis" |
| /patterns-recurrence | "View pattern recurrence clusters and escalation status" |
| /patterns-verify | "Check verification status of applied pattern fixes" |
| /perplexity | Capture Perplexity AI search/chat shares via Playwright. Saves full Q&A to Obsid |
| /stale | Check if agents/skills modified since session start, recommend restart if stale |
| /surface-patterns | "Surface relevant patterns for current context" |
| /timer | Timer Skill |
| /timestamp | Timestamp Validation Skill |

*Auto-generated from frontmatter. See [/workflow-map](skills/workflow-map.md) for relationships.*
<!-- COMPONENTS:END -->

## Related

- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [MCP (Model Context Protocol)](https://modelcontextprotocol.io/)

## License

MIT - Feel free to use and adapt for your own Claude Code setup.
