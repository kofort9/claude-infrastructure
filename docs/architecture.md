# Architecture Overview

How the Claude Code infrastructure components work together.

## Component Hierarchy

```
~/.claude/
├── agents/          # Specialized agent definitions
│   ├── public/      # Shareable agents
│   └── private/     # Personal/project-specific
├── commands/        # Slash commands
│   ├── public/      # General-purpose
│   └── p/           # Private workflows
├── skills/          # Reusable capabilities
├── hooks/           # Event-driven automation
├── patterns/        # Pattern learning system
├── scripts/         # Automation scripts
├── configs/         # Configuration files
└── settings.json    # Claude Code settings
```

## Agent Architecture

Agents are specialized personas invoked via the Task tool:

```yaml
---
name: agent-name
description: When to use this agent
model: sonnet|opus|haiku
tools:
  - Glob
  - Grep
  - Read
  # ... allowed tools
---

# System Prompt

Instructions for how the agent should behave...
```

### Agent Design Principles

1. **Single responsibility** - Each agent has one clear purpose
2. **Minimal tools** - Only grant tools needed for the task
3. **Clear triggers** - Description tells when to invoke
4. **Autonomous operation** - Can complete work without back-and-forth

## Command Architecture

Commands define user-invoked workflows:

```yaml
---
name: command-name
description: What this command does
arguments:
  - name: arg1
    description: First argument
    required: false
allowed-tools:
  - Bash(git:*)
  # ... tools this command can use
---

# Workflow

Step-by-step instructions...
```

### Command Design Principles

1. **Progressive disclosure** - Simple by default, options for power users
2. **Idempotent** - Safe to run multiple times
3. **Verbose feedback** - User knows what's happening
4. **Error handling** - Graceful failure with next steps

## Skill Architecture

Skills provide reusable knowledge:

```yaml
---
name: skill-name
description: What knowledge this provides
---

# Domain Knowledge

Patterns, best practices, examples...
```

Skills are referenced by agents and commands, not invoked directly.

## Hook Architecture

Hooks trigger on Claude Code events:

- `SessionStart` - When a session begins
- `SessionEnd` - When a session ends
- `PreToolUse` - Before a tool executes
- `PostToolUse` - After a tool executes
- `UserPromptSubmit` - When user sends a message

## Data Flow

```
User Input
    │
    ▼
┌─────────────────┐
│  Claude Code    │
│  (Main Agent)   │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐  ┌───────┐
│ Tools │  │ Task  │
│       │  │ (Sub) │
└───────┘  └───┬───┘
               │
               ▼
         ┌───────────┐
         │ Subagent  │
         │ (agents/) │
         └───────────┘
```

## Pattern System

The pattern learning system observes decisions and learns:

1. **Observation** - Log decisions in pattern files
2. **Clustering** - Group similar patterns
3. **Graduation** - High-confidence patterns become automated
4. **Validation** - Human review before graduation

See [patterns.md](patterns.md) for details.
