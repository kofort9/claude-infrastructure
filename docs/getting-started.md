# Getting Started

How to use this Claude Code infrastructure in your own setup.

## Prerequisites

- Claude Code CLI installed
- `~/.claude/` directory exists

## Installation

### Option 1: Copy Individual Components

```bash
# Clone the repo
git clone https://github.com/yourusername/claude-infrastructure.git
cd claude-infrastructure

# Copy agents you want
cp agents/code-reviewer.md ~/.claude/agents/

# Copy commands you want
cp commands/document.md ~/.claude/commands/

# Copy skills you want
cp skills/terminal-ui-design.md ~/.claude/skills/
```

### Option 2: Use as Reference

Browse the repo and adapt patterns to your needs:
1. Read agent definitions for prompt patterns
2. Study command structures for workflow design
3. Use skills as knowledge templates

## Directory Structure

After installation:

```
~/.claude/
├── agents/
│   ├── public/           # Shareable agents
│   │   └── code-reviewer.md
│   └── private/          # Your personal agents
├── commands/
│   ├── public/           # General commands
│   │   └── document.md
│   └── p/                # Private commands
├── skills/
│   ├── public/           # Shareable skills
│   └── p/                # Private skills
├── hooks/                # Event-driven automation (optional)
├── scripts/              # Shell/Python automation (optional)
└── settings.json
```

## Creating Your Own

### Agent Template

```yaml
---
name: my-agent
description: When to use this agent
model: sonnet
tools:
  - Read
  - Glob
  - Grep
---

# My Agent

System prompt describing behavior...

## Workflow

1. Step one
2. Step two
3. ...
```

### Command Template

```yaml
---
name: my-command
description: What this command does
arguments:
  - name: target
    description: What to operate on
    required: false
---

# My Command

Instructions for the command...
```

### Skill Template

```yaml
---
name: my-skill
description: Knowledge this provides
---

# My Skill

Domain knowledge, patterns, examples...
```

## Testing

After adding components:

1. Restart Claude Code (or use `/reload`)
2. Try invoking your agent/command
3. Check for errors in output
4. Iterate on the definition

## Common Patterns

### Agent for Code Review
See `agents/code-reviewer.md` - comprehensive code review with structured output.

### Command for Documentation
See `commands/document.md` - workflow for creating/updating docs.

### Skill for UI Design
See `skills/terminal-ui-design.md` - terminal UI patterns and examples.

## Troubleshooting

**Agent not found:**
- Check file is in correct directory
- Verify YAML frontmatter is valid
- Restart Claude Code

**Command not working:**
- Check `allowed-tools` includes needed tools
- Verify arguments match usage
- Check for syntax errors in workflow

**Skill not referenced:**
- Skills are passive knowledge, not invoked directly
- Reference in agent/command prompts

## Contributing

Found improvements? Issues?
Open an issue or PR at the repo.
