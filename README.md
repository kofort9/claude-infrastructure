# Claude Code Infrastructure

Personal Claude Code setup showcasing agents, skills, and automation patterns.

## What's Here

```
claude-infrastructure/
├── agents/       # Specialized agents (18)
├── commands/     # Slash commands (14)
├── skills/       # Reusable skills (21)
├── protocols/    # Workflow protocols
└── docs/         # Architecture documentation
```

## Quick Start

1. Copy to your Claude Code config directory:
   ```bash
   cp -r agents/* ~/.claude/agents/
   cp -r commands/* ~/.claude/commands/
   cp -r skills/* ~/.claude/skills/
   cp -r protocols/* ~/.claude/protocols/
   ```
2. Restart Claude Code

## Architecture

See [docs/architecture.md](docs/architecture.md) for how components work together.

## Key Concepts

### Agents
Specialized personas with focused capabilities:
- System prompt defining behavior
- Tool allowlist for domain
- Triggers for when to invoke

### Commands
User-invoked workflows via `/command-name`:
- Arguments and options
- Step-by-step workflow
- Expected outputs

### Skills
Reusable capabilities that agents reference:
- Domain knowledge
- Workflow patterns
- Best practices

---

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| [code-reviewer](agents/code-reviewer.md) | sonnet | Proactive code quality specialist for bugs, security, performance |
| [concept-linker](agents/concept-linker.md) | haiku | Build and maintain concept graphs across knowledge bases |
| [conversation-compressor](agents/conversation-compressor.md) | sonnet | Compress long conversations exceeding token limits |
| [gitops-devex](agents/gitops-devex.md) | opus | Unified Git workflow authority with worktree isolation |
| [insight-extractor](agents/insight-extractor.md) | sonnet | Extract atomic insights from conversations and sources |
| [knowledge-citadel](agents/knowledge-citadel.md) | sonnet | Knowledge base retrieval - librarian behavior only |
| [ml-specialist](agents/ml-specialist.md) | opus | ML/statistical review for similarity algorithms, graduation |
| [orchestrator](agents/orchestrator.md) | opus | Task routing to specialized agents |
| [pattern-learner](agents/pattern-learner.md) | opus | Learn patterns, track metrics, suggest graduated autonomy |
| [pdf-ingester](agents/pdf-ingester.md) | sonnet | Import PDFs into knowledge bases |
| [plugin-manager](agents/plugin-manager.md) | haiku | Manage Claude Code plugin repositories |
| [project-researcher](agents/project-researcher.md) | sonnet | Research knowledge bases for project context |
| [repo-topology](agents/repo-topology.md) | sonnet | Understand codebase structure, map dependencies |
| [scout](agents/scout.md) | sonnet | Reconnaissance for content triage and routing |
| [system-admin](agents/system-admin.md) | sonnet | Complex system projects, script development |
| [system-ops](agents/system-ops.md) | sonnet | Quick local machine operations |
| [tech-writer](agents/tech-writer.md) | sonnet | Documentation creation, review, and updates |
| [web-clipper](agents/web-clipper.md) | sonnet | Save web content to knowledge bases |

## Skills

| Skill | Purpose |
|-------|---------|
| [/ai-capture](skills/ai-capture.md) | Auto-detect platform and route to capture command |
| [/branch-health](skills/branch-health.md) | Check git branch health and coordination issues |
| [/browser-scrape](skills/browser-scrape.md) | Screen-scrape UIs via Playwright MCP |
| [/ci-status](skills/ci-status.md) | Get CI job status for PR or branch |
| [/conversation-compression](skills/conversation-compression.md) | RAPTOR-inspired hierarchical summarization |
| [/info-hub](skills/info-hub.md) | Query capability registry for agent routing |
| [/landscape](skills/landscape.md) | Show current work context (branch, PRs, status) |
| [/latest-comment](skills/latest-comment.md) | Get latest review comment on a PR |
| [/pre-commit-gate](skills/pre-commit-gate.md) | Hard gate: lint, typecheck, tests before commit |
| [/pre-push-gate](skills/pre-push-gate.md) | Hard gate: build, all tests before push |
| [/repo-documentation](skills/repo-documentation.md) | Multi-agent documentation maintenance |
| [/review-classify](skills/review-classify.md) | Parse code-reviewer output into action plan |
| [/review-loop](skills/review-loop.md) | Autonomous PR review workflow |
| [/review-response](skills/review-response.md) | Respond to PR review comments |
| [/tasks](skills/tasks.md) | Monitor background task queue |
| [/terminal-ui-design](skills/terminal-ui-design.md) | Design terminal user interfaces |
| [/vault-stats](skills/vault-stats.md) | Display vault health stats |
| [/workflow-map](skills/workflow-map.md) | Visual dashboard of workflows |
| [/worktree-cleanup](skills/worktree-cleanup.md) | Remove worktrees for merged PRs |
| [/worktree-create](skills/worktree-create.md) | Create isolated git worktree |
| [/worktree-list](skills/worktree-list.md) | Show active worktrees with status |

## Commands

| Command | Purpose |
|---------|---------|
| [/afk](commands/afk.md) | AFK return - time gap, stale check, task summary |
| [/chatgpt-single](commands/chatgpt-single.md) | Capture ChatGPT conversations via Playwright |
| [/claude-web](commands/claude-web.md) | Capture Claude.ai conversations via Playwright |
| [/document](commands/document.md) | Create/update documentation via multi-agent workflow |
| [/extract-patterns](commands/extract-patterns.md) | Extract patterns from session logs |
| [/gemini](commands/gemini.md) | Capture Gemini conversations via Playwright |
| [/patterns-improve](commands/patterns-improve.md) | Review and apply pattern improvement suggestions |
| [/patterns-recurrence](commands/patterns-recurrence.md) | View pattern recurrence clusters |
| [/patterns-verify](commands/patterns-verify.md) | Check verification status of pattern fixes |
| [/perplexity](commands/perplexity.md) | Capture Perplexity searches via Playwright |
| [/stale](commands/stale.md) | Check if agents/skills modified since session start |
| [/surface-patterns](commands/surface-patterns.md) | Surface relevant patterns for context |
| [/timer](commands/timer.md) | Time tracking for tasks |
| [/timestamp](commands/timestamp.md) | Timestamp validation |

## Protocols

| Protocol | Purpose |
|----------|---------|
| [parallel-partition](protocols/parallel-partition.md) | Pattern for splitting work across parallel agents |

---

## Related

- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [MCP (Model Context Protocol)](https://modelcontextprotocol.io/)

## License

MIT - Feel free to use and adapt for your own Claude Code setup.
