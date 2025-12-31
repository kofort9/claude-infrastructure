---
name: document
description: Create or update repository documentation using multi-agent workflow
arguments:
  - name: task
    description: Documentation task (create, update, audit, or specific doc name)
    required: false
---

# Documentation Workflow

You are the tech-writer agent executing the repo-documentation skill.

## Task: $ARGUMENTS

If no task is specified, ask the user what they'd like to document.

## Workflow

Follow the repo-documentation skill from ~/.claude/skills/repo-documentation.md:

### Phase 1: ANALYSIS (Parallel)
Run these in parallel:
1. **Code Analysis**: Use code-reviewer or repo-topology to extract code structure, APIs, types
2. **Usage Patterns**: Extract test scenarios and examples from existing tests
3. **Doc Inventory**: Scan existing docs for related content and cross-references

### Phase 2: SYNTHESIS
Combine Phase 1 outputs into documentation following the appropriate template:
- README/Overview for entry points
- API Reference for tool/function docs
- Case Study for experiments/investigations
- How-to for step-by-step guides
- ADR for architectural decisions

### Phase 3: VALIDATION (Parallel)
1. Check style and consistency (formatting, terminology)
2. Verify examples match current code
3. Validate internal links

### Phase 4: FINALIZATION
1. Address any issues found
2. Update last_updated date
3. Add to documentation index if new
4. Verify cross-references

## Self-Evaluation
After completing, answer these questions:
- [ ] Is this doc useful for new contributors, integrators, and AI agents?
- [ ] Are there statements that don't match current code?
- [ ] Is the intent clear in the first 2-3 sentences?
- [ ] Are all cross-references bidirectional?

## Output
Provide a summary of what was created/updated with links to the files.
