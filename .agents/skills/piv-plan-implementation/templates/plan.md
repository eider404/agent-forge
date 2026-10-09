# Feature: <feature-name>

The following plan should be complete, but its important that you validate documentation and codebase patterns and task sanity before you start implementing.

Pay special attention to naming of existing utils types and models. Import from the right files etc.

## Feature Description

<Detailed description of the feature, its purpose, and value to users>

## User Story

As a <type of user>
I want to <action/goal>
So that <benefit/value>

## Problem Statement

<Clearly define the specific problem or opportunity this feature addresses>

## Solution Statement

<Describe the proposed solution approach and how it solves the problem>

## Out of Scope / Non-Goals

<Explicitly bound the work: what this feature does NOT include. Name the things a reasonable reader might assume are in scope but aren't — this is what stops the agent from gold-plating or solving the wrong problem.>

- Not included: <thing> (defer to <later / separate ticket>)
- Not changing: <existing behavior to leave alone>

## Feature Metadata

**Feature Type**: [New Capability/Enhancement/Refactor/Bug Fix]
**Estimated Complexity**: [Low/Medium/High]
**Primary Systems Affected**: [List of main components/services]
**Dependencies**: [External libraries or services required]

## Related Work

<Links between this plan and the work around it. Distinct from CONTEXT REFERENCES below (which lists files/docs to read for *this* implementation) — this is the plan's place in the larger graph.>

**Implements**: <`T-n` (+ tracker key or link), or "n/a — free-form">   ·   **Feature folder**: `.agents/features/<YYYY-MM-DD>-<slug>/` (`prd.md`, `architecture.md`, `tickets.md`)

