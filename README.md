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

## Components

<!-- COMPONENTS:START - Auto-generated, do not edit manually -->
## Components

### Agents (18)

| Agent | Model | Purpose |
|-------|-------|---------|
| **code-reviewer** | sonnet | \n- A logical chunk of code has been written and needs  |
| **concept-linker** | haiku | build and maintain the concept graph. Invoke when ident |
| **conversation-compressor** | sonnet | compress long conversations that exceed token limits. |
| **gitops-devex** | opus | Unified Git workflow authority with worktree-based work |
| **insight-extractor** | sonnet | extract insights from conversations and sources. Invoke |
| **knowledge-citadel** | sonnet | Utility agent for knowledge base retrieval. Returns sum |
| **ml-specialist** | opus | Reviews ML/statistical approaches in pattern systems -  |
| **orchestrator** | opus | A new task arrives that needs to be routed to the appro |
| **pattern-learner** | opus | Observes decisions across all domains, learns system pa |
| **pdf-ingester** | sonnet | import PDFs into the Obsidian knowledge base. |
| **plugin-manager** | haiku | manage Claude Code plugin repositories. |
| **project-researcher** | sonnet | research your vault for project-relevant knowledge. |
| **repo-topology** | ? | you need to understand codebase structure, map module r |
| **scout** | sonnet | Reconnaissance agent for content triage and routing. Us |
| **system-admin** | sonnet | Use this agent for complex system projects, script deve |
| **system-ops** | sonnet | Use this agent for quick local machine operations and d |
| **tech-writer** | sonnet | documentation needs to be created, reviewed, or updated |
| **web-clipper** | sonnet | save web content to the Obsidian vault. |

### Skills (21)

| Skill | Purpose |
|-------|---------|
| /ai-capture | Auto-detect platform from URL and route to appropriate  |
| /branch-health | Check git branch health - detect conflicts, staleness,  |
| /browser-scrape | Use Playwright MCP to screen-scrape external UIs for da |
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
| /tasks | Monitor and manage background task queue |
| /terminal-ui-design | Create distinctive, production-grade terminal user inte |
| /vault-stats | Display quick vault health stats showing file counts, q |
| /workflow-map | Generate a visual dashboard of all coding/GitHub workfl |
| /worktree-cleanup | Remove worktrees for merged/closed PRs and their associ |
| /worktree-create | Create an isolated git worktree for a new feature/fix.  |
| /worktree-list | Show all active git worktrees with their status (branch |

### Commands (14)

| Command | Purpose |
|---------|---------|
| /afk | AFK return command - shows time gap, runs stale check,  |
| /chatgpt-single | Capture ChatGPT share conversations via Playwright. Sav |
| /claude-web | Capture Claude.ai share conversations via Playwright. S |
| /document | Create or update repository documentation using multi-a |
| /extract-patterns | "Extract patterns from session logs (auto or deep mode) |
| /gemini | Capture Gemini AI share conversations via Playwright. S |
| /patterns-improve | "Review and apply improvement suggestions from pattern  |
| /patterns-recurrence | "View pattern recurrence clusters and escalation status |
| /patterns-verify | "Check verification status of applied pattern fixes" |
| /perplexity | Capture Perplexity AI search/chat shares via Playwright |
| /stale | Check if agents/skills modified since session start, re |
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
