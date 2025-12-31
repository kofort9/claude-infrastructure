---
name: pre-push-gate
description: Hard gate before any push - runs full build, all tests, checks branch health. Blocks push if any fail. NEVER allows force push.
---

# Pre-Push Gate Skill

A **hard gate** that must pass before any push is allowed. This is the final checkpoint before code leaves your machine. More comprehensive than pre-commit because this code is about to be shared.

**CRITICAL: Force push is NEVER allowed through this gate.**

## Usage

```
/pre-push-gate
```

## Philosophy

**The last line of defense.** If broken code gets pushed, it affects the whole team. CI will catch it, but that wastes time and money. Catch it here.

Force push destroys history and can cause data loss for collaborators. It's never worth the risk.

## What It Runs

Runs checks in order, **stopping on first failure**:

1. **Build** - Full production build
2. **All Tests** - Unit + integration tests
3. **Lint** - Final style check
4. **Branch Health** - Not too far behind main
5. **Force Push Detection** - BLOCK any force push attempt

```bash
# The checks
npm run build       # Full build
npm test           # All tests
npm run lint       # Style check
# Plus: branch staleness check
# Plus: force push detection
```

## Implementation

```bash
#!/bin/bash
set -e

echo ""
echo "PRE-PUSH GATE"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# ===== FORCE PUSH DETECTION =====
# Check if this is being called as part of a force push
if [[ "${GIT_PUSH_OPTION:-}" == *"force"* ]] || \
   [[ "$*" == *"--force"* ]] || \
   [[ "$*" == *"-f"* ]] || \
   [[ "$*" == *"--force-with-lease"* ]]; then
  echo "FORCE PUSH DETECTED"
  echo ""
  echo "Force push is NEVER allowed through this gate."
  echo ""
  echo "Alternatives:"
  echo "  1. Create a new branch with clean history"
  echo "  2. Use 'git revert' to undo commits"
  echo "  3. If absolutely necessary, run git push manually (not recommended)"
  echo ""
  echo "GATE STATUS: BLOCKED (force push forbidden)"
  exit 1
fi

# ===== BRANCH HEALTH CHECK =====
echo "Checking branch health..."
echo ""

git fetch origin --quiet 2>/dev/null || true

DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo "main")
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

# Don't allow pushing directly to main
if [ "$CURRENT_BRANCH" = "$DEFAULT_BRANCH" ] || [ "$CURRENT_BRANCH" = "master" ]; then
  echo "  Direct push to $CURRENT_BRANCH... BLOCKED"
  echo ""
  echo "  Never push directly to $CURRENT_BRANCH."
  echo "  Create a feature branch and open a PR instead."
  echo ""
  echo "GATE STATUS: BLOCKED (direct push to protected branch)"
  exit 1
fi

BEHIND=$(git rev-list --count HEAD..origin/$DEFAULT_BRANCH 2>/dev/null || echo "0")

echo -n "  Commits behind $DEFAULT_BRANCH: $BEHIND... "

if [ "$BEHIND" -gt 25 ]; then
  echo "BLOCKED"
  echo ""
  echo "  Branch is $BEHIND commits behind $DEFAULT_BRANCH."
  echo "  This is too far - merge conflicts are likely."
  echo ""
  echo "  Run: git fetch origin && git merge origin/$DEFAULT_BRANCH"
  echo ""
  echo "GATE STATUS: BLOCKED (branch too stale)"
  exit 1
elif [ "$BEHIND" -gt 15 ]; then
  echo "WARNING"
  echo "  Consider merging $DEFAULT_BRANCH soon."
else
  echo "OK"
fi

echo ""

# ===== CODE CHECKS =====
CHECKS_PASSED=0
CHECKS_FAILED=0
FAILED_CHECKS=()

run_check() {
  local NAME="$1"
  local COMMAND="$2"

  echo -n "  $NAME... "

  if eval "$COMMAND" > /tmp/gate-output.txt 2>&1; then
    echo "PASS"
    CHECKS_PASSED=$((CHECKS_PASSED + 1))
    return 0
  else
    echo "FAIL"
    CHECKS_FAILED=$((CHECKS_FAILED + 1))
    FAILED_CHECKS+=("$NAME")
    echo ""
    echo "  Error output:"
    sed 's/^/    /' /tmp/gate-output.txt | head -50
    echo ""
    return 1
  fi
}

# Detect project type and run checks
if [ -f "package.json" ]; then
  PKG_MANAGER="npm"
  [ -f "pnpm-lock.yaml" ] && PKG_MANAGER="pnpm"
  [ -f "yarn.lock" ] && PKG_MANAGER="yarn"
  [ -f "bun.lockb" ] && PKG_MANAGER="bun"

  SCRIPTS=$(cat package.json | grep -o '"[^"]*":' | tr -d '":' || true)

  # Build (required for push)
  if echo "$SCRIPTS" | grep -q "^build$"; then
    run_check "Build" "$PKG_MANAGER run build" || true
  fi

  # All tests
  if echo "$SCRIPTS" | grep -q "^test$"; then
    run_check "Tests" "$PKG_MANAGER run test" || true
  fi

  # Lint
  if echo "$SCRIPTS" | grep -q "^lint$"; then
    run_check "Lint" "$PKG_MANAGER run lint" || true
  fi

elif [ -f "pyproject.toml" ] || [ -f "setup.py" ]; then
  # Python - run full test suite
  if command -v pytest &> /dev/null; then
    run_check "Tests (pytest)" "pytest" || true
  fi
  if command -v ruff &> /dev/null; then
    run_check "Lint (ruff)" "ruff check ." || true
  fi
  if command -v mypy &> /dev/null; then
    run_check "Type Check (mypy)" "mypy ." || true
  fi

elif [ -f "Cargo.toml" ]; then
  run_check "Build" "cargo build --release" || true
  run_check "Tests" "cargo test" || true
  run_check "Lint (clippy)" "cargo clippy -- -D warnings" || true

elif [ -f "go.mod" ]; then
  run_check "Build" "go build ./..." || true
  run_check "Tests" "go test ./..." || true
  run_check "Lint (vet)" "go vet ./..." || true
fi

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

if [ $CHECKS_FAILED -gt 0 ]; then
  echo "GATE STATUS: BLOCKED"
  echo ""
  echo "Failed checks:"
  for check in "${FAILED_CHECKS[@]}"; do
    echo "  - $check"
  done
  echo ""
  echo "Fix these issues before pushing."
  echo "DO NOT bypass this gate."
  exit 1
else
  echo "GATE STATUS: PASS"
  echo ""
  echo "Summary:"
  echo "  - $CHECKS_PASSED checks passed"
  echo "  - Branch is ${BEHIND:-0} commits behind $DEFAULT_BRANCH"
  echo ""
  echo "Ready to push!"
  exit 0
fi
```

