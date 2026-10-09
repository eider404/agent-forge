---
name: piv-validate
description: Runs the project's full validation suite (tests, type checks, lint/format across every part of the stack) and reports one PASS/FAIL verdict with a line per check. Finds the commands itself (the project's `## validate` convention, then CI, manifests and Makefile) so it needs no per-project editing. Optionally takes a plan from `piv-plan-implementation` and also runs the plan's own checks, reporting which `AC-n` they cover. Use before committing or opening a PR, after finishing a chunk of work or a round of review fixes, or when asked to "validate", "run all the checks", "run tests and lint", "is everything green" or "check for regressions". Reports only; it never fixes.
argument-hint: "[path-to-plan, e.g. .agents/features/<date>-<slug>/plans/t-2-<name>.md] (optional)"
---

# Validate

Run every check this project has and report a single PASS/FAIL verdict.

## Input

`$ARGUMENTS` is optional: the path to a plan created by `piv-plan-implementation`.

- **Empty**: run the project's suite (step 1).
- **A plan**: run the project's suite, then the plan's own checks (step 2).

## Language

Write the report in the user's language. The verdict (`PASS` / `FAIL`) and the ✅ ❌ ⏭️ markers stay as they are,
because other skills and people scan for them.

## 1. Project checks

### 1.1 Resolve the commands

Never run from memory of what a stack "usually" uses. Take the commands from the project, in this order, and stop at
the first source that answers a given check:

1. **`.claude/references/conventions.md`, section `## validate`**, if it exists. It is where the project says how to
   check itself, and it wins over everything below. Same file and mechanism `piv-commit` uses for `## commit`.
2. **CI workflow** (`.github/workflows/`, `.gitlab-ci.yml`, …). What CI runs is what "green" means for a merge.
3. **Manifests and task runners**: `package.json` scripts, `pyproject.toml`, `Makefile`, `justfile`, `go.mod`,
   `Cargo.toml`, `docker-compose.yml`, the README.

`references/discover-commands.md` has the lookup tables (what to grep per ecosystem, the `## validate` format) and
the traps to avoid. Read it when the project has no `## validate` section.

Resolve three kinds of check for **every surface** (backend, frontend, each package of a monorepo): tests, type
check, lint/format. Each command runs **from the directory that owns its config** (`cd backend && uv run pytest`,
not `uv run pytest` from the root), because many tools only find their config from the current directory.

Print the resolved table (surface, check, command, directory, source) before running anything. A wrong guess is then
visible at once instead of hiding behind a green result.

If a check has no command, mark it `⏭️ skipped — <reason>`; don't invent one. If **nothing** resolves, stop: report
`Overall: FAIL (nothing ran: no validation commands found)` and tell the user to add a `## validate` section or give
the commands.

### 1.2 Run them

Run in order: tests, type check, lint/format, once per surface. Keep going after a failure so the report covers
everything, and capture the output of each failing command.

- **Check mode, non-interactive.** Use the form that only reads and never waits: `ruff check` not `ruff check --fix`,
  `prettier --check` not `--write`, `vitest run` not watch mode. This skill must not change the tree it is judging.
- **The exit code decides**, not the absence of output. A command that did not run to completion is ❌, not ✅.
- **Smoke test** (start the app, hit an endpoint): only if `## validate` or the plan defines one, and only when the
  change touches routing, middleware or startup. Don't improvise server start/stop; a start-in-background-then-kill
  idiom isn't portable.

## 2. Plan checks (only with a plan)

Read the plan and run, without repeating what step 1 already ran:

- **`VALIDATION COMMANDS`**, levels 1–3: every command. Level 4 (manual): run the steps you can from here; list the
  ones that need a person as `Not verified — needs a human`. Level 5: run if the tool is available, otherwise
  `⏭️ skipped`.
- **Each task's `VALIDATE` command**, so each `AC-n` can be tied to a result: group them by the task's `SATISFIES`.

An `AC-n` is `✅ covered` only if at least one command grouped under it ran and every one passed; `❌` if any of
them failed; `— no automated check` if nothing is grouped under it. Say plainly that this reports what the commands
show, not that the criterion is accepted: that call belongs to the review.

Plans from the original planner may lack `SATISFIES` or `AC-n`. Skip the grouping and just report the commands; don't
invent the mapping.

## 3. Report

```
## Validation — <project or feature>

| Surface | Check | Result | Command |
|---|---|---|---|
| backend | tests | ✅ | `cd backend && uv run pytest` |
| frontend | lint | ❌ | `cd frontend && pnpm lint` |
| backend | smoke | ⏭️ skipped — no smoke command defined | |

### Plan checks (only with a plan)
| AC | Result | Evidence |
|---|---|---|
| AC-1 | ✅ covered | `<command>` |
| AC-2 | — no automated check | |

Not verified — needs a human: <list, or "none">

**Overall: PASS** | **Overall: FAIL**
```

- `Overall: PASS` only if at least one check ran and none failed. Skipped checks don't fail the run, but list each
  with its reason so a gap in coverage is never silent.
- For every ❌: the failing command and the relevant output (the failing test, the lint rule and line, not the whole
  log).
- **Do not fix anything here.** Say what failed; fixing is a separate step.

Close with the next step: on PASS, `piv-review-changes` (or `piv-commit` if the work was already reviewed); on FAIL,
fix what failed and run this again.

## Notes

- Keep this fast. It runs before every commit; if a step is slow, that is a signal to fix the slow step, not to drop
  it from the checker.
- A checker that cannot fail is worthless. If every check is ✅ on a tree you expected to be broken, suspect the
  checker (wrong directory, a command that exits 0 on error) before trusting the result.
