---
name: repo-documentation
description: Multi-agent skill for maintaining high-quality, situationally aware repository documentation. Creates, updates, and validates docs using parallel agent workflows.
---

# Repo Documentation Skill

Orchestrates multiple agents to create and maintain comprehensive repository documentation. Optimizes for human readers, downstream agents (RAG, code assistants), and long-term maintainability.

## When to Use This Skill

Use this skill when you need to:
- Create new documentation from scratch (features, APIs, guides)
- Update existing docs after code changes
- Audit documentation for accuracy and completeness
- Generate structured docs with machine-readable metadata
- Run iterative documentation maintenance workflows
- Document experiments, failures, and lessons learned

## Core Principles

1. **Accuracy over eloquence**: Documentation must reflect current code state
2. **Structure for machines**: Emit metadata that RAG systems can consume
3. **Consistency enforcement**: All docs follow the same patterns
4. **Situational awareness**: Know what exists before creating new content
5. **Preserve learnings**: Capture experiments and failures, not just successes

---

## Document Types

### 1. README / Overview
**Purpose**: Primary entry point for the repository

**Required Sections**:
- Purpose & high-level summary
- Key components / architecture
- Getting started (install, run, basic usage)
- Configuration & environment variables
- Common workflows / examples
- Links to deeper documentation

### 2. API Reference
**Purpose**: Complete specification of public interfaces

**Required Sections**:
- Function/endpoint signature
- Parameters & types (with constraints)
- Return values & error codes
- Examples (request/response or usage snippets)
- Rate limits, side effects, and notes

### 3. Design / ADR / Spec
**Purpose**: Capture architectural decisions and rationale

**Required Sections**:
- Context & problem statement
- Alternatives considered
- Decision & rationale
- Implications / tradeoffs
- Future work considerations

### 4. How-to / Tutorial
**Purpose**: Step-by-step guidance for specific tasks

**Required Sections**:
- Audience and prerequisites
- Step-by-step instructions
- Expected outcomes at each step
- Common pitfalls and troubleshooting

### 5. Operations / Runbook
**Purpose**: Operational guidance for running the system

**Required Sections**:
- Setup and configuration
- Monitoring and observability
- Troubleshooting common issues
- Incident response procedures

### 6. Case Study
**Purpose**: Document notable experiments, problems discovered, or significant testing events worth reviewing later

**Required Sections**:
- **Executive Summary**: Key findings, outcome status, high-level impact
- **Context**: What was being tested/investigated and why
- **Methodology**: Test setup, approach used, agents involved (if multi-agent)
- **Findings**: Detailed issues discovered with evidence, root causes, and severity
- **User Decisions**: Product/design decisions made during investigation
- **What Worked Well**: Successes and validated approaches
- **Lessons Learned**: Actionable insights for future work
- **Next Steps**: Immediate actions, short-term improvements, long-term considerations
- **References**: Links to code, docs, test artifacts

**Naming Convention**: `YYYY-MM-DD-descriptive-slug.md`
**Location**: `docs/case-studies/`

**When to Write a Case Study**:
- First real-world test of a new feature reveals critical issues
- Multi-agent testing session produces significant findings
- Production incident with non-obvious root cause
- Experiment yields unexpected results worth preserving
- Debugging session uncovers architectural gaps

### 7. Changelog / Release Notes
**Purpose**: Track version history, breaking changes, and migration paths

**Required Sections**:
- **Version Header**: Version number, date, and optional codename
- **Breaking Changes**: Changes that require user action (highlighted prominently)
- **New Features**: Added functionality with brief descriptions
- **Improvements**: Enhancements to existing features
- **Bug Fixes**: Resolved issues with references to issues/PRs
- **Deprecations**: Features marked for future removal
- **Migration Guide**: Steps to upgrade (when breaking changes exist)

**Naming Convention**: `CHANGELOG.md` (single file) or `releases/vX.Y.Z.md` (per-release)
**Location**: Repository root or `docs/releases/`