## Output Format

### All Checks Pass

```
PRE-PUSH GATE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Checking branch health...

  Commits behind main: 3... OK

  Build... PASS
  Tests... PASS
  Lint... PASS

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GATE STATUS: PASS

Summary:
  - 3 checks passed
  - Branch is 3 commits behind main

Ready to push!
```

### Force Push Blocked

```
PRE-PUSH GATE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FORCE PUSH DETECTED

Force push is NEVER allowed through this gate.

Alternatives:
  1. Create a new branch with clean history
  2. Use 'git revert' to undo commits
  3. If absolutely necessary, run git push manually (not recommended)

GATE STATUS: BLOCKED (force push forbidden)
```

### Branch Too Stale

```
PRE-PUSH GATE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Checking branch health...

  Commits behind main: 28... BLOCKED

  Branch is 28 commits behind main.
  This is too far - merge conflicts are likely.

  Run: git fetch origin && git merge origin/main

GATE STATUS: BLOCKED (branch too stale)
```

## Force Push Policy

**Force push is FORBIDDEN. No exceptions.**

Why:
- Destroys shared history that others may have pulled
- Can cause data loss for collaborators
- Makes debugging harder (commits disappear)
- Usually indicates a workflow problem upstream

If you find yourself needing force push:
1. **You rebased a pushed branch** → Don't rebase pushed branches
2. **You need to fix a commit** → Use `git revert` instead
3. **You made a mess** → Create a new branch with clean history

## Integration

This skill is invoked by:
- **gitops-devex agent**: Before every push (mandatory)
- **system-admin agent**: When setting up push workflows
- **system-ops agent**: Manual validation before pushing

The push should ONLY proceed if this gate returns exit code 0.
