---
name: review-classify
description: Parse code-reviewer output and produce an action plan with explicit next steps. Conservative human-in-the-loop approach with pattern capture for future automation.
argument-hint: "[reviewer-output-file] (optional: --auto-apply-trivial)"
---

# Review Classification Skill

Parse code-reviewer output and classify findings into actionable categories with explicit next steps. Conservative approach - human in the loop for most decisions, with pattern capture for future automation.

## Strategy: Conservative (Human in the Loop)

- **Auto-fix**: Only `[STYLE]` + `trivial` - low risk, high confidence
- **Confirm**: Everything else requiring approval - captures decisions for pattern learning
- **Escalate**: Tradeoffs and questions - always human discussion
- **Skip**: False positives - logged but no action

## Output Structure

```markdown
## REVIEW CLASSIFICATION

### Immediate (Auto-fix)
*Only [STYLE] + trivial - low risk*
- [ ] `file.ts:15` - Use const instead of let [STYLE/trivial]

### Confirm (Human approval required)
*[ACTIONABLE] items - capture decision for pattern learning*
- [ ] `file.ts:42` - Add null check [ACTIONABLE/small]
  - **Approve**: Fix and commit
  - **Modify**: Adjust the fix
  - **Reject**: Mark as false positive (captures pattern)

### Escalate (Needs discussion)
*[TRADEOFF] + [QUESTION] - always human*
- [ ] `file.ts:100` - Error handling strategy [TRADEOFF]

### Skip (No action)
- `file.ts:50` - Empty catch block [FALSE_POSITIVE] - intentional
```

## Decision Log Schema (JSONL)

Captured in `~/.claude/review-decisions.jsonl`:

```jsonl
{"timestamp":"2025-12-22T20:30:00Z","repo":"owner/repo","pr":28,"file":"src/tools.ts","line":42,"tag":"[ACTIONABLE]","effort":"small","issue":"Add null check","decision":"approved","modification":null,"pattern_hint":"null-check-optional-param","risk_tier":"low"}
```

### Schema Fields

| Field | Type | Description |
|-------|------|-------------|
| timestamp | ISO 8601 | When decision was made |
| repo | string | Repository (owner/repo) |
| pr | number | Pull request number |
| file | string | File path relative to repo |
| line | number | Line number |
| tag | string | [STYLE], [ACTIONABLE], [TRADEOFF], [QUESTION], [FALSE_POSITIVE] |
| effort | string | trivial, small, medium, large |
| issue | string | Description of the finding |
| decision | string | approved, rejected, modified |
| modification | string|null | What was changed if modified |
| pattern_hint | string | Kebab-case identifier for pattern matching |
| risk_tier | string | low, medium, high |

## Tag → Action Mapping

| Tag | Effort | Action | Risk Tier |
|-----|--------|--------|-----------|
| `[STYLE]` | trivial | Auto-fix immediately | low |
| `[STYLE]` | small+ | Confirm | low |
| `[ACTIONABLE]` | any | Confirm (capture decision) | varies |
| `[TRADEOFF]` | any | Escalate to human | medium/high |
| `[QUESTION]` | any | Generate clarifying comment | low |
| `[FALSE_POSITIVE]` | any | Skip + log | low |

## Decision Logic

```
IF tag == [STYLE] AND effort == trivial:
    → Auto-fix (Immediate)
ELSE IF tag == [ACTIONABLE]:
    → Confirm with human
ELSE IF tag IN ([TRADEOFF], [QUESTION]):
    → Escalate
ELSE IF tag == [FALSE_POSITIVE]:
    → Skip
ELSE IF tag == [STYLE]:
    → Confirm with human
END
```

## Pattern Graduation Criteria

A pattern may graduate from "Confirm" to "Auto-fix" when:

1. **Consensus**: Same pattern approved 5+ times with 0 rejections
2. **Context**: Within same repository (patterns may be repo-specific)
3. **Reliability**: >95% approval rate within its risk tier
4. **Economics**: Cost per correct decision drops when flipped to auto
5. **Stability**: Pattern stable across at least 3 different PRs
6. **Calibration**: When system says high-confidence, it's actually right

### Pattern Examples

- `null-check-optional-param`: Graduated after 8 approvals, 0 rejections
- `magic-number-extraction`: Not graduated - 5 approvals, 2 modifications
- `template-literal-preference`: Not graduated - 3 approvals, 3 rejections

## Metrics to Track

### Approval Metrics
- Approval/modify/reject rates per tag+effort combo
- Pattern stability over time
- Inter-reviewer agreement

### Accuracy Metrics
- False positive/negative rates by risk tier
- Calibration: confidence vs actual accuracy
- Regression rate: auto-fixes that required revert

### Economics Metrics
- Human-time per decision vs autonomy level
- Review velocity: time from submission to completion
- Context-switch penalty

### Quality Metrics
- Fix quality: approved fixes that improved code
- Pattern coverage: % of findings matching known patterns
- Learning rate: time to pattern graduation

## Integration Flow

```
1. code-reviewer runs → generates tagged findings
2. review-classify parses → categorizes into action plan
3. gitops-devex presents → human makes decisions
4. Decisions logged → ~/.claude/review-decisions.jsonl
5. Future: review-patterns analyzes → graduates patterns
```

## Usage

```bash
/review-classify review-output.md
/review-classify review-output.md --auto-apply-trivial
```

## Configuration

### Decision Log Location
Default: `~/.claude/review-decisions.jsonl`

Override: `export REVIEW_DECISIONS_LOG=/custom/path.jsonl`

## Future Phases

### Phase 1: Current (Human in Loop)
- Manual classification, decision capture, pattern identification

### Phase 2: Pattern Learning
- `/review-patterns` skill analyzes decision log
- Confidence scoring, graduation automation

### Phase 3: Graduated Automation
- Auto-apply graduated patterns
- Continuous calibration

### Phase 4: Full Autonomy (with Safety Rails)
- High-confidence auto-fix
- Automatic regression detection
- Pattern decay monitoring

## Safety Guardrails

### Current
- Only trivial style fixes auto-applied
- All substantive changes require human approval
- All decisions logged for audit

### Future
- Graduated patterns require validation window
- Regression detection with automatic rollback
- Pattern decay detection (stop auto-applying if rejection rate rises)

## Success Metrics

| Phase | Metric | Target |
|-------|--------|--------|
| 1 | Decision log completeness | >95% |
| 1 | Classification accuracy | >90% |
| 2-3 | Pattern graduation rate | 20-30% within 3mo |
| 2-3 | Auto-fix accuracy | >98% |
| 4 | Human time reduction | 70% |
