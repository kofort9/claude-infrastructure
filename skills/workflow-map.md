---
name: workflow-map
description: Generate a visual dashboard of all coding/GitHub workflow agents, skills, and commands. Use for orientation, auditing, and planning.
arguments: [--scope coding|all|personal] [--format compact|full]
---

# Workflow Map Skill

Generates a visual dashboard showing how agents, skills, and commands relate to each other in your workflow. Essential for:
- **Orientation** - Understanding what's available
- **Auditing** - Finding gaps, redundancies, unclear components
- **Planning** - Deciding what to build or revise

## Usage

```
/workflow-map                    # Default: coding workflow, compact format
/workflow-map --scope all        # All agents (including personal/knowledge)
/workflow-map --scope coding     # Just coding/GitHub workflow
/workflow-map --format full      # Include descriptions and tools
```

## What It Shows

### Agents
- Name, model, purpose
- Health status (documented? tools defined? model set?)
- Integration points (what invokes it, what it invokes)

### Skills
- Name, purpose
- Which agents use it
- Arguments/usage

### Commands
- Name, purpose
- User-facing entry points

## Implementation

```bash
#!/bin/bash

SCOPE="${1:-coding}"
FORMAT="${2:-compact}"

AGENTS_DIR="$HOME/.claude/agents"
SKILLS_DIR="$HOME/.claude/skills"
COMMANDS_DIR="$HOME/.claude/commands"
ARCHIVE_DIR="$AGENTS_DIR/archive"

echo ""
echo "╔══════════════════════════════════════════════════════════════════════════╗"
echo "║                         WORKFLOW MAP                                      ║"
echo "║                    Generated: $(date '+%Y-%m-%d %H:%M')                         ║"
echo "╚══════════════════════════════════════════════════════════════════════════╝"
echo ""

# ===== AGENTS =====
echo "AGENTS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# Coding-related keywords for filtering
CODING_KEYWORDS="git|github|code|review|pr|pull|commit|push|branch|repo|build|ci|test|lint|tech|doc|system|plugin|orchestrat"

for agent_file in "$AGENTS_DIR"/*.md; do
  [ -f "$agent_file" ] || continue

  AGENT_NAME=$(basename "$agent_file" .md)

  # Skip if filtering by scope
  if [ "$SCOPE" = "coding" ]; then
    if ! grep -qiE "$CODING_KEYWORDS" "$agent_file"; then
      continue
    fi
  fi

  # Extract metadata
  MODEL=$(grep -E "^model:" "$agent_file" | head -1 | sed 's/model: *//')
  COLOR=$(grep -E "^color:" "$agent_file" | head -1 | sed 's/color: *//')

  # Check health
  HEALTH="✅"
  [ -z "$MODEL" ] && HEALTH="⚠️ no model"

  # Get first line of description
  DESC=$(grep -A1 "^description:" "$agent_file" | tail -1 | head -c 60)

  printf "  %-25s %-8s %-6s %s\n" "$AGENT_NAME" "${MODEL:-?}" "$HEALTH" "$COLOR"
done

echo ""

# ===== ARCHIVED AGENTS =====
if [ -d "$ARCHIVE_DIR" ] && [ "$(ls -A "$ARCHIVE_DIR" 2>/dev/null)" ]; then
  echo "ARCHIVED AGENTS"
  echo "───────────────"
  for agent_file in "$ARCHIVE_DIR"/*.md; do
    [ -f "$agent_file" ] || continue
    AGENT_NAME=$(basename "$agent_file" .md)
    echo "  ❌ $AGENT_NAME"
  done
  echo ""
fi

# ===== SKILLS =====
echo "SKILLS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# Group skills by category (based on name prefix)
declare -A SKILL_CATEGORIES
SKILL_CATEGORIES=(
  ["worktree"]="WORKTREE"
  ["pre-"]="GATES"
  ["review"]="REVIEW"
  ["branch"]="STATUS"
  ["ci-"]="STATUS"
  ["landscape"]="STATUS"
  ["project"]="STATUS"
  ["checkpoint"]="STATUS"
)

for skill_file in "$SKILLS_DIR"/*.md; do
  [ -f "$skill_file" ] || continue

  SKILL_NAME=$(basename "$skill_file" .md)

  # Skip if filtering by scope
  if [ "$SCOPE" = "coding" ]; then
    if ! grep -qiE "$CODING_KEYWORDS" "$skill_file"; then
      continue
    fi
  fi

  # Get description
  DESC=$(grep -E "^description:" "$skill_file" | head -1 | sed 's/description: *//' | head -c 50)

  printf "  /%-24s %s\n" "$SKILL_NAME" "$DESC"
done

echo ""

# ===== COMMANDS =====
echo "COMMANDS (User Entry Points)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

for cmd_file in "$COMMANDS_DIR"/*.md; do
  [ -f "$cmd_file" ] || continue

  CMD_NAME=$(basename "$cmd_file" .md)
  DESC=$(grep -E "^description:" "$cmd_file" | head -1 | sed 's/description: *//' | head -c 50)

  printf "  /%-24s %s\n" "$CMD_NAME" "$DESC"
done

echo ""

# ===== WORKFLOW DIAGRAM =====
echo "WORKFLOW RELATIONSHIPS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
cat << 'DIAGRAM'
                    ┌─────────────────────────────────┐
                    │         orchestrator            │
                    │        (task routing)           │
                    └───────────────┬─────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
  ┌───────────┐             ┌───────────┐             ┌───────────┐
  │ gitops-   │             │  code-    │             │   tech-   │
  │ devex     │◄───────────►│ reviewer  │◄───────────►│  writer   │
  │           │             │           │             │           │
  └─────┬─────┘             └───────────┘             └───────────┘
        │
        │ Skills:
        ├── /worktree-create
        ├── /worktree-list
        ├── /worktree-cleanup
        ├── /pre-commit-gate
        ├── /pre-push-gate
        ├── /branch-health
        ├── /review-loop
        └── /checkpoint

  Supporting:
  ┌───────────┐  ┌───────────┐  ┌───────────┐
  │ system-   │  │ system-   │  │  plugin-  │
  │ admin     │  │ ops       │  │  manager  │
  └───────────┘  └───────────┘  └───────────┘
DIAGRAM

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Run /workflow-map --format full for detailed descriptions"
echo "Run /workflow-map --scope all to include personal/knowledge agents"
```

