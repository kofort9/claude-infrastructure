---
name: pre-commit-gate
description: Hard gate before any commit - runs lint, typecheck, and tests. Blocks commit if any fail.
---

# Pre-Commit Gate Skill

A **hard gate** that must pass before any commit is allowed. No exceptions. No bypass. If this gate fails, the commit is blocked until issues are fixed.

## Usage

```
/pre-commit-gate
```

## Philosophy

**Catch problems early.** A failing lint check caught now saves a broken build later. A failing test caught now saves a debugging session later.

This is not a suggestion - it's a gate. Pass it or fix your code.

## What It Runs

Runs checks in order, **stopping on first failure**:

1. **Lint Check** - Code style and potential errors
2. **Type Check** - Type safety (TypeScript/Python/etc.)
3. **Unit Tests** - Fast, focused tests

```bash
# The checks (project-specific commands detected automatically)
npm run lint        # or: eslint, ruff, flake8, etc.
npm run typecheck   # or: tsc --noEmit, mypy, pyright
npm test           # or: pytest, jest, vitest
```

## Implementation

```bash
#!/bin/bash
set -e

echo ""
echo "PRE-COMMIT GATE"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# Detect project type and available commands
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

# Detect package manager and available scripts
if [ -f "package.json" ]; then
  # Node.js project
  PKG_MANAGER="npm"
  [ -f "pnpm-lock.yaml" ] && PKG_MANAGER="pnpm"
  [ -f "yarn.lock" ] && PKG_MANAGER="yarn"
  [ -f "bun.lockb" ] && PKG_MANAGER="bun"

  # Check which scripts are available
  SCRIPTS=$(cat package.json | grep -o '"[^"]*":' | tr -d '":' || true)

  # Lint
  if echo "$SCRIPTS" | grep -q "^lint$"; then
    run_check "Lint" "$PKG_MANAGER run lint" || true
  elif echo "$SCRIPTS" | grep -q "^eslint$"; then
    run_check "Lint" "$PKG_MANAGER run eslint" || true
  fi

  # Type check
  if echo "$SCRIPTS" | grep -q "^typecheck$"; then
    run_check "Type Check" "$PKG_MANAGER run typecheck" || true
  elif echo "$SCRIPTS" | grep -q "^type-check$"; then
    run_check "Type Check" "$PKG_MANAGER run type-check" || true
  elif [ -f "tsconfig.json" ]; then
    run_check "Type Check" "npx tsc --noEmit" || true
  fi

  # Tests
  if echo "$SCRIPTS" | grep -q "^test$"; then
    run_check "Tests" "$PKG_MANAGER run test" || true
  elif echo "$SCRIPTS" | grep -q "^test:unit$"; then
    run_check "Tests" "$PKG_MANAGER run test:unit" || true
  fi

elif [ -f "pyproject.toml" ] || [ -f "setup.py" ] || [ -f "requirements.txt" ]; then
  # Python project

  # Lint
  if command -v ruff &> /dev/null; then
    run_check "Lint (ruff)" "ruff check ." || true
  elif command -v flake8 &> /dev/null; then
    run_check "Lint (flake8)" "flake8 ." || true
  fi

  # Type check
  if command -v mypy &> /dev/null; then
    run_check "Type Check (mypy)" "mypy ." || true
  elif command -v pyright &> /dev/null; then
    run_check "Type Check (pyright)" "pyright" || true
  fi

  # Tests
  if command -v pytest &> /dev/null; then
    run_check "Tests (pytest)" "pytest -x -q" || true
  fi

elif [ -f "Cargo.toml" ]; then
  # Rust project
  run_check "Lint (clippy)" "cargo clippy -- -D warnings" || true
  run_check "Tests" "cargo test" || true

elif [ -f "go.mod" ]; then
  # Go project
  run_check "Lint (vet)" "go vet ./..." || true
  run_check "Tests" "go test ./..." || true
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
  echo "Fix these issues before committing."
  echo "DO NOT bypass this gate."
  exit 1
else
  echo "GATE STATUS: PASS ($CHECKS_PASSED checks passed)"
  echo ""
  echo "Ready to commit!"
  exit 0
fi
```

## Output Format

### All Checks Pass

```
PRE-COMMIT GATE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Lint... PASS
  Type Check... PASS
  Tests... PASS

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GATE STATUS: PASS (3 checks passed)

Ready to commit!
```

### Check Fails

```
PRE-COMMIT GATE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Lint... PASS
  Type Check... FAIL

  Error output:
    src/auth.ts:42:5 - error TS2322: Type 'string' is not assignable to type 'number'.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GATE STATUS: BLOCKED

Failed checks:
  - Type Check

Fix these issues before committing.
DO NOT bypass this gate.
```

## No Bypass Policy

This gate has **no escape hatch**. If you think you need to bypass it:

1. **Fix the issue** - That's the only valid path
2. **If truly urgent** - Commit manually with `git commit` but know that:
   - Pre-push gate will likely catch it
   - CI will definitely catch it
   - You're accumulating debt

If you find yourself wanting to bypass frequently, that's a signal the checks are too strict or too slow - adjust the checks, don't bypass them.

## Integration

This skill is invoked by:
- **gitops-devex agent**: Before every commit
- **system-admin agent**: When setting up commit workflows
- **system-ops agent**: Manual validation

The commit should ONLY proceed if this gate returns exit code 0.
