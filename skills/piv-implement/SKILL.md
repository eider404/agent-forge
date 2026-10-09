---
name: piv-implement
description: Executes an implementation plan task by task on a feature branch, running each task's own validation before moving on, then writes an implementation report in the feature folder's `reports/`. Takes the path to a plan created by `piv-plan-implementation` (`.agents/features/<date>-<slug>/plans/<id>-<name>.md`). Use when asked to "implement this plan", "implement T-2", "execute the plan" or "build what the plan says", or when a finished plan is ready to be built in one pass.
argument-hint: "[path-to-plan, e.g. .agents/features/<date>-<slug>/plans/t-2-<name>.md]"
---

# Execute: Implement from Plan

## Plan to Execute

Read plan file: `$ARGUMENTS`

If `$ARGUMENTS` is empty, propose the most recent file under `.agents/features/*/plans/` and confirm it before
going on.

The plan's feature folder is the directory two levels above it (`.agents/features/<date>-<slug>/`). A plan outside
that layout (an older one in `.claude/plans/`) has no feature folder.

## Language

Write the report's content in the plan's language. Its headings stay in English, like the plan's.

## Before you start

### 1. Work on a feature branch

A ticket gets built on its own branch, so it can become one PR. **Ideally you're already on that branch (cut it
before planning), so the plan commit rides into the PR; a plan committed on the base branch won't be in it.** If
you're still on base, this step creates the branch now. Detect the base branch (don't hardcode `main`):
`git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@'` (fallback `main`).

- **On the base branch, clean** → create one: `git checkout -b feature/<plan-slug>`, where `<plan-slug>` is the
  plan's file name without `.md` (`feature/t-2-source-adapter`).
- **Already on a feature branch or in a worktree** → use it.
- **On the base branch with uncommitted changes** → STOP: commit or stash first.

(One branch per ticket is also what makes parallel worktrees clean later.)

### 2. Read the plan and what it binds you to

- Read the ENTIRE plan: tasks, testing strategy, validation commands, acceptance criteria.
- Read every file listed under `CONTEXT REFERENCES › Relevant Codebase Files` and the code behind `Patterns to
  Follow`. The plan marks them mandatory because its tasks assume you know them.
- Treat `Out of Scope / Non-Goals` and the `Inherited decisions` under `Related Work` as binding. They were decided
  upstream (PRD, architecture, ticket); building beyond them, or against them, is not yours to decide.
- Plans written by the original planner lack some of these sections (`SATISFIES`, `AC-n`, `Related Work`,
  `AMENDMENTS`). Skip what isn't there; don't invent it.

### 3. Gate on open items

Read `OPEN QUESTIONS / ASSUMPTIONS`. Anything still open (an `Assumed — …, confirm before execution` line, an
unanswered question, a `TBD — needs validation` marker) is a bet the plan made for the user, and building on it
silently turns it into code that is costly to undo.

**GATE:** post those items in one cluster, numbered, each with the plan's assumption as the recommended default,
then stop and wait for the answers. If nothing is open, say so in one line and continue.

If the caller runs you unattended (a factory prompt, or "don't ask"), don't wait: proceed with each assumption as
written and list it in the report under `Assumptions carried`.

## Execution Instructions

### 4. Execute Tasks in Order

Run the phases and tasks top to bottom. The `Depends on` / `Independent of` notes on phases only matter for parallel
runs, which this skill doesn't do.

For EACH task in `STEP-BY-STEP TASKS`:

#### a. Navigate to the task
- Identify the file and action required
- Read existing related files if modifying, and the code the `PATTERN` line points to

#### b. Implement the task
- Follow `IMPLEMENT` exactly, with the `IMPORTS` it names, and honour the `GOTCHA`
- Maintain consistency with existing code patterns
- Include proper type hints and documentation
- Add structured logging where appropriate

#### c. Verify as you go
- After each file change, check syntax, imports and types
- **Run the task's own `VALIDATE` command before starting the next task.** Every task carries one. A task is not
  done until its check passes; if it fails, fix it now rather than carrying the failure forward. The full suite
  still runs at step 6; this per-task gate keeps step 6 from becoming a pile-up.
