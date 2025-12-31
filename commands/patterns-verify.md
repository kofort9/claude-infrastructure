---
name: patterns-verify
description: "Check verification status of applied pattern fixes"
arguments: []
---

# Patterns Verify

Check statistical verification of applied improvement fixes.

## Quick Usage

```bash
# View all verification statuses
python3 ~/.claude/scripts/improvement.py --verify

# Output as JSON for filtering/scripting
python3 ~/.claude/scripts/improvement.py --verify --json
```

## Workflow

### Check Verification Status

```bash
python3 ~/.claude/scripts/improvement.py --verify
```

Output format:
```
★ Fix Verification Report ────────────────────────────────────
Fix ID       Baseline   Post-Fix   p-value    Status
──────────────────────────────────────────────────────────────
fix-abc123   0.50/d     0.10/d     0.020      ✓ verified
fix-def456   0.30/d     0.25/d     0.310      ⏳ monitoring
fix-ghi789   0.40/d     0.00/d     0.001      ✓ verified
fix-jkl012   0.20/d     0.35/d     0.500      ? inconclusive
──────────────────────────────────────────────────────────────
```

### JSON Output for Filtering

Use `--json` and pipe to `jq` for filtering:

```bash
# Get all verifications as JSON
python3 ~/.claude/scripts/improvement.py --verify --json

# Filter to verified only
python3 ~/.claude/scripts/improvement.py --verify --json | jq '.[] | select(.status == "verified")'

# Filter to still monitoring
python3 ~/.claude/scripts/improvement.py --verify --json | jq '.[] | select(.status == "monitoring")'

# Get fix with lowest p-value
python3 ~/.claude/scripts/improvement.py --verify --json | jq 'sort_by(.p_value) | .[0]'
```

Example JSON output:
```json
[
  {
    "fix_id": "abc12345",
    "check_at": "2025-12-24T10:00:00+00:00",
    "baseline_rate": 0.5,
    "post_fix_rate": 0.1,
    "sample_size": 14,
    "statistical_test": "poisson_rate",
    "p_value": 0.02,
    "status": "verified",
    "decision_reason": "rate_reduction:80%",
    "canary_passed": true,
    "full_rollout": true
  }
]
```

## Verification Logic

### Statistical Test

Uses Poisson rate comparison:

```
H0: post_rate >= baseline_rate (fix didn't help)
H1: post_rate < baseline_rate (fix improved)

p-value calculated using Poisson distribution
```

### Decision Thresholds

| Condition | Decision |
|-----------|----------|
| p < 0.05 AND rate_reduction > 0 | VERIFIED |
| p >= 0.05 AND sample_days < required | MONITORING |
| p >= 0.05 AND sample_days >= required | INCONCLUSIVE |
| rate_increase >= 1.5x baseline | FAILED |

### Auto-Rollback

If post-fix rate increases by 1.5x or more:
1. Automatic rollback is triggered
2. Fix is marked as FAILED
3. Pattern returns to manual handling
4. Suggestion is re-queued with notes

## Monitoring Periods

| Severity | Days | Rationale |
|----------|------|-----------|
| critical | 7 | Fast feedback loop |
| high | 14 | Standard verification |
| medium | 21 | More data for reliability |
| low | 30 | Extended monitoring for rare patterns |

## Integration

Called by:
- `pattern-learner` agent during weekly review
- `/patterns improve` after applying fixes
- Manual user invocation

Source data:
- `~/.claude/patterns/improvement/verification.jsonl`
- `~/.claude/patterns/improvement/applied-fixes.jsonl`
