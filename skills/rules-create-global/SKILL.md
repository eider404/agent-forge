---
name: rules-create-global
description: "Creates the project's global rules: a lean root CLAUDE.md (or AGENTS.md) with a codebase map, specific ground rules, working principles and where new code goes, plus the `.claude/references/` files other skills read (`conventions.md` with `## validate` and `## commit`, and per-topic stubs). Greenfield: takes a feature folder (`.agents/features/<date>-<slug>/`) or the PRD and architecture paths, and derives the rules from the architecture's decisions. Brownfield: derives them from the codebase (primed, or a codebase-analysis doc). Use when asked to \"create the global rules\", \"set up CLAUDE.md\", \"replace /init\", \"derive the rules from the architecture\" or \"onboard this repo to the AI layer\", or after `plan-architecture` settles the stack. The customizable replacement for /init."
argument-hint: "[feature folder .agents/features/<date>-<slug>/ | prd-path architecture-path | codebase-analysis-path] (blank = offer the latest architected feature, or derive from the codebase)"
arguments: [path1, path2]
---

# Create Rules: Set Up Your Project's Global Rules

**Input**: `$path1` `$path2`, both optional. Reads the technical decisions or the code, and writes the always-on rules
file plus the `.claude/references/` files the PIV skills look up. Never invent a rule: no source, no line.

## Language

Work in the language of the input: the architecture doc, else the PRD, else the existing rules file, else the user's
first message. Use it for the conversation, the rules file and the report. Paths, commands and identifiers stay
verbatim. The headings `## validate` and `## commit` in `conventions.md` stay exactly as written, because
`piv-validate` and `piv-commit` find them by name. The marker `TBD — needs validation` stays in English, verbatim,
in every artifact of the flow; that is also how you spot unsettled decisions in the architecture.

## Pick the lane

