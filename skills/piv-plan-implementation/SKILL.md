---
name: piv-plan-implementation
description: Creates a comprehensive, context-rich implementation plan for one ticket by reading the upstream artifacts (ticket, architecture, PRD), analyzing the codebase, running a short clarifying interview and researching external docs, then saves it in the feature folder's `plans/`. Accepts a ticket from a feature's `tickets.md` (`T-n`), a tracker key or URL (Jira/Linear/GitHub), a feature folder (`.agents/features/<date>-<slug>/`) for a small scope that skips slicing, or a free-form feature request. Use when asked to "plan this ticket", "plan T-2", "plan ACC-30" or "plan the implementation", or when you have a ticket or small feature and need a one-pass-ready plan before writing any code.
argument-hint: "[T-n | ticket key/URL | feature folder .agents/features/<date>-<slug>/ | free-form feature description] (a T-n may follow the folder: <folder> T-2)"
---

# Plan a new task

## Feature: $ARGUMENTS

## Input

`$ARGUMENTS` takes one of four forms; the first three come with upstream artifacts that Phase 0 reads:

- **A ticket from `tickets.md`**: `T-n`, alone or after its feature folder (`<folder> T-2`). Alone, use the most
  recent feature folder that has a `tickets.md`; Phase 0's summary names it, so a wrong pick is caught at once.
- **A tracker ticket**: a key such as `ABC-123`, or a Jira / Linear / GitHub issue URL. If it appears in a local
  `tickets.md` (grep the key in `.agents/features/*/tickets.md`), treat it as the form above. Otherwise **fetch it
  from the tracker before you plan** (Jira via the Atlassian MCP, GitHub via `gh issue view`, etc.) and follow its
  links up to the epic and the epic's linked architecture page (Confluence via the Atlassian MCP).
- **A feature folder** `.agents/features/<YYYY-MM-DD>-<slug>/` with `prd.md` and `architecture.md` but no ticket:
  the small-scope path `plan-architecture` offers when slicing isn't worth it. The PRD's MVP is the scope.
- **A free-form description**: plan directly from it (greenfield or ad-hoc), asking clarifying questions as needed.
  If it clearly matches an existing feature folder, offer that folder; otherwise the plan goes in a new one (see
  "Output Format").

Never plan from a bare key or title: the ticket body plus its architecture and PRD are the real input.

## Language

Work in the language of the ticket (or the PRD): the conversation, the plan's content and the closing report. The
plan's headings and task keywords (`IMPLEMENT`, `VALIDATE`, …) stay in English, because `piv-implement` and the
factory prompts read them. Keep the "TBD — needs validation" marker, translated, and use it consistently.

## Mission

Transform a feature request into a **comprehensive implementation plan** through systematic codebase analysis, external research, and strategic planning.

**Core Principle**: We do NOT write code in this phase. Our goal is to create a context-rich implementation plan that enables one-pass implementation success for ai agents.

**Key Philosophy**: Context is King. The plan must contain ALL information needed for implementation - patterns, mandatory reading, documentation, validation commands - so the execution agent succeeds on the first attempt.

## Planning Process

### Phase 0: Gather Upstream Context

Before touching the code, read what the earlier skills already decided; that is the context this plan inherits.
Prefer the local feature folder over the tracker (the same rule `plan-architecture` and `plan-slice-tickets`
follow); go to the tracker when there is no local copy, or to check that the local ticket hasn't drifted from it.
Read only what the ticket points to, in this order:

| Read | What | Feeds |
|---|---|---|
| The ticket (`T-n` in `tickets.md`, or the tracker issue) | Type, Scope, `AC-n`, Architecture decisions, Per-ticket context, Blocked by, Depends on | User Story, Acceptance Criteria, each task's `SATISFIES`, Out of Scope |
| `architecture.md` | The sections named in `Architecture decisions`, plus any Missing pieces, Spikes and Open questions that touch this ticket (the whole file if there is no ticket) | Inherited decisions; the seams Phase 2 starts from |
| `prd.md` | The sections named in `Per-ticket context`; always MVP and Non-goals | Problem Statement, Out of Scope |
| Each `Depends on` ticket's plan and report | `plans/` and `reports/` in the same feature folder, if they exist: what was actually built and any deviations | Back-references, Patterns to Follow |

**Inherit, don't re-decide**: this is a **per-ticket** plan. Treat the cross-cutting calls in those documents
(stack & versions, data model, security boundaries, the seams new code plugs into) as **already decided**: inherit
them, don't reopen them. Plan only what's left at the ticket level: the specific files, the local patterns to mirror,
the tests. If a ticket genuinely needs to break one of those decisions, flag it in Open Questions rather than silently
diverging. (A tracker-only setup with no local `architecture.md`: the linked architecture page, or an
`## Architecture` / `## Engineering` section on the epic, plays the same role.)

