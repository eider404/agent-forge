---
name: piv-run-full-loop
description: Builds one ticket end to end on its own feature branch by chaining the core PIV-loop skills (`prime-codebase` → `piv-plan-implementation` → `piv-implement` → `piv-commit`). Accepts a ticket from a feature's `tickets.md` (`T-n`), a tracker key or URL, a feature folder or the path to its `prd.md` (then it builds the next ready ticket), or a free-form description. Pauses once, at the planner's clarifying questions, then plans, implements, writes the report and commits, stopping early only when something goes wrong. Use when asked to "run the full loop", "build T-2 end to end", "take this ticket from plan to commit" or "build the next ticket of this PRD", or when a ticket should be built hands-off after one round of questions.
argument-hint: "[T-n | ticket key/URL | feature folder .agents/features/<date>-<slug>/ or its prd.md | free-form feature description]"
---

# Run the Full PIV Loop

**Input**: $ARGUMENTS

Build one ticket from priming to commit by chaining four skills: `prime-codebase` → `piv-plan-implementation` →
`piv-implement` → `piv-commit`. One run is one ticket, one branch, one commit. The loop asks the user only once, at
the planner's clarifying GATE; after that it runs to the commit and stops early only when something goes wrong.

Each step follows its own skill (`.claude/skills/<name>/SKILL.md`). This file only adds what chaining them needs:
what each step receives, and where the loop stops.

## Language

Work in the language of the ticket (or the PRD), as the planner does. The plan and the report follow their own
skills' language rules.

## Step 0: Resolve the input and the branch

### 0a. What to build

`$ARGUMENTS` takes the planner's input forms, plus the path to a PRD:

| Input | Goes to the planner as |
|---|---|
| `T-n`, alone or after its feature folder (`<folder> T-2`) | as is |
| A tracker key or URL (Jira / Linear / GitHub) | as is; a Jira key also goes to `prime-codebase` (Step 1) |
| A feature folder `.agents/features/<date>-<slug>/`, or its `prd.md` | decided by the folder's contents (below) |
| A free-form description | as is |

**Feature folder or `prd.md`**: a path to `prd.md` stands for its folder. Look at what the folder holds:

- **`tickets.md`** → the feature is sliced, so build the next ticket. A ticket's state comes from the folder:
  **built** = a `COMPLETE` report `reports/t-<n>-*-report*.md` (highest `-n` suffix); **planned** = a plan
  `plans/t-<n>-*.md` but no `COMPLETE` report, or a branch `feature/t-<n>-*` not merged into the base (`git branch
  --no-merged <base> --list 'feature/t-*'`: from the base branch, an unmerged ticket's plan isn't visible);
  **ready** = neither, and every `Depends on` ticket is built. Check in that order: a squash-merged ticket shows as
  built through its report even if its branch looks unmerged.
  - A **planned** ticket comes first: it was started and left unfinished. Don't plan it again (that would write a
    `-2` plan); stop, name it and its branch, and offer to resume it: switch to that branch, then
    `piv-implement <plan>` and `piv-commit` (or, if it is already committed and only awaits review, open or merge
    its PR). If it is in progress elsewhere on purpose (another worktree), the user can say so and the loop goes on
    with the next ready ticket.
  - Otherwise pick the first **ready** ticket in `Suggested execution order` (document order if there is none),
    pass `<folder> T-n` to the planner and say in one line which one and why; the planner's Phase 0 summary names
    it again, so a wrong pick is caught at its GATE.
  - None ready → stop and list each ticket with its state (built / planned / waiting on T-x). A ticket waiting on a
    built dependency whose branch isn't merged yet won't see its report: merge that PR first.
- **`architecture.md` and no `tickets.md`** → pass the folder: the planner's small-scope path (it warns if the MVP
  exceeds one loop).
- **Only `prd.md`** → stop and suggest `plan-architecture` with that folder. Planning straight from the PRD would
  decide the stack, the data model and the boundaries inside a one-ticket plan, which is built to inherit them.
- **No `prd.md`**, or a `prd.md` outside `.agents/features/<date>-<slug>/` → stop: there is no feature folder for
  the plan and the report to live in.

Never hand the planner a feature folder that has a `tickets.md` without a `T-n`: it would plan the whole MVP and
ignore the slicing.

### 0b. Check the branch before planning

