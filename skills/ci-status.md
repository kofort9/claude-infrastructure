---
name: ci-status
description: Get CI job status for a PR or current branch
argument-hint: "[pr-number] [--branch branch-name]"
---

# CI Status Skill

Fetch CI/CD job status for a PR or branch. Shows pass/fail/pending for each check.

## Usage

```
/ci-status              # Current branch
/ci-status <pr-number>  # Specific PR
/ci-status --branch <name>
```

## Output Format

```
CI Status: PR #<num> | <branch>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Overall: <PASS | FAIL | PENDING>

Jobs:
  Build     ✅ pass    (12s)
  Test      ✅ pass    (28s)
  Style     ✅ pass    (8s)
  Docs      ❌ fail    (17s)
  Audit     ✅ pass    (14s)

Failed Jobs Detail:
━━━━━━━━━━━━━━━━━━━━
Docs: Check markdown links
  → View: <github-actions-url>
  → Fix: Run /document to validate links

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Execution Steps

### 1. Parse Arguments
- If PR number provided, use that
- If --branch provided, find PR for that branch
- Otherwise, use current branch and find associated PR

### 2. Fetch CI Status
```bash
# For PR
gh pr checks <pr-number> 2>&1 | head -20
```

### 3. Categorize Results
- **PASS**: All checks passed
- **FAIL**: Any check failed (including Docs)
- **PENDING**: Checks still running

### 4. For Failed Jobs, Provide Fix Guidance

| Failed Check | Fix Action |
|--------------|------------|
| Build | Run `npm run build` locally, fix TS errors |
| Test | Run `npm test` locally, fix failing tests |
| Style | Run `npm run format` and `npm run lint:fix` |
| Docs | Run `/document` skill to validate/fix links |
| Audit | Run `npm audit fix` for security issues |

### 5. Get Failure Details
```bash
# Get job log for failed check
gh run view <run-id> --job <job-id> --log 2>&1 | grep -i "error\|fail" | head -20
```

## Pre-Push Checklist

Before pushing, ALL checks should pass locally:

```bash
npm run build    # TypeScript compilation
npm test         # All tests pass
npm run lint     # No lint errors
# Then run /document to check links
```

## Integration with Pre-Push Workflow

```
1. /ci-status <pr>
2. If any failures → fix locally first
3. For Docs failure → /document to find broken links
4. Push only when all checks expected to pass
```
