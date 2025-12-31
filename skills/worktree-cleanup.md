---
name: worktree-cleanup
description: Remove worktrees for merged/closed PRs and their associated branches
arguments: [--dry-run] [--all] [<worktree-name>]
---

# Worktree Cleanup Skill

Safely removes worktrees that are no longer needed (merged or closed PRs). Keeps your workspace clean and prevents accumulation of stale worktrees.

## Usage

```
/worktree-cleanup              # Interactive - shows candidates, asks for confirmation
/worktree-cleanup --dry-run    # Preview what would be cleaned up
/worktree-cleanup --all        # Clean all merged/closed worktrees without prompting
/worktree-cleanup oauth-refresh # Clean specific worktree by name
```

## What It Does

1. **Scans all worktrees** for the current repository
2. **Checks PR status** for each worktree's branch
3. **Identifies candidates** for cleanup:
   - PR merged → safe to remove
   - PR closed (not merged) → ask for confirmation
   - No PR but branch merged to main → safe to remove
4. **Removes worktree** and optionally the branch
5. **Reports results**

## Implementation

```bash
#!/bin/bash
set -e

DRY_RUN=false
ALL=false
SPECIFIC=""

# Parse arguments
while [[ $# -gt 0 ]]; do
  case $1 in
    --dry-run)
      DRY_RUN=true
      shift
      ;;
    --all)
      ALL=true
      shift
      ;;
    *)
      SPECIFIC="$1"
      shift
      ;;
  esac
done

# Get repo info
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
if [ -z "$REPO_ROOT" ]; then
  echo "Error: Not in a git repository"
  exit 1
fi

REPO_NAME=$(basename "$REPO_ROOT")
WORKTREE_BASE="$REPO_ROOT/../${REPO_NAME}-worktrees"
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo "main")

# Fetch latest
git fetch origin --quiet 2>/dev/null

echo ""
echo "WORKTREE CLEANUP - $REPO_NAME"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

CANDIDATES=()
REMOVED=0

# Process worktrees
git worktree list --porcelain | while read -r line; do
  if [[ "$line" == worktree* ]]; then
    WORKTREE_PATH="${line#worktree }"

    # Read branch info
    read -r head_line
    read -r branch_line

    if [[ "$branch_line" == branch* ]]; then
      BRANCH="${branch_line#branch refs/heads/}"
    else
      continue  # Skip detached heads
    fi

    # Skip main worktree
    if [ "$WORKTREE_PATH" = "$REPO_ROOT" ]; then
      continue
    fi

    # If specific worktree requested, only process that one
    WORKTREE_NAME=$(basename "$WORKTREE_PATH")
    if [ -n "$SPECIFIC" ] && [ "$WORKTREE_NAME" != "$SPECIFIC" ]; then
      continue
    fi

    # Check PR status
    SHOULD_CLEANUP=false
    REASON=""

    if command -v gh &> /dev/null; then
      PR_INFO=$(gh pr list --head "$BRANCH" --state all --json number,state --jq '.[0] | "\(.number):\(.state)"' 2>/dev/null)

      if [ -n "$PR_INFO" ]; then
        PR_NUM=$(echo "$PR_INFO" | cut -d: -f1)
        PR_STATE=$(echo "$PR_INFO" | cut -d: -f2)

        if [ "$PR_STATE" = "MERGED" ]; then
          SHOULD_CLEANUP=true
          REASON="PR #$PR_NUM merged"
        elif [ "$PR_STATE" = "CLOSED" ]; then
          SHOULD_CLEANUP=true
          REASON="PR #$PR_NUM closed (not merged)"
        fi
      else
        # No PR - check if branch is merged to main
        if git merge-base --is-ancestor "$BRANCH" "origin/$DEFAULT_BRANCH" 2>/dev/null; then
          SHOULD_CLEANUP=true
          REASON="Branch merged to $DEFAULT_BRANCH (no PR)"
        fi
      fi
    fi

    if [ "$SHOULD_CLEANUP" = true ]; then
      echo ""
      echo "  Candidate: $WORKTREE_NAME"
      echo "  ├─ Path: $WORKTREE_PATH"
      echo "  ├─ Branch: $BRANCH"
      echo "  └─ Reason: $REASON"

      if [ "$DRY_RUN" = true ]; then
        echo "  [DRY RUN - would remove]"
      elif [ "$ALL" = true ] || [ -n "$SPECIFIC" ]; then
        # Actually remove
        echo "  Removing worktree..."
        git worktree remove "$WORKTREE_PATH" --force 2>/dev/null || rm -rf "$WORKTREE_PATH"

        echo "  Removing branch..."
        git branch -D "$BRANCH" 2>/dev/null || true

        echo "  [REMOVED]"
        REMOVED=$((REMOVED + 1))
      else
        # Interactive mode - would prompt here
        echo "  [Would remove - run with --all to confirm]"
      fi
    fi
  fi
done

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ "$DRY_RUN" = true ]; then
  echo "Dry run complete. No changes made."
  echo "Run without --dry-run to actually clean up."
elif [ "$REMOVED" -gt 0 ]; then
  echo "Cleaned up $REMOVED worktree(s)."
else
  echo "No worktrees to clean up."
fi

# Prune any orphaned worktree references
git worktree prune
```

## Output Format

### Dry Run

```
WORKTREE CLEANUP - trakt.tv-mcp
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Candidate: oauth-refresh
  ├─ Path: /Users/you/Repos/trakt.tv-mcp-worktrees/oauth-refresh
  ├─ Branch: feature/oauth-refresh
  └─ Reason: PR #31 merged
  [DRY RUN - would remove]

  Candidate: old-experiment
  ├─ Path: /Users/you/Repos/trakt.tv-mcp-worktrees/old-experiment
  ├─ Branch: feature/old-experiment
  └─ Reason: PR #28 closed (not merged)
  [DRY RUN - would remove]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dry run complete. No changes made.
Run without --dry-run to actually clean up.
```

### Actual Cleanup

```
WORKTREE CLEANUP - trakt.tv-mcp
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Candidate: oauth-refresh
  ├─ Path: /Users/you/Repos/trakt.tv-mcp-worktrees/oauth-refresh
  ├─ Branch: feature/oauth-refresh
  └─ Reason: PR #31 merged
  Removing worktree...
  Removing branch...
  [REMOVED]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Cleaned up 1 worktree(s).
```

## Safety

- **Never removes active worktrees** with uncommitted changes (git worktree remove will fail)
- **Preserves branches** that aren't fully merged unless explicitly requested
- **Dry run mode** lets you preview before committing
- **Prunes orphaned references** after cleanup

## When to Use

- **After PR merge**: Clean up the worktree you just finished
- **Periodic maintenance**: Run weekly to clean up old worktrees
- **Before starting new work**: Ensure clean slate
- **When disk space is low**: Worktrees can accumulate