**Inherited decisions** (already decided upstream — Phase 0; don't reopen):

- <decision, one line> - Source: `architecture.md › <section>`

**Back-references** (plans this builds on or inherits decisions from — the `Depends on` tickets first; paths relative to the feature folder):

- `plans/<prior-plan>.md` (report: `reports/<prior-plan>-report.md`) - Why: shares the auth seam / reuses the X service

**Forward-references** (plans that extend or supersede this — append as follow-ups get created):

- (none yet)

---

## CONTEXT REFERENCES

### Relevant Codebase Files IMPORTANT: YOU MUST READ THESE FILES BEFORE IMPLEMENTING!

<List files with line numbers and relevance>

- `path/to/file.py` (lines 15-45) - Why: Contains pattern for X that we'll mirror
- `path/to/model.py` (lines 100-120) - Why: Database model structure to follow
- `path/to/test.py` - Why: Test pattern example

### New Files to Create

- `path/to/new_service.py` - Service implementation for X functionality
- `path/to/new_model.py` - Data model for Y resource
- `tests/path/to/test_new_service.py` - Unit tests for new service

### Relevant Documentation YOU SHOULD READ THESE BEFORE IMPLEMENTING!

- [Documentation Link 1](https://example.com/doc1#section)
  - Specific section: Authentication setup
  - Why: Required for implementing secure endpoints
- [Documentation Link 2](https://example.com/doc2#integration)
  - Specific section: Database integration
  - Why: Shows proper async database patterns

### Patterns to Follow

<Specific patterns extracted from codebase - include actual code examples from the project>

**Naming Conventions:** (for example)

**Error Handling:** (for example)

**Logging Pattern:** (for example)

**Other Relevant Patterns:** (for example)

---

## IMPLEMENTATION PLAN

Phases run **top to bottom by default** — each assumes the phase above it is done. Where that is NOT the true dependency, make it explicit with a `**Depends on:**` line under the phase header, and a `**Independent of:**` line where two phases don't block each other. Independent phases are candidates to run in **parallel** (e.g. separate worktrees / parallel loops). Only annotate where it changes execution order or unlocks parallelism — skip the obvious sequential case.

### Phase 1: Foundation

<Describe foundational work needed before main implementation>

**Tasks:**

- Set up base structures (schemas, types, interfaces)
- Configure necessary dependencies
- Create foundational utilities or helpers

### Phase 2: Core Implementation

**Depends on:** Phase 1 (needs the base schemas/types)

<Describe the main implementation work>

**Tasks:**

- Implement core business logic
- Create service layer components
- Add API endpoints or interfaces
- Implement data models

### Phase 3: Integration

<Describe how feature integrates with existing functionality>

**Tasks:**

- Connect to existing routers/handlers
- Register new components
- Update configuration files
- Add middleware or interceptors if needed

### Phase 4: Testing & Validation

<Describe testing approach>

**Tasks:**

- Implement unit tests for each component
- Create integration tests for feature workflow
- Add edge case tests
- Validate against acceptance criteria

---

## STEP-BY-STEP TASKS

IMPORTANT: Execute every task in order, top to bottom. Each task is atomic and independently testable.

### Task Format Guidelines

Use information-dense keywords for clarity:

- **CREATE**: New files or components
- **UPDATE**: Modify existing files
- **ADD**: Insert new functionality into existing code
- **REMOVE**: Delete deprecated code
- **REFACTOR**: Restructure without changing behavior
- **MIRROR**: Copy pattern from elsewhere in codebase

### {ACTION} {target_file}

- **IMPLEMENT**: {Specific implementation detail}
- **PATTERN**: {Reference to existing pattern - file:line}
- **IMPORTS**: {Required imports and dependencies}
- **GOTCHA**: {Known issues or constraints to avoid}
- **VALIDATE**: `{executable validation command}`
- **SATISFIES**: {which acceptance criterion this task advances — the ticket's own ID, e.g. AC-2 — so every task traces to a criterion}

<Continue with all tasks in dependency order...>

---

## TESTING STRATEGY

<Define testing approach based on project's test framework and patterns discovered during research>

### Unit Tests

<Scope and requirements based on project standards>

Design unit tests with fixtures and assertions following existing testing approaches

### Integration Tests

<Scope and requirements based on project standards>

### Edge Cases

<List specific edge cases that must be tested for this feature>

---

## VALIDATION COMMANDS

<Levels 1–3: the project's own commands, from `.claude/references/conventions.md › ## validate` or, without it, from CI and the manifests — each with the directory it runs from (`cd backend && uv run pytest`). Levels 4–5: feature-specific.>

Execute every command to ensure zero regressions and 100% feature correctness.

### Level 1: Syntax & Style

<Project-specific linting and formatting commands>

### Level 2: Unit Tests

<Project-specific unit test commands>

### Level 3: Integration Tests

<Project-specific integration test commands>

### Level 4: Manual Validation

<Feature-specific manual testing steps - API calls, UI testing, etc.>

### Level 5: Additional Validation (Optional)

<MCP servers or additional CLI tools if available>

---

## ACCEPTANCE CRITERIA

<List specific, measurable criteria that must be met for completion. Start with the ticket's own criteria, verbatim and with their IDs (`AC-1`, `AC-2`, …); with no ticket, the criteria confirmed at the Phase 2 GATE. The generic gates follow.>

- [ ] AC-1 — <the ticket's criterion>
- [ ] Feature implements all specified functionality
- [ ] All validation commands pass with zero errors
- [ ] Unit test coverage meets requirements (80%+)
- [ ] Integration tests verify end-to-end workflows
- [ ] Code follows project conventions and patterns
- [ ] No regressions in existing functionality
- [ ] Documentation is updated (if applicable)
- [ ] Performance meets requirements (if applicable)
- [ ] Security considerations addressed (if applicable)

---

## COMPLETION CHECKLIST

- [ ] All tasks completed in order
- [ ] Each task validation passed immediately
- [ ] All validation commands executed successfully
- [ ] Full test suite passes (unit + integration)
- [ ] No linting or type checking errors
- [ ] Manual testing confirms feature works
- [ ] Acceptance criteria all met
- [ ] Code reviewed for quality and maintainability

---

## OPEN QUESTIONS / ASSUMPTIONS

<Surface anything still uncertain instead of silently guessing. List the assumptions this plan makes, and any question that — if answered differently — would change the plan. Flag unresolved critical questions for the user before execution.>

## NOTES (open canvas)

<No fixed shape. Reason freely here: alternatives you weighed and rejected and why, a tradeoff matrix, a sequencing or rollout risk, a data-flow sketch, open threads, links — whatever serves the plan. The sections above template the plan's *shape* so the implementation agent can consume it; this section keeps your *reasoning* unconstrained. Prose, lists, tables, code blocks all welcome.>

## AMENDMENTS

<Append-only history of changes made to this plan AFTER it was first approved/executed. Leave empty at creation; newest entry at the bottom. Each entry: date — what changed and why.>

- <ISO date> — <what changed and why, e.g. "scope cut: deferred bulk-import to a follow-up ticket after AC review">
