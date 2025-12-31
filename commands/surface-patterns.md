---
name: surface-patterns
description: "Surface relevant patterns for current context"
arguments:
  - name: query
    description: "Keywords or context to find relevant patterns"
    required: false
  - name: domain
    description: "Filter by domain (workflow, architecture, security, discovery)"
    required: false
  - name: type
    description: "Filter by type (success, correction, novel, principle)"
    required: false
---

# Surface Patterns

Find and surface relevant patterns from the pattern registry.

## Quick Usage

```bash
# Find patterns by keywords
python3 ~/.claude/scripts/pattern_query.py --keywords "batch parallel"

# Find patterns similar to a context
python3 ~/.claude/scripts/pattern_query.py --similar "implementing a new hook"

# Filter by domain
python3 ~/.claude/scripts/pattern_query.py --domain architecture --limit 5

# Find corrections (for error handling)
python3 ~/.claude/scripts/pattern_query.py --type correction

# Recent patterns (last 7 days)
python3 ~/.claude/scripts/pattern_query.py --recent 7

# Full stats
python3 ~/.claude/scripts/pattern_query.py --stats
```

## Workflow

### 1. Determine Context

If user provided a query, use it. Otherwise, infer from:
- Current task being discussed
- Recent errors or challenges
- Planning context

### 2. Run Query

```bash
python3 ~/.claude/scripts/pattern_query.py --similar "[context]" --limit 5
```

### 3. Format Results

Present as Pattern Insight block:

```markdown
`★ Pattern Insight ─────────────────────────────`
**Relevant patterns for "[context]":**

1. **[domain/type]** [observation]
   → Suggests: [actionable insight]

2. **[domain/type]** [observation]
   → Suggests: [actionable insight]
`───────────────────────────────────────────────`
```

### 4. Actionable Recommendations

Based on patterns found, suggest:
- Approaches that worked before
- Pitfalls to avoid (from corrections)
- Relevant techniques (from novel/discovery)

## When to Use

### Automatically (inline in other skills)

- `/plan` - Surface architectural patterns before planning
- Error handling - Surface corrections when errors occur
- New feature work - Surface similar past successes

### Manually

- "What patterns do I have about X?"
- "How have I solved similar problems?"
- "Show me my workflow patterns"

## Pattern Types

| Type | What It Tells You |
|------|-------------------|
| `success` | What worked well - repeat this |
| `correction` | What was wrong and how it was fixed - avoid the mistake |
| `novel` | New discovery - consider applying |
| `principle` | Rule or discipline - follow this |

## Confidence Levels

- **>= 0.9**: High confidence, strongly recommend
- **0.6-0.9**: Medium confidence, consider applying
- **0.3-0.6**: Low confidence, use with judgment
- **< 0.3**: Noise, usually filtered out

## Integration Points

This skill should be called by:
- `pattern-learner` agent for reporting
- `/plan` skill for planning context
- Error handlers for correction lookup
- `orchestrator` for routing decisions
