---
name: review-loop
description: Autonomous PR review workflow - address latest comment, push, loop until clean
argument-hint: "<pr-number> [--repo owner/repo]"
---

# Review Loop Skill

Iterative PR review workflow. Addresses one comment at a time, pushes, checks for new comments, loops.

## Usage

```
/review-loop <pr-number>
/review-loop <pr-number> --repo owner/repo
```

## Core Loop (Latest-Comment Approach)

```
┌─────────────────────────────────────────┐
│  1. /latest-comment <pr>                │
│     ↓                                   │
│  2. No comment? → EXIT (clean)          │
│     ↓                                   │
│  3. Already addressed? → EXIT (done)    │
│     ↓                                   │
│  4. Categorize: BLOCKING|SUGGESTION|Q   │
│     ↓                                   │
│  5. QUESTION? → Respond inline, EXIT    │
│     ↓                                   │
│  6. Address comment (edit code)         │
│     ↓                                   │
│  7. Commit + Push                       │
│     ↓                                   │
│  8. Loop back to step 1                 │
└─────────────────────────────────────────┘
```

## Execution Steps

### Step 1: Initialize
```bash
# Parse PR number
PR_NUM=$1
REPO=${2:-$(git remote get-url origin | sed 's/.*github.com[:/]//' | sed 's/.git$//')}

# Checkout PR branch
gh pr checkout $PR_NUM

# Set iteration counter
ITERATION=1
MAX_ITERATIONS=5
```

### Step 2: Get Latest Comment
Use `/latest-comment <pr>` skill to fetch only the most recent review comment.

### Step 3: Check Exit Conditions
- **No comment**: PR is clean, exit with success
- **Same comment as last iteration**: Already addressed, exit
- **Max iterations reached**: Safety exit

### Step 4: Categorize Comment

| Category | Indicators | Action |
|----------|------------|--------|
| **BLOCKING** | "bug", "error", "security", "breaks", "must" | Fix immediately |
| **SUGGESTION** | "consider", "could", "might", "optional" | Fix if in-scope |
| **QUESTION** | "why", "?", "what", "how come" | Respond only |

### Step 5: Address Comment
1. Read the file at the specific line mentioned
2. Understand what change is requested
3. Make the fix using Edit tool
4. Keep change minimal and focused

### Step 6: Commit and Push
```bash
git add -A
git commit -m "Address review: <summary of fix>"
git push
```

### Step 7: Loop
Increment iteration counter, go back to Step 2.

## Scope Rules

**Address comments that are**:
- Directly about code in this PR
- Actionable with a code change
- Not superseded by newer comments

**Skip comments that are**:
- Future work suggestions
- Architecture debates
- Already fixed in newer commits

## Exit Conditions

| Condition | Action |
|-----------|--------|
| No comments | Exit: "PR clean" |
| Same comment twice | Exit: "Already addressed" |
| Only questions | Respond inline, exit |
| Iteration 5 | Safety exit with summary |
| Merge conflict | Exit with error |

## Example Run

```
> /review-loop 28

Iteration 1:
  Latest: @greptile[bot] on tools.ts:1682
  Category: SUGGESTION
  Action: Adding defensive null check
  Commit: "Address review: add null check for effectiveType"
  Push: success

Iteration 2:
  Latest: (same as before - no new comments)
  Exit: Already addressed

Summary: 1 comment addressed, PR ready for re-review
```

## Integration

- `/latest-comment` - Fetches single latest comment
- `/landscape` - View current context
- GitHub MCP - Push changes
- Git CLI - Commit operations
