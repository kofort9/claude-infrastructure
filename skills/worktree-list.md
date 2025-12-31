---
name: worktree-list
description: Show all active git worktrees with their status (branch, commits behind, PR state)
---

# Worktree List Skill

Displays all active worktrees for the current repository with their health status. Helps you understand what parallel work is in progress and which worktrees need attention.

## Usage

```
/worktree-list
```

## What It Shows

For each worktree:
- **Path**: Where the worktree lives
- **Branch**: What branch it's on
- **Commits behind main**: Staleness indicator
- **PR status**: Open/Draft/Merged/None
- **Last commit**: When was it last worked on
- **Health**: Green/Yellow/Red based on staleness

## Implementation

```bash
#!/bin/bash

# Get repo info
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
if [ -z "$REPO_ROOT" ]; then
  echo "Error: Not in a git repository"
  exit 1
fi

REPO_NAME=$(basename "$REPO_ROOT")
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo "main")

# Fetch latest to get accurate behind counts
git fetch origin --quiet 2>/dev/null

echo ""
echo "ACTIVE WORKTREES - $REPO_NAME"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Get all worktrees
git worktree list --porcelain | while read -r line; do
  if [[ "$line" == worktree* ]]; then
    WORKTREE_PATH="${line#worktree }"

    # Read subsequent lines for this worktree
    read -r head_line
    HEAD_SHA="${head_line#HEAD }"

    read -r branch_line
    if [[ "$branch_line" == branch* ]]; then
      BRANCH="${branch_line#branch refs/heads/}"
    else
      BRANCH="(detached)"
    fi

    # Skip the main worktree (it's the repo root)
    if [ "$WORKTREE_PATH" = "$REPO_ROOT" ]; then
      continue
    fi

    # Calculate commits behind main
    BEHIND=$(git rev-list --count "$HEAD_SHA..origin/$DEFAULT_BRANCH" 2>/dev/null || echo "?")

    # Get last commit date
    LAST_COMMIT=$(git log -1 --format="%ar" "$HEAD_SHA" 2>/dev/null || echo "unknown")

    # Determine health status
    if [ "$BEHIND" = "?" ]; then
      HEALTH="?"
    elif [ "$BEHIND" -le 5 ]; then
      HEALTH="Healthy"
    elif [ "$BEHIND" -le 15 ]; then
      HEALTH="Stale"
    else
      HEALTH="Critical"
    fi

    # Check for PR (requires gh CLI)
    PR_STATUS="None"
    if command -v gh &> /dev/null; then
      PR_INFO=$(gh pr list --head "$BRANCH" --json number,state,isDraft --jq '.[0] | "\(.number):\(.state):\(.isDraft)"' 2>/dev/null)
      if [ -n "$PR_INFO" ]; then
        PR_NUM=$(echo "$PR_INFO" | cut -d: -f1)
        PR_STATE=$(echo "$PR_INFO" | cut -d: -f2)
        PR_DRAFT=$(echo "$PR_INFO" | cut -d: -f3)

        if [ "$PR_DRAFT" = "true" ]; then
          PR_STATUS="Draft #$PR_NUM"
        elif [ "$PR_STATE" = "OPEN" ]; then
          PR_STATUS="Open #$PR_NUM"
        elif [ "$PR_STATE" = "MERGED" ]; then
          PR_STATUS="Merged #$PR_NUM"
        elif [ "$PR_STATE" = "CLOSED" ]; then
          PR_STATUS="Closed #$PR_NUM"
        fi
      fi
    fi

    # Format output
    echo ""
    echo "  $BRANCH"
    echo "  ├─ Path: $WORKTREE_PATH"
    echo "  ├─ Behind main: $BEHIND commits"
    echo "  ├─ Last commit: $LAST_COMMIT"
    echo "  ├─ PR: $PR_STATUS"
    echo "  └─ Health: $HEALTH"
  fi
done

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Commands:"
echo "  /worktree-create <name>  Create new worktree"
echo "  /worktree-cleanup        Remove merged worktrees"
echo "  /branch-health           Detailed health for current branch"
```

## Output Format

```
ACTIVE WORKTREES - trakt.tv-mcp
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  feature/oauth-refresh
  ├─ Path: /Users/you/Repos/trakt.tv-mcp-worktrees/oauth-refresh
  ├─ Behind main: 3 commits
  ├─ Last commit: 2 hours ago
  ├─ PR: Open #31
  └─ Health: Healthy

  fix/rate-limit-bug
  ├─ Path: /Users/you/Repos/trakt.tv-mcp-worktrees/rate-limit-bug
  ├─ Behind main: 12 commits
  ├─ Last commit: 3 days ago
  ├─ PR: None
  └─ Health: Stale

  feature/watchlist-sync
  ├─ Path: /Users/you/Repos/trakt.tv-mcp-worktrees/watchlist-sync
  ├─ Behind main: 0 commits
  ├─ Last commit: 10 minutes ago
  ├─ PR: Draft #32
  └─ Health: Healthy

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Commands:
  /worktree-create <name>  Create new worktree
  /worktree-cleanup        Remove merged worktrees
  /branch-health           Detailed health for current branch
```

## Health Indicators

| Status | Commits Behind | Meaning |
|--------|---------------|---------|
| Healthy | 0-5 | No action needed |
| Stale | 6-15 | Consider syncing with main |
| Critical | 16+ | Sync immediately to avoid conflicts |

## When to Use

- **Session start**: Orient yourself to active work
- **Before creating new worktree**: Check what's already in progress
- **Periodic check**: Identify stale worktrees that need attention
- **Before cleanup**: See which worktrees have merged PRs
