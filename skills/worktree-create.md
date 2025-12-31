---
name: worktree-create
description: Create an isolated git worktree for a new feature/fix. Returns the workspace path for all subsequent work.
arguments: <feature-name> [--type feature|fix|chore|docs]
---

# Worktree Create Skill

Creates an isolated git worktree for parallel feature development. Each worktree has its own working directory, branch, and git state - no stashing, no context switching, no collisions.

## Usage

```
/worktree-create oauth-refresh
/worktree-create rate-limit-bug --type fix
/worktree-create update-deps --type chore
```

## What It Does

1. **Validates main repo state**
   - Ensures you're in a git repository
   - Fetches latest from origin
   - Verifies main/master is up-to-date

2. **Creates worktree structure**
   ```
   ~/Repos/{repo}/                          # Main repo (stays on main)
   ~/Repos/{repo}-worktrees/{feature-name}/ # New isolated workspace
   ```

3. **Sets up the branch**
   - Creates branch: `{type}/{feature-name}` (e.g., `feature/oauth-refresh`)
   - Bases it off latest main/master
   - Checks out in the new worktree

4. **Runs initial health check**
   - Verifies clean starting state
   - Confirms 0 commits behind main

5. **Creates PR_SCOPE.md**
   - Template file defining what this PR will contain
   - You fill in the scope before coding

6. **Returns workspace path**
   - All subsequent work happens in this directory

## Implementation

```bash
#!/bin/bash
set -e

FEATURE_NAME="$1"
TYPE="${2:-feature}"  # Default to feature

# Validate inputs
if [ -z "$FEATURE_NAME" ]; then
  echo "Error: Feature name required"
  echo "Usage: /worktree-create <feature-name> [--type feature|fix|chore|docs]"
  exit 1
fi

# Get repo info
REPO_ROOT=$(git rev-parse --show-toplevel)
REPO_NAME=$(basename "$REPO_ROOT")
WORKTREE_BASE="$REPO_ROOT/../${REPO_NAME}-worktrees"
WORKTREE_PATH="$WORKTREE_BASE/$FEATURE_NAME"
BRANCH_NAME="$TYPE/$FEATURE_NAME"

# Ensure we're on main and up-to-date
echo "Fetching latest from origin..."
git fetch origin

CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo "main")

# Check if worktree already exists
if [ -d "$WORKTREE_PATH" ]; then
  echo "Error: Worktree already exists at $WORKTREE_PATH"
  echo "Use /worktree-list to see active worktrees"
  exit 1
fi

# Check if branch already exists
if git show-ref --verify --quiet "refs/heads/$BRANCH_NAME"; then
  echo "Error: Branch $BRANCH_NAME already exists"
  exit 1
fi

# Create worktree directory structure
mkdir -p "$WORKTREE_BASE"

# Create worktree with new branch based on origin/main
echo "Creating worktree at $WORKTREE_PATH..."
git worktree add "$WORKTREE_PATH" -b "$BRANCH_NAME" "origin/$DEFAULT_BRANCH"

# Create PR_SCOPE.md template
cat > "$WORKTREE_PATH/PR_SCOPE.md" << 'EOF'
# PR Scope Definition

## Summary
<!-- One-line description of what this PR does -->

## Changes
<!-- List of files/modules to be modified -->
- [ ]

## Acceptance Criteria
<!-- What must be true for this PR to be complete -->
- [ ]

## Out of Scope
<!-- What this PR explicitly does NOT include -->
-

## Testing Plan
<!-- How will you verify these changes work? -->
- [ ]

---
*This file is created by /worktree-create and should be filled in before coding.*
*Delete this file before creating the PR (or add to .gitignore).*
EOF

# Output success
echo ""
echo "Worktree created successfully!"
echo ""
echo "Workspace: $WORKTREE_PATH"
echo "Branch: $BRANCH_NAME"
echo "Based on: origin/$DEFAULT_BRANCH"
echo ""
echo "Next steps:"
echo "  1. cd $WORKTREE_PATH"
echo "  2. Fill in PR_SCOPE.md to define what this PR will contain"
echo "  3. Start coding!"
echo ""
echo "When done:"
echo "  /pre-commit-gate  # Validate before commit"
echo "  /pre-push-gate    # Validate before push"
echo "  /worktree-cleanup # After PR is merged"
```

## Output Format

```
Worktree created successfully!

Workspace: /Users/you/Repos/trakt.tv-mcp-worktrees/oauth-refresh
Branch: feature/oauth-refresh
Based on: origin/main

Next steps:
  1. cd /Users/you/Repos/trakt.tv-mcp-worktrees/oauth-refresh
  2. Fill in PR_SCOPE.md to define what this PR will contain
  3. Start coding!

When done:
  /pre-commit-gate  # Validate before commit
  /pre-push-gate    # Validate before push
  /worktree-cleanup # After PR is merged
```

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| "Not in a git repository" | Run from non-git directory | cd to a git repo first |
| "Worktree already exists" | Feature name already in use | Choose different name or use existing worktree |
| "Branch already exists" | Branch name collision | Delete old branch or choose different name |
| "Failed to fetch" | Network/auth issue | Check git credentials |

## Integration

This skill can be invoked by:
- **gitops-devex agent**: Primary user - when starting new feature work
- **orchestrator**: When planning multi-feature work
- **system-admin agent**: When setting up development environments or automation
- **system-ops agent**: Quick worktree creation for operational tasks

After creation, all work in the worktree uses the same gitops-devex agent but operates in the isolated workspace.