**Entry Format**:
```markdown
## [1.2.0] - 2025-12-20

### Breaking Changes
- `authenticate()` now requires explicit scopes parameter (#123)

### New Features
- Added bulk episode logging with range support (#145)

### Improvements
- Improved error messages for rate limiting

### Bug Fixes
- Fixed duplicate detection for movies (#142)

### Migration
1. Update all `authenticate()` calls to include scopes
2. Run `npm run migrate:auth` to update stored tokens
```

### 8. Troubleshooting Guide
**Purpose**: Help users diagnose and resolve common problems quickly

**Required Sections**:
- **Quick Diagnostics**: First steps to identify the problem category
- **Common Issues**: Organized by symptom, not cause
- **Error Reference**: Specific error codes/messages with solutions
- **Environment Issues**: Platform-specific problems
- **Getting Help**: Escalation paths when self-service fails

**Structure for Each Issue**:
```markdown
### Issue: [Symptom as user would describe it]

**Symptoms**:
- What the user sees/experiences

**Likely Causes**:
1. Most common cause
2. Second most common
3. Less common but possible

**Solutions**:

**Cause 1: [Name]**
1. Step to diagnose
2. Step to fix
3. How to verify fix worked

**Cause 2: [Name]**
...

**Still Not Working?**
- Link to relevant docs
- How to report the issue
```

**Naming Convention**: `TROUBLESHOOTING.md` or `docs/guides/TROUBLESHOOTING.md`
**Location**: Repository root or `docs/guides/`

**When to Add New Entries**:
- Same question asked 3+ times in issues/support
- Error message is confusing without context
- Platform-specific issue discovered
- Post-incident (add the fix to prevent recurrence)

---

## Metadata Schema (Progressive)

Start with minimal metadata, add fields as needs emerge.

### Phase 1: Minimal (Required)
```yaml
---
doc_type: [overview|api|adr|howto|operations|case-study|changelog|troubleshooting]
status: [draft|stable|deprecated]
last_updated: YYYY-MM-DD
---
```

### Phase 2: Enhanced (Add when useful)
```yaml
---
doc_type: case-study
status: stable
last_updated: 2025-12-19
audience: [users|developers|ai-assistants]
test_date: 2025-12-16
feature: Offline Watch Log Queue Synchronization
outcome: [success|partial|failed|blocked]
---
```

### Phase 3: Full (For RAG optimization)
```yaml
---
doc_type: api
status: stable
last_updated: 2025-12-19
audience: developers
prerequisites: [TypeScript, MCP basics]
source_context:
  files: [src/tools/log_watch.ts, src/schemas/watch.ts]
  commit: abc123
related_docs:
  - docs/guides/NATURAL_LANGUAGE_GUIDE.md
  - docs/operations/DEBUGGING.md
generated_by: tech-writer
---
```

---

## Multi-Agent Workflow

### Phase 1: ANALYSIS (Parallel)

Gather context using available tools and agents:

#### Code Analysis (use: code-reviewer or repo-topology)
**Questions to answer**:
- What changed in behavior, inputs, outputs, and error handling?
- Which public functions/classes are user-facing?
- What are the side effects and external dependencies?
- What validation or constraints exist?

#### Usage Pattern Extraction (analyze tests directly)
**Questions to answer**:
- What test scenarios exist for this feature?
- What edge cases are handled?
- What natural language patterns map to this functionality?
- What are common user mistakes?

#### Documentation Inventory (use: tech-writer with tools)
**Actions**:
- Scan existing docs for related content (Grep, Glob)
- Identify where new docs should live (follow existing patterns)
- Check for potential duplications or contradictions
- Build list of cross-references to include

### Phase 2: SYNTHESIS (Sequential)

Tech-writer combines Phase 1 outputs:
1. Draft documentation following document type template
2. Add metadata header (start with Phase 1 minimal)
3. Include examples from test scenarios
4. Add cross-references to related docs
5. Mark uncertain sections with `[Needs Verification]`

### Phase 3: VALIDATION (Parallel)

Run quality checks simultaneously:

#### Style & Consistency (use: code-reviewer)
**Check**:
- Consistent terminology across docs
- Proper markdown formatting
- Code blocks have language tags
- Internal links are valid

#### Example Verification (use: code-reviewer or manual testing)
**Check**:
- Code examples are syntactically correct
- Examples match current API signatures
- Edge cases are documented
- Error scenarios are covered

### Phase 4: FINALIZATION (Sequential)

Tech-writer incorporates validation feedback:
1. Address all flagged issues
2. Update `last_updated` date
3. Add to documentation index (if new doc)
4. Verify all cross-references work

---

## Self-Evaluation Questions

Run these checks on every documentation task:

### Usefulness & Coverage
- [ ] On a 1-5 scale, how useful is this doc for: (a) new contributors, (b) integrators, (c) LLM-based agents?
- [ ] What key tasks can a user perform after reading this doc?
- [ ] What tasks remain unclear?

### Accuracy & Drift
- [ ] Are there statements in the doc that no longer match current code?
- [ ] Does the doc make assumptions not backed by current implementation?
- [ ] Are dependency versions and paths current?

### Structure & Intent
- [ ] Is the intent explicitly stated in the first 2-3 sentences?
- [ ] Are preconditions, postconditions, and side effects clearly labeled?
- [ ] Are headings and anchors stable (for deep linking)?

### Situational Awareness
- [ ] Which other modules or features depend on this component?
- [ ] Are relationships with other docs described or linked?
- [ ] What context would an incident responder need that's currently missing?

### For Case Studies Specifically
- [ ] Are key findings actionable (not just observations)?
- [ ] Are lessons learned specific enough to apply to future work?
- [ ] Is there a clear "what worked" section to preserve successful patterns?
- [ ] Are next steps prioritized (immediate/short-term/long-term)?

---

## Quality Gates

Documentation must pass these gates before completion:

### Gate 1: Completeness
- [ ] All required sections for document type are present
- [ ] Metadata header is included
- [ ] At least one example exists (where applicable)

### Gate 2: Accuracy
- [ ] File paths referenced exist in codebase
- [ ] Code examples compile/run without errors
- [ ] API signatures match implementation

### Gate 3: Consistency
- [ ] Follows project terminology conventions
- [ ] Uses consistent formatting (headings, lists, code blocks)
- [ ] Links to related docs work

### Gate 4: Indexing
- [ ] New docs are added to documentation index
- [ ] Cross-references are bidirectional where appropriate
- [ ] Metadata is machine-parseable

---

## Tools Used

- **Discovery**: Glob (find docs), Grep (search content), Read (examine files)
- **Analysis**: code-reviewer or repo-topology (code structure), test file analysis (patterns)
- **Validation**: code-reviewer (quality and example verification)
- **Output**: Write, Edit (create/modify documentation files)

---

## Example Workflow: Document New Feature

**Input**: "Document the new bulk_log feature"

**Phase 1 (Parallel)**:
```
[repo-topology]: Extract bulk_log tool schema, parameters, validation rules
[code-reviewer]: Find test scenarios in src/__tests__/bulk-log.test.ts
[tech-writer]: Scan docs/ for existing bulk logging mentions
```

**Phase 2**:
```
[tech-writer]: Draft API reference with:
- Tool signature from repo-topology analysis
- Examples from test scenario analysis
- Cross-references from doc scan
- Minimal metadata header
```

**Phase 3 (Parallel)**:
```
[code-reviewer]: Validate style, formatting, link integrity
[code-reviewer]: Verify examples match actual API behavior
```

**Phase 4**:
```
[tech-writer]:
- Address review feedback
- Add to docs/README.md index
- Update last_updated date
- Commit with conventional commit message
```

---

## Example Workflow: Create Case Study

**Input**: "Document the sync queue testing session from today"

**Phase 1 (Parallel)**:
```
[code-reviewer]: Extract test findings, failure patterns, edge cases discovered
[repo-topology]: Identify code issues, missing instrumentation, API errors
[tech-writer]: Gather decisions made during testing, previous related docs
```

