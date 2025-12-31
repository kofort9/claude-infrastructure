---
name: landscape
description: Show current work landscape - branch, repo, PRs, latest activity, git status
---

# Landscape Skill

Quick situational awareness view of current work context.

## Usage

```
/landscape
```

## Output Format

```
=== LANDSCAPE ===

Branch: <current-branch>
Repo: <owner/repo>
Remote: <origin-url>

Open PRs:
  #<num> [<branch>] <title>
  ...

Latest Review Comment:
  PR #<num>: @<author> (<date>)
  > <first line of comment>
  File: <path>:<line>

Git Status:
  <clean | N files modified>

Active Work:
  <from todos if any>
```

## Execution Steps

When invoked, gather this information:

### 1. Git Context
```bash
# Current branch
git branch --show-current

# Repo name from remote
git remote get-url origin | sed 's/.*github.com[:/]//' | sed 's/.git$//'
```

### 2. Open PRs
```bash
gh pr list --state open --json number,title,headRefName --jq '.[] | "#\(.number) [\(.headRefName)] \(.title)"'
```

### 3. Latest Review Comment (most recent across open PRs)
```bash
# Get the current branch's PR number first
PR_NUM=$(gh pr view --json number --jq '.number' 2>/dev/null)

# If on a PR branch, get its latest review comment
if [ -n "$PR_NUM" ]; then
  gh api "repos/{owner}/{repo}/pulls/${PR_NUM}/comments" \
    --jq 'sort_by(.created_at) | .[-1] | "@\(.user.login) (\(.created_at | split("T")[0])): \(.body | split("\n")[0]) [File: \(.path):\(.line // .original_line)]"'
fi
```

### 4. Git Status
```bash
# Count modified files
MODIFIED=$(git status --porcelain | wc -l | tr -d ' ')
if [ "$MODIFIED" = "0" ]; then
  echo "clean"
else
  echo "$MODIFIED files modified"
fi
```

### 5. Active Todos
Check current todo list state and summarize active items.

## Example Output

```
=== LANDSCAPE ===

Branch: feat/smart-type-inference
Repo: youruser/trakt.tv-mcp
Remote: github.com

Open PRs:
  #28 [feat/smart-type-inference] Phase 0: Smart type inference
  #27 [feat/observability] feat(observability): add Langfuse spans

Latest Review Comment:
  PR #28: @greptile[bot] (2025-12-22)
  > Consider adding error handling for edge case
  File: src/domain/trakt/tools.ts:1682

Git Status:
  clean

Active Work:
  - Testing agentic review loop
```

## Integration

- Works with any git repository
- Uses `gh` CLI for GitHub operations
- Falls back gracefully if not in a PR context