**Blocked by**: a spike or open question the ticket is blocked by, with no recorded outcome, is not settled, so
don't plan around a guess. An unresolved spike goes first (say so; or plan the spike, if that is this ticket); an
open question goes to the Phase 2 GATE. Product questions belong to the PRD owner: carry them over, don't decide them.

**Spike tickets** (`Type: spike`): plan the experiment, not the feature. Tasks build the smallest thing that answers
the question within the timebox, the acceptance criteria are the ticket's decision rule, and the last task records
which side of the rule the result fell on.

**No ticket** (feature folder or free-form): there are no `AC-n`. Propose criteria derived from the MVP and mark
them "TBD — needs validation" until the Phase 2 GATE confirms them. For a feature folder, if the MVP clearly exceeds
one PIV loop (~8–10 subtasks, ~500–1500 lines of change), say so and suggest `plan-slice-tickets`; don't block.

Close Phase 0 with a 3–5 line summary: the ticket and its feature folder, the decisions inherited, the dependencies
read, any blocker.

### Phase 1: Feature Understanding

**Deep Feature Analysis:**

- Extract the core problem being solved
- Identify user value and business impact
- Determine feature type: New Capability/Enhancement/Refactor/Bug Fix
- Assess complexity: Low/Medium/High
- Map affected systems and components

**Create User Story Format Or Refine If Story Was Provided By The User:**

```
As a <type of user>
I want to <action/goal>
So that <benefit/value>
```

### Phase 2: Codebase Intelligence Gathering

**Use specialized agents and parallel analysis.** Start from what Phase 0 surfaced (the seams, data model and missing
pieces the architecture names) and widen only as far as the ticket needs. If the session is already primed
(`prime-codebase` output in the conversation, or a priming file the caller points to), reuse it and skip item 1.

**1. Project Structure Analysis**

- Detect primary language(s), frameworks, and runtime versions
- Map directory structure and architectural patterns
- Identify service/component boundaries and integration points
- Locate configuration files (pyproject.toml, package.json, etc.)
- Find environment setup and build processes

**2. Pattern Recognition** (Use specialized subagents when beneficial)

- Search for similar implementations in codebase
- Identify coding conventions:
  - Naming patterns (CamelCase, snake_case, kebab-case)
  - File organization and module structure
  - Error handling approaches
  - Logging patterns and standards
- Extract common patterns for the feature's domain
- Document anti-patterns to avoid
- Check CLAUDE.md for project-specific rules and conventions

**3. Dependency Analysis**

- Catalog external libraries relevant to feature
- Understand how libraries are integrated (check imports, configs)
- Find relevant documentation in docs/, ai_docs/, .claude/references or ai-wiki if available
- Note library versions and compatibility requirements

**4. Testing Patterns**

- Identify test framework and structure (pytest, jest, etc.)
- Find similar test examples for reference
- Understand test organization (unit vs integration)
- Note coverage requirements and testing standards

**5. Integration Points**

- Identify existing files that need updates
- Determine new files that need creation and their locations
- Map router/API registration patterns
- Understand database/model patterns if applicable
- Identify authentication/authorization patterns if relevant

**Clarify Ambiguities — GATE:**

Codebase analysis is done, so the open questions are now *specific*. This is the one moment where you know
enough to ask well and have not yet written anything. **GATE** means: post the questions, then stop. End the
turn and wait for the answers. Do not ask and answer in the same breath, and do not roll into Phase 3.

Ask in **one cluster**, numbered, 3-6 questions max, each carrying a **recommended default** so answering is
cheap ("I'll mirror the first unless you say otherwise"). Draw them only from what the analysis actually left
open:

1. **Scope boundary** — the adjacent thing a reasonable reader would assume is in scope. Confirm it is out.
2. **Pattern fork** — two existing patterns both fit. Name both with `file:line` and ask which to mirror.
3. **Contract shape** — the API surface, payload, or data-model change the ticket implies but never states.
4. **Failure behavior** — what happens on the error path the ticket is silent about.
5. **Preference** — a library or trade-off with no precedent in this codebase to inherit.
6. **Done** — an acceptance criterion that is missing, or written so that it cannot be checked. The ticket's `AC-n`
   are the baseline; with no ticket, confirm the criteria Phase 0 proposed from the MVP.

Skip any category with nothing genuinely open; never manufacture questions to fill the list. If the ticket, the
architecture and the PRD genuinely settle everything, say so in one line and proceed. Silence is not the
same as clearance.

