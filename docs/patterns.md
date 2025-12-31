# Pattern Learning System

A system for observing Claude's decisions and graduating high-confidence patterns to automation.

## Overview

The pattern system learns from:
- Successful workflows
- Corrections and fixes
- Novel approaches
- Recurring decisions

Over time, patterns that consistently work can be "graduated" to automatic application.

## Pattern Types

### Success Patterns
Things that worked well:
- Effective prompts
- Useful agent combinations
- Workflow sequences

### Correction Patterns
Mistakes and their fixes:
- Common errors
- Better approaches discovered
- Anti-patterns to avoid

### Novel Patterns
New discoveries:
- First-time solutions
- Creative approaches
- Unexpected insights

### Principle Patterns
Guiding rules:
- "Always X before Y"
- "Never do Z in context W"
- Design philosophies

## Pattern Lifecycle

```
1. OBSERVATION
   └─ Decision logged with context

2. CLUSTERING
   └─ Similar patterns grouped

3. CONFIDENCE
   └─ Track success rate over time

4. GRADUATION
   └─ High-confidence → automatic
   └─ Human review required

5. VALIDATION
   └─ Verify graduated patterns work
```

## Storage

Patterns stored in JSONL format:

```json
{
  "id": "domain:name:date",
  "domain": "workflow|architecture|security|discovery",
  "pattern_type": "success|correction|novel|principle",
  "timestamp": "2025-01-15T10:30:00Z",
  "observation": "Description of what happened",
  "context": "Where this occurred",
  "confidence": 0.85,
  "source": "manual|auto|log-dualwrite"
}
```

## Domains

- **workflow** - Process and automation patterns
- **architecture** - System design patterns
- **security** - Safety and validation patterns
- **discovery** - Novel approaches and insights

## Graduation Criteria

A pattern graduates when:
1. Observed 5+ times
2. Success rate > 90%
3. Human review approved
4. No recent failures

## Integration Points

### During Logging (`/log`)
Entries are classified for pattern indicators:
- Domain keywords trigger logging
- Dual-write to pattern files

### During Sessions
Pattern-aware agents can:
- Reference known patterns
- Suggest based on context
- Learn from corrections

### At Session End
Sync hook:
- Aggregates session patterns
- Updates confidence scores
- Triggers graduation checks

## Example Patterns

**Workflow Success:**
```json
{
  "domain": "workflow",
  "pattern_type": "success",
  "observation": "Parallel agent execution reduced task time 3x",
  "confidence": 0.92
}
```

**Correction:**
```json
{
  "domain": "architecture",
  "pattern_type": "correction",
  "observation": "Sync from public/ subdirs, not root with exclusions",
  "confidence": 0.95
}
```

## Privacy

- Pattern files may contain context
- Runtime JSONL files are gitignored
- Only config/thresholds are tracked
- Public sync excludes pattern data
