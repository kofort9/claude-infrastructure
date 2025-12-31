---
name: latest-comment
description: Get just the latest review comment on a PR
argument-hint: "<pr-number> [--repo owner/repo]"
---

# Latest Comment Skill

Fetch only the most recent review comment on a PR. Efficient, targeted, no bulk fetching.

## Usage

```
/latest-comment <pr-number>
/latest-comment <pr-number> --repo owner/repo
```

## Output Format

```
PR #<number> - Latest Review Comment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Author: @<username>
Date: <YYYY-MM-DD HH:MM>
File: <path>:<line>
Status: <pending | addressed | outdated>

Comment:
<full comment body>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Execution Steps

### 1. Parse Arguments
- Extract PR number (required)
- Extract --repo if provided, otherwise detect from git remote

### 2. Fetch Latest Review Comment Only
```bash
# Get just the last review comment (not issue comments)
gh api "repos/<owner>/<repo>/pulls/<pr>/comments" \
  --jq 'sort_by(.created_at) | .[-1]'
```

### 3. Determine Status
- **pending**: Comment has no reply, code unchanged at that line
- **addressed**: Code at that line was modified after comment timestamp
- **outdated**: Comment is on code that no longer exists (file deleted/moved)

### 4. Format and Display

## Edge Cases

- **No comments**: Return "No review comments on PR #<num>"
- **Only resolved comments**: Return "All comments resolved on PR #<num>"
- **API error**: Return error with suggestion to check PR number/repo

## Example Output

```
PR #28 - Latest Review Comment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Author: @greptile[bot]
Date: 2025-12-22 14:30
File: src/domain/trakt/tools.ts:1682
Status: pending

Comment:
Consider adding a defensive check for undefined effectiveType
before using it in the conditional. This could prevent silent
failures if search returns unexpected results.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Integration with Review Loop

This skill is called by `/review-loop` to get the next comment to address:

```
1. /latest-comment 28
2. If pending → address it, commit, push
3. /latest-comment 28 again
4. If same comment → already addressed, done
5. If new comment → address it, loop
```