## Output Example (Compact)

```
╔══════════════════════════════════════════════════════════════════════════╗
║                         WORKFLOW MAP                                      ║
║                    Generated: 2025-12-22 17:30                           ║
╚══════════════════════════════════════════════════════════════════════════╝

AGENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  orchestrator              opus     ✅     cyan
  gitops-devex              opus     ✅     green
  code-reviewer             sonnet   ✅     red
  tech-writer               sonnet   ✅     purple
  system-admin              sonnet   ✅     orange
  system-ops                sonnet   ✅     yellow
  repo-topology             sonnet   ✅     -
  plugin-manager            haiku    ✅     cyan
  knowledge-citadel         sonnet   ✅     amber
  scout                     sonnet   ✅     slate

SKILLS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  /worktree-create          Create isolated git worktree for feature
  /worktree-list            Show all active worktrees with status
  /worktree-cleanup         Remove worktrees for merged PRs
  /pre-commit-gate          Hard gate before commit - lint, typecheck, tests
  /pre-push-gate            Hard gate before push - build, tests, health
  /branch-health            Check branch staleness and conflict risk
  /review-loop              Autonomous PR review iteration workflow
  /latest-comment           Get latest review comment on PR
  /landscape                Quick situational awareness
  /checkpoint               Capture resumable state snapshot
  /workflow-map             This skill - generate workflow dashboard

COMMANDS (User Entry Points)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  /afk                      AFK return - shows time gap, runs stale check
  /document                 Documentation tasks using multi-agent workflow
  /extract-patterns         Extract patterns from session logs
  /surface-patterns         Surface relevant patterns for context
  /timer                    Time tracking for tasks
  /timestamp                Validated timestamps with rollover detection
  /stale                    Check if agents/skills modified since session
```

## When to Use

- **Session start**: Orient yourself to available tools
- **After changes**: Verify agents/skills are registered correctly
- **Planning**: Identify gaps before building new features
- **Auditing**: Find undocumented or redundant components
- **Onboarding**: Show someone what's available

## Integration

Can be invoked by:
- **Human**: `/workflow-map` for quick orientation
- **orchestrator**: When uncertain about capabilities
- **system-admin**: During audits and cleanup
