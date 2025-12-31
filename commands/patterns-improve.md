---
name: patterns-improve
description: "Review and apply improvement suggestions from pattern analysis"
arguments:
  - name: action
    description: "Action: list (default), apply, reject, defer, preview"
    required: false
  - name: suggestion_id
    description: "Suggestion ID for apply/reject/defer actions"
    required: false
  - name: priority
    description: "Filter by priority: high, medium, low"
    required: false
---

# Patterns Improve

Review improvement suggestions generated from recurring correction patterns.

## Quick Usage

```bash
# List pending suggestions
python3 ~/.claude/scripts/improvement.py list

# Preview a specific suggestion
python3 ~/.claude/scripts/improvement.py preview <suggestion_id>

# Apply a suggestion
python3 ~/.claude/scripts/improvement.py apply <suggestion_id>

# Reject a suggestion
python3 ~/.claude/scripts/improvement.py reject <suggestion_id> --reason "Not applicable"

# Filter by priority
python3 ~/.claude/scripts/improvement.py list --priority high
```

## Workflow

### 1. List Pending Suggestions

```bash
python3 ~/.claude/scripts/improvement.py list
```

Output format:
```
IMPROVEMENT SUGGESTIONS

Priority  Severity  Confidence  Cluster ID                  Target
HIGH      medium    0.82        routing:agent-mismatch      orchestrator.md
MEDIUM    low       0.75        timestamp:stale-context     timestamp.md
LOW       low       0.68        security:input-validation   hooks.json

3 pending suggestions
```

### 2. Preview Suggestion

Before applying, always preview the full suggestion:

```bash
python3 ~/.claude/scripts/improvement.py preview <suggestion_id>
```

Output format:
```markdown
★ Improvement Suggestion ────────────────────────
[HIGH] severity:medium  confidence:0.82  model:sonnet

Cluster: routing:agent-mismatch (3 recurrences in 5 days)
Rationale: All patterns involve git operations routed to orchestrator
           instead of gitops-devex agent

Target: ~/.claude/agents/orchestrator.md :: Routing Rules
Summary: Add explicit git ops → gitops-devex routing rule

--- a/orchestrator.md
+++ b/orchestrator.md
@@ -45,6 +45,7 @@
+- Git operations (commit, push, branch) → gitops-devex

Source patterns:
  - correction:2025-12-20:git-commit-wrong-agent
  - correction:2025-12-22:git-push-routing-error
  - correction:2025-12-24:git-branch-misdirected

[Apply] [Reject] [Defer]
─────────────────────────────────────────────────
```

### 3. Apply Suggestion

If the suggestion looks correct:

```bash
python3 ~/.claude/scripts/improvement.py apply <suggestion_id>
```

This will:
1. Validate the patch against current file state
2. Apply the change atomically
3. Log the fix to `applied-fixes.jsonl`
4. Start verification monitoring
5. Record rollback token for recovery

### 4. Reject Suggestion

If the suggestion is incorrect or not applicable:

```bash
python3 ~/.claude/scripts/improvement.py reject <suggestion_id> --reason "Already handled manually"
```

### 5. Defer Suggestion

To review later:

```bash
python3 ~/.claude/scripts/improvement.py defer <suggestion_id>
```

## Suggestion Priority

| Priority | Based On | Action |
|----------|----------|--------|
| HIGH | severity=high/critical OR confidence>=0.9 | Review ASAP |
| MEDIUM | severity=medium AND confidence 0.7-0.9 | Review when convenient |
| LOW | severity=low OR confidence<0.7 | Optional review |

## Safety Checks

Before applying, the system verifies:

1. **File exists**: Target file must be accessible
2. **Patch applies**: Context matches current state
3. **No oscillation**: Prevents A→B→A flips within 30 days
4. **Rate limit**: Max 3 suggestions per file per day

If any check fails, application is blocked with explanation.

## After Applying

Applied fixes enter verification monitoring:

- **Critical**: 7 days
- **High**: 14 days
- **Medium**: 21 days
- **Low**: 30 days

Use `/patterns verify` to check effectiveness.

## Rollback

If an applied fix causes issues:

```bash
python3 ~/.claude/scripts/improvement.py rollback <fix_id>
```

This restores the file to its pre-fix state.

## Integration

Called by:
- `pattern-learner` agent during improvement loop
- Weekly review process
- Manual user invocation

Source data:
- `~/.claude/patterns/improvement/suggestions.jsonl`
- `~/.claude/patterns/improvement/applied-fixes.jsonl`
