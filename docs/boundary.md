# Infrastructure vs Content Boundary

How to separate Claude Code infrastructure from project content.

## The Separation

| Aspect | Infrastructure (`~/.claude`) | Content (Project) |
|--------|------------------------------|-------------------|
| **Purpose** | Powers how Claude works | What you're building |
| **Changes** | Evolves with Claude Code | Evolves with your work |
| **Sharing** | Can be public (sanitized) | Usually private |
| **Examples** | Agents, commands, hooks | Code, docs, data |

## Infrastructure (`~/.claude`)

**What lives here:**
- `agents/` - Agent definitions (system prompts, tools)
- `commands/` - Slash commands
- `skills/` - Reusable workflow skills
- `hooks/` - Event-driven automation
- `patterns/` - Pattern learning system
- `scripts/` - Shell/Python automation
- `configs/` - Configuration files (credentials, accounts)
- `settings.json` - Claude Code settings

**Characteristics:**
- Reusable across projects
- Defines capabilities, not content
- Can be version controlled separately
- Public subset can be shared

## Content (Your Project)

**What lives here:**
- Source code
- Documentation
- Project-specific configs
- Data and assets

**Characteristics:**
- Project-specific
- The actual work product
- Usually private
- Standard git workflow

## Why Separate?

1. **Portability** - Infrastructure works across projects
2. **Sharing** - Share your setup without exposing project details
3. **Versioning** - Track infrastructure evolution separately
4. **Security** - Keep credentials isolated

## Implementation

### Directory Structure

```
~/.claude/                    # Infrastructure (tracked separately)
├── agents/
├── commands/
├── skills/
└── ...

~/Projects/
├── project-a/                # Content (project git)
│   ├── src/
│   ├── docs/
│   └── .git/
└── project-b/
    └── ...
```

### Git Strategy

**Infrastructure:**
```bash
# Private repo with full infrastructure
cd ~/Repos/claude-system
git push origin main

# Public repo with sanitized subset
cd ~/Repos/claude-infrastructure
# Auto-synced, sanitized
```

**Projects:**
```bash
# Standard project workflow
cd ~/Projects/project-a
git commit -m "feature: add new capability"
```

## Sync Automation

The `sync-agents-hook.sh` script:
1. Copies `public/` directories only
2. Sanitizes personal paths
3. Validates no secrets
4. Commits to public repo

Run on `SessionEnd` hook for automatic sync.