Classify each path by its content, by meaning and not by literal heading (headings follow the document's language):

| A passed path that is... | Lane | Use it as |
|---|---|---|
| A feature folder `.agents/features/<date>-<slug>/` | greenfield | its `architecture.md` is the source; `prd.md` is context |
| An architecture doc from `plan-architecture` (or a tracker URL/key of its page: fetch it via the Atlassian MCP, prefer the local copy) | greenfield | the technical decisions the rules derive from |
| A PRD from `plan-create-prd` | greenfield | product context ("what this is") only; a PRD never carries engineering decisions |
| A codebase-analysis doc | brownfield | the source of "what is" |

- **PRD with no architecture** (a PRD path, or a folder without `architecture.md`): stop and point to
  `plan-architecture`. Technical rules don't live in a product spec.
- **Nothing passed**: look in `.agents/features/` for the most recent folder with an `architecture.md`. If the
  workspace is a bare scaffold, offer that folder ("Derive the rules from `<folder>`?") and **GATE**. Otherwise it is
  brownfield. If unclear, ask: "Brand-new build, or an existing codebase?"

## What goes where

The root file holds what is true on every task: a **map** of the codebase, **ground rules** (the specific
conventions), **working principles** (how the agent should operate: planning posture, ask vs assume, scope, the
engineering primitives the user holds) and the **commands** it runs itself. Sort every candidate line:

- **Always-on** (root file): map, ground rules, working principles, seams.
- **On-demand**: a recurring but task-type-specific pattern goes to `.claude/references/<topic>.md` (a stub of a
  paragraph, not a doc). Test: does it recur every time that area is touched?
- **A plan**: "what to build next" was never a rule; it belongs in the ticket's plan.
- **Delete**: redundant lines, and slogans the model already follows ("write clean code").

State the choice, not the slogan. Brownfield is "what is": every rule points to the file that proves it, or it stays
out, because aspirational rules make the agent fight the codebase. Stay lean: cut anything that wouldn't cause a
mistake if removed.

## Before writing: protect what exists

Pick the file: `CLAUDE.md` (Claude Code), `AGENTS.md` (the cross-tool standard, https://agents.md), or both. When both
exist, the shared content usually lives in `AGENTS.md` and `CLAUDE.md` is a single `@AGENTS.md` line: write to
`AGENTS.md`. Neither exists: `CLAUDE.md` for a Claude-only project, `AGENTS.md` for a multi-tool team. Before drafting,
read the matching guidance (https://code.claude.com/docs/en/memory and
https://code.claude.com/docs/en/best-practices for `CLAUDE.md`), not a remembered snapshot.

If the rules file exists, read it and build on it. Before overwriting, check `git status --porcelain -- <file>`: if
it has uncommitted changes, or the project is not in git, copy it to `<file>.bak` (never over an existing `.bak`) and
say so in the report.

## Workflow

### 1. Read the inputs

**Greenfield**: read the architecture and map its parts (by meaning) to a destination:

| Architecture part | Becomes |
|---|---|
| Recommended approach · Missing pieces | the map (target structure) and the seams |
| Key decisions: stack & libraries, boundaries & contracts, other durable calls | ground rules |
| Key decisions: data model, measurability | a rule only if it constrains code (where models live, naming); otherwise a stub |
| Spikes with an `Outcome:` | the decision they settled: a ground rule if it is durable |
| Spikes with no `Outcome:` · Open questions · anything marked `TBD — needs validation` | **not rules**: unsettled is not a rule. Leave them out and count them for the report |

The PRD supplies the one-paragraph "what this is". If the folder holds scaffold files, spot-check the map against them.

**Brownfield**: read the codebase-analysis doc (with its `file:line` citations), else the codebase already loaded in
this conversation by `prime-codebase`. Not primed and no doc: run `/prime-codebase`, or read the minimum yourself:
the tree, the manifests, the CI config, the existing rules file and a few entry points. Spot-check the code either
way.

### 2. Elicit the working principles: GATE

These cannot be derived from code, and they are the part an agent won't follow unless the user states them. Post one
numbered cluster of 3–6 questions, each with a **recommended default** so answering is cheap, then stop and wait:

1. When to plan first vs just do it (default: plan before anything that touches more than one concern).
2. Ambiguity: ask or assume (default: ask).
3. Scope discipline (default: only what the ticket asks; no refactors of adjacent code).
4. What "done" means (default: the real checks pass, not "it should work").
5. Engineering primitives they hold: fail fast, explicit errors, single responsibility, simplest thing that works.
6. Commit format, only if the git history, `CONTRIBUTING` or a commitlint config don't already settle it.

Keep only answers that change behavior and reflect the user's stance. If they decline, leave the section out and say
so in the report: a generic list is the filler this section exists to avoid.

### 3. Write the root file

Fill [templates/claude-md.md](templates/claude-md.md) from steps 1 and 2. Link to the PRD or the analysis; don't
paste them. Delete any section with nothing real in it.

### 4. Write `conventions.md`

`.claude/references/conventions.md` is the file `piv-validate` and `piv-commit` read, so the project's specifics live
there and those skills stay general. Write only the sections that have content, and never rewrite a file that exists:
read it and add the missing sections.

- **`## validate`**: one bullet per check, with its command and directory, in the format of
  `.claude/skills/piv-validate/references/discover-commands.md`, which also says where to find the commands (CI first,
  then manifests). List only commands that exist in the manifests or CI, or that the architecture names and the
  scaffold supports. Never a guess: `piv-validate` runs what is listed as written. Nothing verifiable yet (a
  greenfield with no scaffold): omit the section and say it in the report; discovery covers it.
- **`## commit`**: only where the project deviates from `piv-commit`'s defaults (`<tag>: <description>`, about 72
  characters). Take it from the git history, `CONTRIBUTING` or the user's answer to question 6:

  ```markdown
  ## commit

  - tags: feat, fix, docs, refactor, test, chore, perf
  - subject: `<tag>(<scope>): <description>`, with the ticket key in the body
  ```

Coding standards stay in the root file (ground rules); this file only holds what other skills parse.

### 5. Extract on-demand context

Write the recurring, task-type-specific patterns as `.claude/references/<topic>.md` stubs. Add each one to the root
file's "Load on demand" with its trigger.

### 6. Prune to lean

The first draft is always too big. Delete generic advice, restated defaults and anything that can't point to its
evidence. Per-line test: would removing it cause a mistake? If not, cut it.

### 7. Report

- Files created or changed, and any `.bak` made.
- 3–5 lines on what went into the root file and why; what went to `conventions.md` and to on-demand stubs.
- What was left out on purpose: decisions still TBD or unresolved spikes (count them), commands you couldn't verify,
  the working principles if the user declined.
- Next step. Greenfield: `plan-slice-tickets <feature folder>`, or `piv-run-full-loop <folder>` for a small scope
  (it plans the whole MVP as one ticket). Brownfield: the first loop, `piv-run-full-loop` or `piv-plan-implementation`.
  When a spike ticket later records its `Outcome:`, run this skill again on the folder to pick up the settled decision.

## Success criteria

- ✅ Started from technical decisions or real code; a PRD alone → pointed to `plan-architecture`.
- ✅ The root file is a **map + ground rules**, not documentation or a copy of the PRD or the analysis.
- ✅ Every ground rule is a specific choice (greenfield: from `architecture.md`; brownfield: traceable to a file).
- ✅ Nothing unsettled became a rule: TBD decisions and open spikes were left out and counted.
- ✅ Working principles were **elicited** (GATE), lean and behavior-changing, or the section is absent.
- ✅ `conventions.md` has only verifiable `## validate` commands and a `## commit` only if the project deviates; an
  existing file was never rewritten.
- ✅ Recurring task-type detail lives in `.claude/references/`, not always-on.
- ✅ An existing rules file was read first and backed up when it could be lost.

## Notes

- Rules evolve: revisit the root file as the project grows and after major model releases. Run `/rules-check-drift`
  before merges; map entries that are real paths, and the `## validate` commands, are what it can check.
- `piv-review-changes` takes the root file as its quality bar and `piv-plan-implementation` reads it and
  `.claude/references/` when it plans, so a wrong or bloated line costs on every ticket.