- Fix the source, not the check: never weaken a test or assertion to get green. If the check itself is wrong, say so
  in the report's deviations.

#### d. Record deviations as they happen
Different file, pattern or library, a skipped or added task, anything outside the plan: write down what and why at
that moment, not from memory at the end. If a task can only be done by breaking an inherited decision or stepping
into `Out of Scope`, stop and flag it to the user instead of choosing. Unattended: don't deviate, stop at that
task and report `PARTIAL`.

### 5. Implement Testing Strategy

After the implementation tasks:

- Create all test files specified in the plan
- Implement all test cases in `TESTING STRATEGY` (unit, integration, edge cases)
- Ensure tests cover edge cases

### 6. Run Validation Commands

Execute ALL commands under `VALIDATION COMMANDS` in order, exactly as written:

- Levels 1–3 (syntax & style, unit, integration): run every command. If one fails, fix the issue, re-run it, and
  continue only when it passes.
- Level 4 (manual validation): run the steps you can from here (API calls, CLI). For those that need a person (visual
  UI checks), list them in the report as `Not verified — needs a human`.
- Level 5 (optional tools): run it if the tool is available, otherwise say it was skipped.

### 7. Final Verification

Before completing:

- ✅ All tasks from plan completed
- ✅ All tests created and passing
- ✅ All validation commands pass
- ✅ Code follows project conventions
- ✅ Documentation added/updated as needed

In the plan file, tick (`- [x]`) the `ACCEPTANCE CRITERIA` and `COMPLETION CHECKLIST` items you verified and leave
the rest unticked. For each deviation that makes the plan stop describing what was built, append one dated line to
`AMENDMENTS` (what changed and why), so the tickets that depend on this one aren't misled by the plan.

## Output — write an implementation report

Write the report to `<feature folder>/reports/<plan-file-stem>-report.md` (create `reports/`) and print the
summary. Use the plan's own file name so `piv-plan-implementation` finds it when planning a ticket that depends on
this one. Never overwrite an existing report: add a numeric suffix (`…-report-2.md`).

- Plan outside a feature folder: use `.agents/features/<today>-<plan-slug>/reports/` (`date +%F`).
- The caller gave an explicit report path: use that one.

This is what a PR reviewer reads, especially the **deviations** (a documented deviation is an *intentional*
decision the reviewer should not flag), and what the next tickets build on:

```markdown
# Implementation Report — <feature>

**Plan**: <path>   **Ticket**: <T-n / tracker key / n/a>   **Branch**: <feature/...>   **Status**: COMPLETE | PARTIAL

## Summary
{What was built, 2-4 sentences. For PARTIAL: what is left and why.}

## Tasks completed
- [task] → `path/to/file` (CREATE/UPDATE) — AC-n

## Acceptance criteria
| Criterion | Status | Evidence |
|---|---|---|
| AC-1 — <text> | Met / Partial / Not met | tasks, test or command that shows it |

## Tests added
{Test files + cases + results.}

## Validation results
{Type-check / lint / tests / build — pass/fail with counts. Manual steps not verified.}

## Deviations from the plan
{What changed vs the plan and WHY — or "none". This is the reviewer's signal of intent.}

## Assumptions carried
{Open items you proceeded on without confirmation — or "none".}

## Built (for dependent tickets)
{Contracts the next tickets will rely on, as actually built: endpoints, signatures, schema or migration names,
config keys — or "none".}

## Issues encountered
{Anything notable, or "none".}
```

`COMPLETE` means every task is done, every validation passes and every `AC-n` is met. Anything less is `PARTIAL`.

### Ready for the next step
- Confirm all changes are complete and validations pass.
- Next: `piv-commit` the work. The report sits in the feature folder, so it goes into the PR with the code.

## Notes

- If you encounter issues not addressed in the plan, document them
- If you need to deviate from the plan, explain why
- If tests fail, fix implementation until they pass
- Don't skip validation steps