**Thin answers:** reflect a vague answer back as the concrete choice it leaves open ("'handle errors gracefully'
— a 4xx with a message, or retry then 503?") and ask once more. Never upgrade a vague answer into a confident plan.

**If they decline** ("just write it"): honour it, but name what you are guessing. Every unanswered item becomes
an `Assumed — <the assumption>, confirm before execution` line in `OPEN QUESTIONS / ASSUMPTIONS`, and the task it
affects carries a `**GOTCHA**` naming it. Never guess silently.

**Already settled upstream:** anything the ticket, `architecture.md` or the PRD already answers is not open.
Inherit it and skip (Phase 0). The exception is a blocker Phase 0 flagged: it is still open, so ask it here.

### Phase 3: External Research & Documentation

**Use specialized subagents when beneficial for external research:**

**Documentation Gathering:**

- Research latest library versions and best practices
- Find official documentation with specific section anchors
- Locate implementation examples and tutorials
- Identify common gotchas and known issues
- Check for breaking changes and migration guides

**Technology Trends:**

- Research current best practices for the technology stack
- Find relevant blog posts, guides, or case studies
- Identify performance optimization patterns
- Document security considerations

**Compile Research References:**

```markdown
## Relevant Documentation

- [Library Official Docs](https://example.com/docs#section)
  - Specific feature implementation guide
  - Why: Needed for X functionality
- [Framework Guide](https://example.com/guide#integration)
  - Integration patterns section
  - Why: Shows how to connect components
```

### Phase 4: Deep Strategic Thinking

**Think Harder About:**

- How does this feature fit into the existing architecture?
- What are the critical dependencies and order of operations?
- What could go wrong? (Edge cases, race conditions, errors)
- How will this be tested comprehensively?
- What performance implications exist?
- Are there security considerations?
- How maintainable is this approach?

**Design Decisions:**

- Choose between alternative approaches with clear rationale
- Design for extensibility and future modifications
- Plan for backward compatibility if needed
- Consider scalability implications

### Phase 5: Plan Structure Generation

**Create comprehensive plan with the following structure:**

Whats below here is a template for you to fill for the implementation agent:

```markdown
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

<Define validation commands based on project's tools discovered in Phase 2>

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
```

## Output Format

**Location**: the feature folder's `plans/`:
`.agents/features/<YYYY-MM-DD>-<slug>/plans/<id>-<kebab-case-name>.md`. Keeping a feature's artifacts together stops
plans from piling up in one flat directory. Create `plans/` if it doesn't exist.

| Input | `<id>-` prefix | Example |
|---|---|---|
| A ticket from `tickets.md` | `t-<n>-` | `plans/t-2-source-adapter.md` |
| A tracker ticket not in any `tickets.md` | `<key>-` (lowercase) | `plans/acc-30-source-adapter.md` |
| A feature folder, no ticket | none | `plans/csv-export.md` |
| A free-form description | none | `plans/add-user-authentication.md` |

- `<kebab-case-name>`: a short, descriptive feature name.
- **Folder**: reuse the one from Phase 0. A tracker-only ticket or a free-form description has none: create
  `.agents/features/<today>-<slug>/` (`date +%F`) holding just `plans/`.
- **Never overwrite**: the plan may already be executed or amended. If the file exists, add a numeric suffix
  (`…-2.md`).

## Quality Criteria

### Context Completeness ✓

- [ ] All necessary patterns identified and documented
- [ ] External library usage documented with links
- [ ] Integration points clearly mapped
- [ ] Gotchas and anti-patterns captured
- [ ] Every task has executable validation command
- [ ] Phase 0 read the ticket, the architecture sections that bind it, the PRD scope and any dependency's plan and report; the inherited decisions are listed under Related Work
- [ ] Phase 2's clarifying cluster was asked and answered, or explicitly recorded as nothing open

### Implementation Ready ✓

- [ ] Another developer could execute without additional context
- [ ] Tasks ordered by dependency (can execute top-to-bottom)
- [ ] Each task is atomic and independently testable
- [ ] Pattern references include specific file:line numbers
- [ ] Every acceptance criterion (the ticket's `AC-n`) is satisfied by at least one task

### Pattern Consistency ✓

- [ ] Tasks follow existing codebase conventions
- [ ] New patterns justified with clear rationale
- [ ] No reinvention of existing patterns or utils
- [ ] Testing approach matches project standards

### Information Density ✓

- [ ] No generic references (all specific and actionable)
- [ ] URLs include section anchors when applicable
- [ ] Task descriptions use codebase keywords
- [ ] Validation commands are non interactive executable

## Success Metrics

**One-Pass Implementation**: Execution agent can complete feature without additional research or clarification — clarification the *user* owes the plan belongs in Phase 2's gate, not deferred to the execution agent

**Validation Complete**: Every task has at least one working validation command

**Context Rich**: The Plan passes "No Prior Knowledge Test" - someone unfamiliar with codebase can implement using only Plan content

**Confidence Score**: #/10 that execution will succeed on first attempt

## Report

After creating the Plan, provide:

- Summary of feature and approach
- Full path to created Plan file (the next step is `piv-implement` with that path)
- Complexity assessment
- Key implementation risks or considerations
- Estimated confidence score for one-pass success