One run is one ticket on its own branch. The planner cuts it (`piv-plan-implementation`, Phase 5, holds the branch
rule); check first what would make the loop stop there anyway:

- **Not a git repository** → stop: the loop ends in a commit.
- **On another ticket's branch** (typically the one the previous run just committed) → stop and ask: switch to the
  base branch (after that ticket's PR is merged, if this one depends on it), or stack this ticket on purpose. Never
  build a second ticket on the first one's branch silently.
- **On the base branch with uncommitted changes outside `.agents/features/`** → stop: commit or stash first.

## Step 1: Prime

Run `prime-codebase`, with the Jira key from 0a if there is one. The planner reuses this priming instead of
analysing the project structure again.

## Step 2: Plan — the one pause

Run `piv-plan-implementation` with the input resolved in 0a.

Its Phase 2 GATE is the loop's pause: post the questions, end the turn and wait. **When the user answers, carry on
with the loop**: finish the plan and go straight to Step 3; don't stop at the planner's report. If the user declines
to answer, the planner records each guess as an `Assumed — …` line, and that is enough to go on.

Note the plan's path (the planner's report prints it); every later step uses it.

## Step 3: Implement

Run `piv-implement` with the plan path. The user is present and has already been asked, so:

- **Its step 3 GATE**: the plan's `OPEN QUESTIONS / ASSUMPTIONS` were already put to the user at the planner's
  GATE. Don't ask them again: proceed, and list the `Assumed — …` lines left under `Assumptions carried`. Ask only
  about an item that GATE never covered.
- **Everything else runs attended**: a task that can only be done by breaking an inherited decision or stepping into
  `Out of Scope` still stops and goes to the user.

`piv-implement` writes its report to `<feature folder>/reports/<plan-file-stem>-report.md`.

## Step 4: Commit — only a COMPLETE ticket

Read the report's `Status`:

- **`COMPLETE`** → run `piv-commit`. The commit carries the code together with the plan and the report.
- **`PARTIAL`** → stop without committing. Say what is left and why (from the report), and the way forward: finish
  the work and run `piv-commit`, or commit it as it is if the user decides so.

## Left out on purpose

`piv-validate` and `piv-review-changes` are not steps of this loop: `piv-implement` already runs every task's
`VALIDATE` and all of the plan's `VALIDATION COMMANDS`, whose levels 1–3 the planner takes from the same source as
`piv-validate` (`## validate`, then CI and manifests), and `COMPLETE` requires them to pass. Run them separately
when a change needs the project's whole suite or a second look, for example before opening the PR.

## Final summary

Print it whether the loop finished or stopped. On a stop, mark the step that stopped with ⏹️ and its reason, and
leave out the steps that didn't run.

```markdown
### Ticket built — <ticket or feature name>

**Input**: <$ARGUMENTS>
**Ticket**: <T-n / tracker key / n/a>   **Feature folder**: `.agents/features/<date>-<slug>/`   **Branch**: `feature/...`

**Steps:**
1. ✅ Prime
2. ✅ Plan — `<plan path>` (questions answered | <n> assumptions)
3. ✅ Implement — `<report path>`, status COMPLETE | PARTIAL
4. ✅ Commit — `<short hash> <subject>` | ⏹️ not committed — <reason>

**Acceptance criteria**: <n> of <m> met; list any not met (from the report)
**Assumptions carried**: <from the report, or "none">
**Files**: <created / modified>; **Tests added**: <list>

**Next steps**: `git push` and open the PR (the report is what the reviewer reads; `piv-review-changes <plan>` first
if the change needs a second look: fix what it finds, then `piv-validate` and `piv-commit`). Merge it, switch back to
the base branch, then the next ticket: `/piv-run-full-loop <feature folder>`.
```

## Success criteria

- ✅ The input became one ticket, a small-scope feature folder or a free-form request; a PRD with no
  `architecture.md` was sent to `plan-architecture`, not planned.
- ✅ The feature branch was cut before planning; nothing was built on the base branch or on another ticket's branch.
- ✅ A planned-but-unbuilt ticket was offered for resuming, not planned again.
- ✅ The user was asked once, at the planner's GATE; `piv-implement` didn't ask the same items again.
- ✅ The loop committed only a `COMPLETE` ticket; a `PARTIAL` one stopped with what is left.
- ✅ The summary names the branch, the plan, the report and the commit (or where and why it stopped).
