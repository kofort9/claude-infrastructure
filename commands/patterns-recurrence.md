---
name: patterns-recurrence
description: "View pattern recurrence clusters and escalation status"
arguments: []
---

# Patterns Recurrence

View recurrence clusters to understand which patterns are repeating.

## Quick Usage

Recurrence tracking is integrated into pattern logging. View clusters via:

```bash
# Show recurrence info when logging a pattern
python3 ~/.claude/scripts/log_pattern.py --domain workflow --type correction \
  --observation "Test" --show-recurrence --dry-run

# View raw recurrence index
cat ~/.claude/patterns/improvement/recurrence-index.jsonl | jq

# Count clusters by escalation status
cat ~/.claude/patterns/improvement/recurrence-index.jsonl | \
  jq -s 'group_by(.escalated) | map({escalated: .[0].escalated, count: length})'
```

## Viewing Clusters

### List All Clusters

```bash
# View all clusters (formatted)
cat ~/.claude/patterns/improvement/recurrence-index.jsonl | \
  jq -r '"\(.cluster_id)\t\(.occurrence_count)\t\(.severity)\t\(.escalated)"' | \
  column -t -s $'\t'
```

Example output:
```
cluster_id                    count  severity  escalated
routing:agent-mismatch        3      medium    true
timestamp:stale-context       2      low       false
security:external-content     1      high      false
```

### Filter by Domain

```bash
cat ~/.claude/patterns/improvement/recurrence-index.jsonl | \
  jq 'select(.domain == "workflow")'
```

### Filter to Escalated Only

```bash
cat ~/.claude/patterns/improvement/recurrence-index.jsonl | \
  jq 'select(.escalated == true)'
```

### View Cluster Details

```bash
cat ~/.claude/patterns/improvement/recurrence-index.jsonl | \
  jq 'select(.cluster_id == "routing:agent-mismatch")'
```

Example cluster record:
```json
{
  "cluster_id": "routing:agent-mismatch",
  "canonical_pattern_id": "correction:2025-12-20:git-commit",
  "canonical_observation": "Git operations routed to orchestrator instead of gitops-devex",
  "member_ids": ["id1", "id2", "id3"],
  "occurrence_count": 3,
  "effective_count": 2.8,
  "occurrence_rate": 0.5,
  "first_seen_at": "2025-12-20T14:30:00Z",
  "last_seen_at": "2025-12-24T09:45:00Z",
  "severity": "medium",
  "recurrence_risk": "high",
  "cluster_confidence": 0.85,
  "escalated": true,
  "escalated_at": "2025-12-24T10:00:00Z"
}
```

## Understanding Clusters

### How Clusters Form

Patterns are grouped using **hybrid similarity** (2-of-3 voting):

| Method | Threshold | Weight |
|--------|-----------|--------|
| Jaccard (lexical) | 0.60 | Word overlap |
| TF-IDF Cosine | 0.55 | Term importance |
| Embedding | 0.70 | Semantic similarity |

A pattern joins a cluster if 2+ methods exceed their thresholds.

### Decay-Weighted Counting

Recent patterns count more than old ones:

```
weight = exp(-days_ago * ln(2) / 14)  # 14-day half-life

Day 0:  weight = 1.0
Day 7:  weight = 0.71
Day 14: weight = 0.50
Day 28: weight = 0.25
```

This means:
- Pattern from yesterday: counts as 1.0
- Pattern from 2 weeks ago: counts as 0.5
- Very old patterns: negligible weight

### Escalation Thresholds

| Severity | Base Threshold | High Risk (0.5x) | Medium Risk (0.75x) |
|----------|----------------|------------------|---------------------|
| critical | 1 | 1 | 1 |
| high | 2 | 1 | 2 |
| medium | 3 | 2 | 3 |
| low | 5 | 3 | 4 |

When effective_count >= threshold, cluster escalates and triggers suggestion.

### Cluster Confidence

Average similarity score of all members to the canonical pattern.

| Confidence | Meaning |
|------------|---------|
| >= 0.9 | Very tight cluster, high certainty |
| 0.7-0.9 | Good cluster, reliable |
| 0.6-0.7 | Acceptable, might be noisy |
| < 0.6 | Weak cluster, may contain unrelated patterns |

## Actions on Clusters

- **View suggestion**: If escalated, use `/patterns improve` to see the suggestion
- **Increase severity**: If cluster is important, manually set higher severity
- **Add pattern**: When you notice a similar pattern, it auto-joins the cluster

## Integration

Called by:
- `pattern-learner` agent for reporting
- Weekly review process
- Manual user invocation

Source data:
- `~/.claude/patterns/improvement/recurrence-index.jsonl`
- `~/.claude/patterns/recovery/corrections.jsonl`
