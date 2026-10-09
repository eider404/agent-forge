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
plan's headings, task keywords (`IMPLEMENT`, `VALIDATE`, …), IDs (`T-n`, `AC-n`) and the markers
`TBD — needs validation` and `Assumed — …` stay in English, verbatim, because `piv-implement`, `piv-validate` and
`piv-review-changes` find them by name. The ticket fields you read in `tickets.md` are in English for the same reason.

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
| Each `Depends on` ticket's plan and report | `plans/t-<m>-*.md` and `reports/t-<m>-*-report.md` in the same feature folder (highest `-n` suffix if there are several): what was actually built (`Built (for dependent tickets)`) and any deviations | Back-references, Patterns to Follow |

**Inherit, don't re-decide**: this is a **per-ticket** plan. Treat the cross-cutting calls in those documents
(stack & versions, data model, security boundaries, the seams new code plugs into) as **already decided**: inherit
them, don't reopen them. Plan only what's left at the ticket level: the specific files, the local patterns to mirror,
the tests. If a ticket genuinely needs to break one of those decisions, flag it in Open Questions rather than silently
diverging. (A tracker-only setup with no local `architecture.md`: the linked architecture page, or an
`## Architecture` / `## Engineering` section on the epic, plays the same role.)

**Dependencies not built**: a `Depends on` ticket with no `COMPLETE` report hasn't delivered the contracts this plan
would build on. Don't block: name it in the Phase 0 summary and ask at the Phase 2 GATE (wait for it, or plan against
its plan and mark the guesses `Assumed — …`).

**Blocked by**: a spike or open question the ticket is blocked by, with no recorded outcome (an `Outcome:` line under
that spike in `architecture.md`, or the spike ticket's report), is not settled, so
don't plan around a guess. An unresolved spike goes first (say so; or plan the spike, if that is this ticket); an
open question goes to the Phase 2 GATE. Product questions belong to the PRD owner: carry them over, don't decide them.

**Spike tickets** (`Type: spike`): plan the experiment, not the feature. Tasks build the smallest thing that answers
the question within the timebox, the acceptance criteria are the ticket's decision rule, and the last task records
which side of the rule the result fell on, in `architecture.md` (Phase 5).

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

### Phase 5: Branch, then write the plan

**Cut the ticket's branch first.** The plan, the code and the implementation report go in one PR, so the branch
comes before the plan file. This is the one place the branch rule lives; `piv-implement` and `piv-run-full-loop`
defer to it. Detect the base branch: `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed
's@^refs/remotes/origin/@@'` (fallback `main`).

- **On the base branch, clean** → `git checkout -b feature/<id>-<kebab-case-name>`, with `<id>` and the name the plan
  file will use (Output Format): `feature/t-2-source-adapter`, `feature/acc-30-source-adapter`, `feature/csv-export`.
- **Already on this ticket's branch, or in a worktree** → use it.
- **This ticket's branch already exists but you're elsewhere** (`git branch --list 'feature/<id>-*'`; a ticket
  started before and not merged) → stop and ask: switch to it and resume (its plan is there; next step
  `piv-implement`), or plan again from scratch.
- **On another ticket's branch** (`feature/t-<m>-…` or another key) → stop and ask: switch to the base branch (the
  usual answer once that ticket is merged), or stack this ticket on top of it on purpose.
- **On the base branch with uncommitted changes outside `.agents/features/`** → stop: commit or stash them first.
- **Not a git repository** → say so in one line and write the plan anyway; `piv-implement` needs git later.

Then fill [templates/plan.md](templates/plan.md): it is the plan's shape, the contract `piv-implement` executes. Its
headings and task keywords stay in English (see Language).

**`VALIDATION COMMANDS` levels 1–3 come from the project, not from memory**: take them from
`.claude/references/conventions.md › ## validate` if it exists; otherwise resolve them the way `piv-validate` does
(CI first, then manifests and task runners; `.claude/skills/piv-validate/references/discover-commands.md`). Same
source, same commands: a plan that passes them is green by the project's own definition, which is what lets
`piv-run-full-loop` skip `piv-validate`.

**Spike tickets**: the last task also writes the outcome into the feature's `architecture.md`: an `Outcome:` line
under that spike (which side of the decision rule, and the evidence) and, when it settles a decision, one line under
`Key decisions`. Dependent tickets and `rules-create-global` read the outcome from there.

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
  (`…-2.md`); the highest suffix is the current plan, and its report takes the same stem.

## Quality Criteria

### Context Completeness ✓

- [ ] All necessary patterns identified and documented
- [ ] External library usage documented with links
- [ ] Integration points clearly mapped
- [ ] Gotchas and anti-patterns captured
- [ ] Every task has executable validation command
- [ ] Phase 0 read the ticket, the architecture sections that bind it, the PRD scope and any dependency's plan and report; the inherited decisions are listed under Related Work
- [ ] Phase 2's clarifying cluster was asked and answered, or explicitly recorded as nothing open
- [ ] `VALIDATION COMMANDS` levels 1–3 come from `## validate` or the project's CI/manifests, not from memory

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
- Full path to created Plan file and the branch it sits on (the next step is `piv-implement` with that path)
- Complexity assessment
- Key implementation risks or considerations
- Estimated confidence score for one-pass success
