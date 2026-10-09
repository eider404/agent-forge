---
name: piv-review-changes
description: Performs a technical code review of the changed files (uncommitted, or the branch's commits when the tree is clean) for bugs, security issues, performance problems and standards compliance, then writes a findings report. Optionally takes the plan from `piv-plan-implementation` and also checks the change against the plan's acceptance criteria (`AC-n`), scope and inherited decisions, treating the deviations documented in the implementation report as intentional. Use when asked to "review my changes", "code review this", "review before committing" or "review the PR", after `piv-validate` passes, or as the pre-commit quality gate. Reports only; it never fixes.
argument-hint: "[path-to-plan, e.g. .agents/features/<date>-<slug>/plans/t-2-<name>.md] (optional)"
---

# Code Review

Perform a technical code review of the changed files.

## Input

`$ARGUMENTS` is optional: the path to a plan created by `piv-plan-implementation`.

- **Empty**: review the change against the codebase's standards (steps 1, 2 and 3).
- **A plan**: also review it against what was planned and what was reported as built (step 1b and the plan-fit check
  in step 3).

## Language

Write the report in the plan's language (or the user's). Field names (`severity`, `file`, …), severity values and
the pass sentence stay in English, because `piv-fix-review-findings` reads them.

## Core Principles

- Simplicity is the ultimate sophistication - every line should justify its existence
- Code is read far more often than it's written - optimize for readability
- The best code is often the code you don't write
- Elegance emerges from clarity of intent and economy of expression

## 1. Gather context

### 1a. The codebase's standards

Start by understanding the codebase's standards and patterns:

- `CLAUDE.md` and `README.md`
- `.claude/references/conventions.md` if it exists: its coding standards count as the bar. Its `## commit` and
  `## validate` sections belong to other skills.
- Other documented standards (docs directory, the rest of `.claude/references/`) and the key files of the core module

### 1b. What was planned (only with a plan)

The plan's feature folder is the directory two levels above it (`.agents/features/<date>-<slug>/`). Read:

- **The plan**: `ACCEPTANCE CRITERIA` (`AC-n`), `Out of Scope / Non-Goals`, the `Inherited decisions` under `Related
  Work`, and `AMENDMENTS`.
- **The implementation report**: `<feature folder>/reports/<plan-file-stem>-report.md` (if there are `-2` suffixed
  versions, the highest). Its `Deviations from the plan` and `Assumptions carried` are decisions the author already
  made on purpose, and its `Acceptance criteria` table is what the author *claims*, which this review checks.

If there is no report, say so in one line and review against the plan alone. Plans from the original planner lack
`AC-n` and `Related Work`: skip what isn't there; don't invent it.

## 2. Get the changes

```bash
git status
git diff HEAD
git diff --stat HEAD
git ls-files --others --exclude-standard
```

**Clean tree?** If there is nothing uncommitted and the branch has commits ahead of the base (a PR already cut, or a
re-review after `piv-fix-review-findings` committed its fixes), review those commits: `git diff <base>...HEAD`, with
the base detected as `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@'`
(fallback `main`). Record which scope you reviewed in the report. If there is nothing at all, say "nothing to
review" and stop.

Read each new file and each changed file in its entirety (not just the diff) to understand its full context. Skip
lockfiles and generated files; just count them in the stats. The feature folder's own artifacts (`.agents/features/**`:
plan, report, tickets) are the intent behind the change, not the change: read them for context, don't review them as
code.

## 3. Analyze

For each changed or new file, analyze for:

1. **Logic errors**: off-by-one errors, incorrect conditionals, missing error handling, race conditions
2. **Security issues**: SQL injection, XSS, insecure data handling, exposed secrets or API keys
3. **Performance problems**: N+1 queries, inefficient algorithms, memory leaks, unnecessary computations
4. **Code quality**: DRY violations, overly complex functions, poor naming, missing type hints. Report these only
   when they hurt maintainability or break a documented standard; what the linter already enforces is `piv-validate`'s
   job.
5. **Adherence to codebase standards and existing patterns**: documented standards, linting/typing/formatting,
   logging, testing
6. **Plan fit** (only with a plan):
   - **Acceptance criteria**: for each `AC-n`, does the code deliver it? A criterion the report marks `Met` that the
     code doesn't deliver is `high`; `Partial` or unverifiable is a finding too, with what is missing.
   - **Scope**: code that does what `Out of Scope / Non-Goals` excludes, or unrelated to the ticket (`medium`).
   - **Inherited decisions**: code that contradicts one without a documented deviation (`high`).

A **documented deviation is not a finding**. Judge its code like any other (it can still contain a bug), but don't
flag the choice itself. A deviation that is *not* in the report is a finding.

## 4. Verify issues are real

Before reporting, confirm each issue instead of pattern-matching:

- Run specific tests for the issues found
- Confirm type errors are legitimate
- Validate security concerns with context (is the input actually attacker-controlled?)

Drop what doesn't survive. This skill reads and runs checks; it never edits the code under review.

## Output Format

Save a new file to `.claude/code-reviews/<name>.md`, where `<name>` is the plan's file stem plus `-review`
(`t-2-source-adapter-review.md`) or, without a plan, a short descriptive name. Never overwrite an existing review:
add a numeric suffix (`…-review-2.md`), which also keeps the first review next to a re-review.

**Stats:**

- Scope reviewed: uncommitted | `<base>...HEAD`
- Plan: `<path>` | none
- Files Modified: 0
- Files Added: 0
- Files Deleted: 0
- New lines: 0
- Deleted lines: 0

**With a plan, add an acceptance-criteria table** (this is where the call that `piv-validate` leaves to the review
is made):

| Criterion | Delivered | Evidence |
|---|---|---|
| AC-1 — <text> | Yes / Partial / No | `file:line` or the test that shows it |

**For each issue found:**

```
severity: critical|high|medium|low
file: path/to/file.py
line: 42
issue: [one-line description]
detail: [explanation of why this is a problem]
suggestion: [how to fix it]
```

If no issues found: "Code review passed. No technical issues detected."

## Report

After saving, print the path to the review, the count of findings per severity and, with a plan, the `AC-n` that are
not fully delivered. Then the next step: with findings, `piv-fix-review-findings <review path>`; passed, `piv-commit`
(or open the PR if the work is already committed).

## Important

- Be specific (line numbers, not vague complaints)
- Focus on real bugs, not style
- Suggest fixes, don't just complain
- Flag security issues as CRITICAL
