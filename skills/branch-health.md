---
name: branch-health
description: Check git branch health - detect conflicts, staleness, and coordination issues with open PRs
---

# Branch Health Skill

This skill provides on-demand analysis of your current git branch to prevent merge conflicts and workflow issues.

## Usage

Run `/branch-health` to get a comprehensive health report for your current branch.

## What It Checks

### 1. Branch Staleness
How far behind `main` is your branch?

| Commits Behind | Status | Recommendation |
|---------------|--------|----------------|
| 0-5 | Healthy | No action needed |
| 6-15 | Stale | Consider rebasing soon |
| 16+ | Critical | Rebase immediately to avoid conflicts |

### 2. Conflict Risk Detection
Identifies files modified in both your branch AND recent main commits:

```
⚠️ Conflict Risk: 3 files modified in both your branch and main

Files at risk:
  - .husky/pre-commit (you: 2 commits, main: 1 commit)
  - CLAUDE.md (you: 3 commits, main: 2 commits)
  - src/api/handlers.ts (you: 1 commit, main: 4 commits)

Recommendation: Merge main now to resolve while changes are small
```

### 3. PR Coordination
Checks if open PRs touch the same files you're modifying:

```
⚠️ PR Overlap Detected: 2 open PRs touch files you've modified

PR #27 "feat: add new endpoint" modifies:
  - src/api/handlers.ts (you also modified)

PR #28 "docs: update guide" modifies:
  - docs/guides/CONTRIBUTING.md (you also modified)

Recommendation: Coordinate with PR authors or wait for merge
```

### 4. Rebase Recommendations
Based on the analysis, provides actionable next steps:

- **Safe to continue**: Branch is healthy, no conflicts expected
- **Merge main soon**: Getting stale, merge before your next commit
- **Merge main now**: High conflict risk, do it immediately
- **Wait for PRs**: Overlapping PRs should merge first

## Output Format

```
╔══════════════════════════════════════════════════════════════╗
║                    BRANCH HEALTH REPORT                       ║
╠══════════════════════════════════════════════════════════════╣
║ Branch: feature/my-feature                                    ║
║ Base: main                                                    ║
║ Status: ⚠️ NEEDS ATTENTION                                    ║
╠══════════════════════════════════════════════════════════════╣
║ STALENESS                                                     ║
║   Commits behind main: 12                                     ║
║   Last sync: 3 days ago                                       ║
║   → Recommendation: Merge main before next push               ║
╠══════════════════════════════════════════════════════════════╣
║ CONFLICT RISK                                                 ║
║   Files at risk: 2                                            ║
║   - .husky/pre-push (modified in main 2 commits ago)          ║
║   - CLAUDE.md (modified in main 1 commit ago)                 ║
║   → Recommendation: Merge now, conflicts will be small        ║
╠══════════════════════════════════════════════════════════════╣
║ PR COORDINATION                                               ║
║   Overlapping PRs: 1                                          ║
║   - PR #29 touches src/utils.ts (you also modified)           ║
║   → Recommendation: Wait for PR #29 to merge first            ║
╠══════════════════════════════════════════════════════════════╣
║ RECOMMENDED ACTION                                            ║
║   git fetch origin && git merge origin/main                   ║
╚══════════════════════════════════════════════════════════════╝
```

## Implementation

When invoked, this skill runs the following checks:

```bash
# 1. Get commits behind main
git rev-list --count HEAD..origin/main

# 2. Get files changed in branch
git diff --name-only origin/main...HEAD

# 3. Get files changed in main since branch point
git diff --name-only $(git merge-base HEAD origin/main)..origin/main

# 4. Find intersection (conflict risk)
# Compare the two file lists

# 5. Check open PRs via GitHub API
# gh pr list --json number,title,files
```

## Integration with gitops-devex

This skill can be invoked by the `gitops-devex` agent when it detects potential issues, or run manually via `/branch-health`.

## When to Run

- Before starting a new feature or making significant changes
- Before pushing commits after a long pause
- When you see merge conflicts in a PR
- Daily on long-running feature branches
- After major PRs are merged to main