**Phase 2**:
```
[tech-writer]: Draft case study with:
- Executive summary from consolidated findings
- Methodology from test session structure
- Detailed findings organized by category
- Lessons learned with specific applications
- Next steps with prioritization
```

**Phase 3**:
```
[code-reviewer]: Validate technical accuracy of findings
[code-reviewer]: Confirm edge cases and failure patterns are correctly described
```

**Phase 4**:
```
[tech-writer]:
- Name file with date: docs/case-studies/YYYY-MM-DD-descriptive-slug.md
- Add to docs/README.md under Case Studies section
- Cross-reference from related feature docs
```

---

## Integration Notes

### For Manual Invocation
User or assistant invokes skill with specific documentation task.

### For CI/CD Integration (Future)
- Hook into PR workflow to validate doc changes
- Auto-detect when code changes need doc updates
- Generate changelog entries from commit messages

### For RAG Optimization
- Emit structured metadata for vector indexing
- Use stable heading anchors for chunk references
- Include `source_context` for traceability

---

## Common Patterns

### Creating New Documentation
1. Identify document type from task
2. Use template for that type
3. Run full 4-phase workflow
4. Add to index

### Updating Existing Documentation
1. Read current doc and related code
2. Identify what changed (diff analysis)
3. Propose specific edits (not full rewrites)
4. Run Phase 3-4 only

### Documentation Audit
1. Scan all docs for staleness (last_updated > 90 days)
2. Cross-reference with recent code changes
3. Generate report of docs needing updates
4. Prioritize by traffic/importance

### Post-Incident Case Study
1. Wait until incident is resolved (not during)
2. Gather timeline, findings, and decisions from session
3. Write case study within 24-48 hours while fresh
4. Include "What Worked Well" to reinforce good patterns
5. Mark status as stable (case studies don't get deprecated)

---

## Repository-Specific Rules

This skill can be extended with repo-specific rules by creating:
`{repo}/.claude/repo-docs-rules.md`

Contents can include:
- Custom document types
- Additional required sections
- Terminology conventions
- Style guide overrides
- Index location and structure

---

## Appendix: Case Study Template

```markdown
# Case Study: [Descriptive Title]

**Date:** YYYY-MM-DD
**Feature:** [Feature or system being tested]
**Test Type:** [Manual|Automated|Multi-Agent|Integration|etc.]
**Status:** [Success|Partial Success|Failed|Blocked]

---

## Executive Summary

[2-3 paragraph summary of what was tested, key findings, and outcome]

### Key Findings
- Finding 1
- Finding 2
- Finding 3

### Outcome
[One sentence describing the result and immediate impact]

---

## Context

### What is [Feature]?
[Explain the feature, its purpose, and architecture]

### Previous Testing
[What testing existed before this case study]

---

## Methodology

### Test Setup
[Environment, branch, data sources]

### Approach
[Detailed methodology, agents used, phases]

---

## Findings

### 1. [Issue Category]
**Issue:** [Description]
**Evidence:** [Data, logs, screenshots]
**Root Cause:** [Analysis]
**Impact:** [Severity and scope]

[Repeat for each finding]

---

## User Decisions

### Decision 1: [Topic]
**Decision:** [What was decided]
**Rationale:** [Why]

[Repeat for each decision]

---

## What Worked Well

### 1. [Success Area]
**Observation:** [What worked]
**Value:** [Why it matters]

[Repeat for each success]

---

## Lessons Learned

### 1. [Lesson Topic]
**Lesson:** [The insight]
**Application:** [How to apply it going forward]

[Repeat for each lesson]

---

## Next Steps

### Immediate Actions (Before Next Test)
1. Action 1
2. Action 2

### Short-Term Improvements
1. Improvement 1
2. Improvement 2

### Long-Term Considerations
1. Consideration 1
2. Consideration 2

---

## References

### Documentation
- Link 1
- Link 2

### Test Artifacts
- Artifact 1
- Artifact 2

---

**End of Case Study**
```
