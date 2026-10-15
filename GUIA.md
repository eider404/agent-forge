# Skills v2 usage guide

What the skills in this repository's `.agents/skills/` are, how to install them, which workflows they cover and how to run each one.

---

## 1. What they are

They are **12 Claude Code skills**: markdown instructions (`.agents/skills/<name>/SKILL.md`, plus their `templates/` and `references/`) that the agent loads when you invoke them. Together they form an AI-assisted development flow in **two loops**:

- **Outer loop: what to build.** Runs once per feature: PRD → architecture → project rules → tickets.
- **Inner loop (PIV: Plan → Implement → Validate): how to build it.** Runs once per ticket: prepare context → plan → implementation → commit.

```
OUTER LOOP (once per feature)
  plan-create-prd → plan-architecture → rules-create-global → plan-slice-tickets
                                         (greenfield, or when
                                          the rules change)
                                                                    │
INNER LOOP (once per ticket)                                        ▼
  piv-run-full-loop = prime-codebase → piv-plan-implementation → piv-implement → piv-commit
                                                                    │
BEFORE THE PR (optional)                                            ▼
  piv-validate → piv-review-changes → (fix → piv-validate → re-review) → rules-check-drift
```

Each skill leaves an **artifact** (a file) that the next one reads. That is why you can cut the flow at any point, close the session and resume later: state lives on disk, not in the conversation.

### The skills by group

| Group | Skill | In one sentence |
|---|---|---|
| **Plan** | `plan-create-prd` | Interviews you and writes a problem-centered PRD (what and why, never how) |
| | `plan-architecture` | Working session with a "CTO" to decide stack, data shape, boundaries, risks and spikes |
| | `plan-slice-tickets` | Cuts the feature into PIV-loop-sized tickets, with `AC-n` criteria and a dependency graph |
| **Prime** | `prime-codebase` | Orients the agent in the repo (structure, docs, conventions, git) before planning |
| **PIV** | `piv-plan-implementation` | Creates the ticket's branch and a plan ready to implement in one pass |
| | `piv-implement` | Executes the plan task by task, validating each one, and writes the report |
| | `piv-validate` | Runs the project's whole suite (tests, types, lint) and gives a PASS/FAIL verdict |
| | `piv-review-changes` | Technical review of the change (and against the plan, if you pass it); report only |
| | `piv-commit` | One atomic commit with a conventional message, no secrets or build artifacts |
| | `piv-run-full-loop` | Builds a ticket end to end with a single pause for questions |
| **AI layer** | `rules-create-global` | Writes `CLAUDE.md`/`AGENTS.md` and `.claude/references/conventions.md` (the customizable `/init`) |
| | `rules-check-drift` | Checks whether the rules and the `## validate` commands are still true after a change |

---

## 2. Installation

1. From the root of this repository, copy the 12 folders in `.agents/skills/` to `<your-project>/.claude/skills/`:

   ```bash
   mkdir -p <your-project>/.claude/skills
   cp -r .agents/skills/* <your-project>/.claude/skills/
   ```

2. Open Claude Code **at the root of your project**.

> **Install them in `<your-project>/.claude/skills/`, not only in `~/.claude/skills/` or `.agents/skills/`.** In this repository they live in `.agents/skills/`, but the skills reference each other with paths relative to the target project (`.claude/skills/<name>/…`; for example, `piv-plan-implementation` reads `.claude/skills/piv-validate/references/discover-commands.md`). If they are anywhere else, those references break.

Requirements: git (the inner loop ends in a commit per branch). Optional: `gh` for GitHub Issues, the Atlassian MCP for Jira/Confluence, and `tree` (if it is not installed, `prime-codebase` uses `find`).

---

## 3. How to run a skill

There are two ways:

- **Explicit, with a slash:** `/<skill-name> <arguments>`. For example:
  ```
  /piv-run-full-loop .agents/features/2026-10-08-csv-export T-2
  ```
- **Implicit, in natural language:** the agent picks the skill from its `description`. For example: "plan T-2", "review my changes", "commit".

Things worth knowing when running them:

- **Without arguments, most infer their input:** they offer the most recent feature folder that fits (the latest one with `prd.md` and no `architecture.md`, the latest one with architecture, etc.) and name it so you can correct it.
- **Interactive workflows stop at a `GATE`:** they post a batch of questions, each with a recommended *default*, and wait for your answer. You can reply "accept the defaults". Whatever isn't known is not invented: it is marked `TBD — needs validation` or `Assumed — …`.
- **When finished, each skill suggests the next step** with the exact command.

---

## 4. Key concepts

### The feature folder

Everything a feature generates lives together and goes into the PR with the code:

```
.agents/features/<YYYY-MM-DD>-<slug>/
  prd.md                          ← plan-create-prd
  architecture.md                 ← plan-architecture   (spikes fill in their Outcome:)
  tickets.md                      ← plan-slice-tickets
  plans/t-2-<name>.md             ← piv-plan-implementation
  reports/t-2-<name>-report.md    ← piv-implement
  reviews/t-2-<name>-review.md    ← piv-review-changes
```

- `prd.md`, `architecture.md` and `tickets.md` are **canonical**: there is one per feature and they are only overwritten if you confirm.
- Plans, reports and reviews are **never overwritten**: a second version gets the suffix `-2`, `-3`… and the highest-numbered one is the current one.

What belongs to the project rather than to a feature lives outside that folder: `CLAUDE.md`/`AGENTS.md` at the root and `.claude/references/conventions.md`.

### `conventions.md`: how to validate and how to commit

`.claude/references/conventions.md` is created by `rules-create-global` and you can edit it. It has two sections that other skills read by their literal name:

- **`## validate`**: one bullet per check, with its command and directory. Used by `piv-validate`, `piv-plan-implementation` (for the plan's `VALIDATION COMMANDS`) and `rules-check-drift`. If missing, the skills discover the commands from CI and the manifests (`package.json`, `pyproject.toml`, `Makefile`…).
- **`## commit`**: exists only if your project departs from the conventional-commits defaults. Used by `piv-commit`.

### One ticket = one branch = one PR

- `piv-plan-implementation` creates the branch `feature/<id>-<name>` (for example, `feature/t-2-source-adapter`) **before** writing the plan.
- If you are on another ticket's branch, the planner and the full-loop stop and ask you.
- **If a ticket depends on another, merge the earlier PR first:** from the base branch, the previous ticket's report is only visible once it has been merged.

### Ticket state

`piv-run-full-loop` infers it from the feature folder:

| State | Signal |
|---|---|
| **built** | There is a `reports/t-<n>-…-report.md` with `Status: COMPLETE` |
| **planned** | There is a plan, or an unmerged `feature/t-<n>-…` branch, but no `COMPLETE` report |
| **ready** | It is neither planned nor built, and all its dependencies (`Depends on`) are *built* |

### Language

You write in your language and the documents come out in your language. The **structure** that other skills read stays in English: headings and fields of `tickets.md`, the plan and the report, the `T-n`/`AC-n` IDs, values such as `COMPLETE`/`PARTIAL` and `Met`/`Partial`/`Not met`, and the markers `TBD — needs validation`, `Assumed — …` and `Outcome:`. Do not translate that structure by hand: the loop looks for it by its exact name.

---

## 5. Workflows

### Workflow A: new project (greenfield), from idea to first PR

This is the complete flow and the case the skills were designed for.

```
1. /plan-create-prd <idea> · [research docs]
      → .agents/features/2026-10-06-thread-grouping/prd.md
2. /plan-architecture .agents/features/2026-10-06-thread-grouping
      → architecture.md (stack, data, boundaries, spikes)
3. /rules-create-global .agents/features/2026-10-06-thread-grouping
      → CLAUDE.md + .claude/references/conventions.md + stubs
4. /plan-slice-tickets .agents/features/2026-10-06-thread-grouping
      → tickets.md (+ tracker if you confirm)
5. /piv-run-full-loop .agents/features/2026-10-06-thread-grouping
      → picks the next ready ticket, plans it (pauses for questions),
        implements it, writes the report and commits it on its branch
6. (optional) /piv-validate · /piv-review-changes <plan> · /rules-check-drift main...HEAD
7. push → PR → merge → go back to the base branch → repeat step 5 with the next ticket
```

Notes:

- Steps 1 and 2 have several GATEs: it is a conversation, not a document generated in one shot.
- Step 3 only turns settled decisions into rules. Those marked `TBD` and spikes without an `Outcome:` do not become rules. Run it again when a spike records its result.
- In step 5 passing the folder is enough: the loop picks the ticket by `Suggested execution order` and `Depends on`, and tells you which one it chose and why. To force a specific one: `/piv-run-full-loop <folder> T-3`.

### Workflow B: new feature in an existing project (brownfield)

```
1. /rules-create-global
      (only the first time you bring the repo into the flow)
      → derives CLAUDE.md and conventions.md from the existing code
2. /plan-create-prd <idea>
3. /plan-architecture <folder>
      → detects it is brownfield and explores the code before proposing
4. /plan-slice-tickets <folder>
5. /piv-run-full-loop <folder>   (once per ticket, as in workflow A)
```

If the repo already has a `CLAUDE.md` and a `conventions.md` with `## validate`, skip step 1. `rules-create-global` never rewrites an existing `conventions.md`.

### Workflow C: small scope, no tickets

If the feature fits in a single loop, there is no need to cut it into tickets:

```
1. /plan-create-prd <idea>
2. /plan-architecture <folder>
3. /piv-run-full-loop <folder>
      (or /piv-plan-implementation <folder> → /piv-implement <plan> → /piv-commit)
```

With a folder that has `architecture.md` but no `tickets.md`, the planner takes the small-scope path: it proposes criteria derived from the MVP and warns you if the MVP does not fit in one loop. In that case, go back to `plan-slice-tickets`.

> The loop does **not** plan directly from a folder that only has `prd.md`: it stops and suggests `plan-architecture`. If it planned from the PRD, it would decide the stack and the data model inside a single-ticket plan.

### Workflow D: one-off change (bugfix, tweak, loose task)

For something that doesn't deserve a PRD, use free text or a tracker key:

```
/piv-run-full-loop Add pagination to the GET /orders endpoint (25 per page)
/piv-run-full-loop ACC-42
/piv-run-full-loop https://github.com/org/repo/issues/17
```

The plan and report are saved in a new folder, `.agents/features/<today>-<slug>/`. If the description clearly matches an existing feature, the planner offers to use that folder.

### Workflow E: step by step, without the full-loop

Use it when you want to review or edit the plan before building, or leave the code uncommitted so you can review it:

```
1. /prime-codebase                                   (optional; the planner reuses its output)
2. /piv-plan-implementation <folder> T-2             → creates the branch and writes plans/t-2-<name>.md
      ✋ review or edit the plan (AMENDMENTS section if you change anything)
3. /piv-implement .agents/features/<…>/plans/t-2-<name>.md
                                                     → code + reports/t-2-<name>-report.md
4. /piv-validate <plan>                              (optional)
5. /piv-review-changes <plan>                        (optional)
6. /piv-commit
```

Differences from the full-loop:

- `piv-implement` run on its own has its own GATE if the plan carries open assumptions.
- In the full-loop, those questions were already asked at the planner's GATE and are not repeated.

### Workflow F: resume an interrupted or `PARTIAL` ticket

- **The session was cut after the plan:** run `/piv-run-full-loop <folder>` again. It detects the *planned* ticket, tells you which one it is and which branch it is on, and offers to resume it with `piv-implement <plan>` + `piv-commit`, instead of writing a `-2` plan.
- **The report ended up `PARTIAL`:** the loop stops **without** committing and explains what is missing and why. Finish the work (by hand or by asking the agent) and run `/piv-commit`. You can also decide to commit the work as it stands.
- **No ticket is ready:** the loop lists each ticket with its state (*built* / *planned* / waiting on `T-x`). The usual cause is that a dependency's PR has not been merged yet.

### Workflow G: quality control before the PR

The full-loop does not include `piv-validate` or `piv-review-changes`: `piv-implement` already runs the plan's checks, and a `COMPLETE` report requires them to pass. Add this cycle when the change needs the full suite or a second pair of eyes:

```
/piv-validate                         → the project's full suite: Overall PASS/FAIL
/piv-review-changes <plan>            → reviews/<plan>-review.md + AC-n → Yes/Partial/No table
   findings? → fix (by hand or by asking the agent with the review file)
             → /piv-validate → /piv-review-changes <plan>   (repeat until clean)
/piv-commit                           (if the fixes left uncommitted changes)
/rules-check-drift main...HEAD        → are the rules and ## validate still true?
```

`piv-validate` and `piv-review-changes` **only report, they never fix**. `rules-check-drift` only suggests changes. If you want to apply them, ask for it and they are committed with `piv-commit`.

### Workflow H: spikes (risky decisions)

1. `plan-architecture` marks each irreversible or risky decision as a spike, with `Question · Spike · Decision rule · Outcome:`. The `Outcome:` is left empty.
2. `plan-slice-tickets` turns each spike without an `Outcome:` into a ticket of type `spike`. Tickets that depend on that decision are blocked (`Blocked by open questions / spikes`).
3. Build the spike like any other ticket (`/piv-run-full-loop <folder> T-1`). The last task of its plan writes the result into the `Outcome:` in `architecture.md`, and `piv-implement` verifies it.
4. Run `/rules-create-global <folder>` again if the decision should become a project rule.

### Workflow I: with a tracker (Jira, GitHub, Linear)

- `plan-create-prd` publishes the PRD to the tracker (Confluence/Jira) if you ask it to. If the PRD lives there, `plan-architecture` also publishes the architecture as a child page, linked both ways.
- `plan-slice-tickets` always writes `tickets.md` and, **only after your confirmation**, creates the tickets in the tracker.
- The PIV skills accept the ticket's key or URL: `/piv-run-full-loop ACC-30`. A Jira key is also passed to `prime-codebase` to anchor the exploration.
- The local folder takes precedence over the tracker when both exist.

---

## 6. Reference by skill

### `plan-create-prd`

**What it does:** interviews you in phases (foundation → deepening → hypothesis → MVP and *door check*) to write a product PRD: problem, evidence, thesis, falsifiable hypothesis, user and JTBD, constraints, MVP, metrics, non-goals and open questions. It makes no engineering decisions.

**Running it:**

```
/plan-create-prd                                         # starts by asking
/plan-create-prd Group threads when volume exceeds ~100/day
/plan-create-prd Group threads · docs/research/surveys.md   # reads the research first as evidence
```

**Produces:** `.agents/features/<date>-<slug>/prd.md` (and a tracker page, if you ask).

**Next step:** `plan-architecture`.

### `plan-architecture`

**What it does:** iterative session with a CTO/staff-engineer advisor. It proposes 2–3 approaches with their trade-offs, recommends one, and decides stack, libraries, data shape, security boundaries and contracts, and which decisions need a spike. It detects whether the project is greenfield or brownfield. It requires a PRD.

**Running it:**

```
/plan-architecture                                       # offers the latest PRD without architecture
/plan-architecture .agents/features/2026-10-06-thread-grouping
/plan-architecture .agents/features/2026-10-06-thread-grouping/prd.md · docs/adr/
```

**Produces:** `architecture.md` in the PRD's folder (if it already exists, it asks for confirmation before overwriting it).

**Next step:** `rules-create-global` (greenfield) and then `plan-slice-tickets`; or `piv-plan-implementation` if the scope is small.

### `plan-slice-tickets`

**What it does:** breaks the feature into tickets that one loop can resolve at once (~500–1500 lines of change, one testable concern per ticket). Each ticket carries `Type` (`build`/`spike`), `Scope`, `Acceptance criteria` (`AC-n`), `Architecture decisions`, `Per-ticket context`, `Files touched` and `Depends on`. It also generates the dependency graph and the suggested execution order.

**Running it:**

```
/plan-slice-tickets                                      # offers the latest architected feature
/plan-slice-tickets .agents/features/2026-10-06-thread-grouping
```

**Produces:** `tickets.md`. When re-slicing, tickets that already have a plan or report keep their number. After your confirmation, it also creates the tickets in the tracker.

**Next step:** `piv-run-full-loop <folder>`.

### `prime-codebase`

**What it does:** orients the agent. It reads the structure (`tree`/`find`), `CLAUDE.md`/`AGENTS.md`, `conventions.md` and the READMEs; identifies entry points, configuration and key models; and checks git state (it also works outside a repo). It returns a scannable summary.

**Running it:**

```
/prime-codebase
/prime-codebase ACC-2,ACC-3            # first loads those Jira issues
/prime-codebase ACC-2 4521098          # Jira issue + Confluence page
```

**Produces:** only the summary in the conversation. The planner reuses it.

### `piv-plan-implementation`

**What it does:**

1. Reads the ticket, the architecture decisions that affect it, the PRD scope and the plan and report of its dependencies.
2. Analyzes the code and asks a short batch of questions (GATE).
3. Researches the external documentation.
4. **Creates the branch** and writes the plan in the shape of `templates/plan.md`: inherited decisions, patterns to follow, atomic tasks (`IMPLEMENT/PATTERN/IMPORTS/GOTCHA/VALIDATE/SATISFIES`), `VALIDATION COMMANDS` and `ACCEPTANCE CRITERIA`.

**Running it:**

```
/piv-plan-implementation .agents/features/2026-10-08-csv-export T-2
/piv-plan-implementation T-2                             # uses the latest folder with tickets.md
/piv-plan-implementation ACC-30                          # tracker key or URL
/piv-plan-implementation .agents/features/2026-10-08-csv-export   # small scope, no tickets
/piv-plan-implementation Add login with JWT              # free text
```

**Produces:** the branch `feature/<id>-<name>` and `plans/<id>-<name>.md`.

**Next step:** `piv-implement <plan>`.

### `piv-implement`

**What it does:**

1. Verifies it is on the plan's branch. If the plan was written on the base branch, it creates the branch and carries over the changes in the feature folder.
2. Runs a GATE if the plan carries open assumptions.
3. Implements task by task and runs each one's `VALIDATE` before moving to the next.
4. Runs all the `VALIDATION COMMANDS` and marks the criteria met.

It respects the plan's `Out of Scope` and the inherited decisions.

**Running it:**

```
/piv-implement .agents/features/2026-10-08-csv-export/plans/t-2-source-adapter.md
```

**Produces:** the code and `reports/<plan>-report.md`, with `Status` (`COMPLETE`/`PARTIAL`), the `AC-n` table, deviations, `Assumptions carried` and `Built (for dependent tickets)`. In a spike, it verifies that the `Outcome:` was written in `architecture.md`.

**Next step:** `piv-commit`.

### `piv-validate`

**What it does:** discovers the commands (`## validate` → CI → manifests), shows you the table of resolved commands and runs them all: tests, type-check and lint/format for every part of the stack. It keeps going even if one fails and returns one row per check and an `Overall: PASS/FAIL` verdict. A check without a command appears as `⏭️ skipped`, with its reason. It writes no files and fixes nothing.

**Running it:**

```
/piv-validate                                            # the project's full suite
/piv-validate .agents/features/<…>/plans/t-2-<name>.md   # + the plan's checks and AC-n → result table
```

**Next step:** `piv-review-changes`, or `piv-commit`.

### `piv-review-changes`

**What it does:** reviews the change by reading each file in full: logic, security, performance and adherence to `CLAUDE.md` and `conventions.md`. It reviews uncommitted changes or, with a clean tree, the `<base>...HEAD` commits. With a plan, it also reviews the `AC-n`, the `Out of Scope` and the `Inherited decisions`. Deviations documented in the report do not count as findings; an `AC-n` declared `Met` that the code does not satisfy does. It only reports.

**Running it:**

```
/piv-review-changes
/piv-review-changes .agents/features/<…>/plans/t-2-<name>.md
```

**Produces:** `reviews/<plan>-review.md`, with `severity/file/line/issue/detail/suggestion` findings and, if you pass a plan, the `AC-n → Yes/Partial/No` table. Without a plan, it saves it in the affected feature's folder or in a new one dated today.

**Next step:** fix → `piv-validate` → repeat the review; or `piv-commit`.

### `piv-commit`

**What it does:** creates a single atomic commit with all pending changes and a `<tag>: <description>` message. If `conventions.md › ## commit` exists, its rules take precedence. It leaves secrets (`.env`, keys, credentials, also inside a file's content) and build files or files over ~5 MB out of the commit, and tells you what it left out. **It does not push.**

**Running it:**

```
/piv-commit
```

**Produces:** the commit, and a summary with the hash, what changed, `Left Out of the Commit` and `AI Layer Changes` if `.claude/` was touched.

### `piv-run-full-loop`

**What it does:** chains `prime-codebase` → `piv-plan-implementation` → `piv-implement` → `piv-commit` for **one** ticket. It stops only once, at the planner's GATE, and after that goes all the way to the commit unless something fails.

**Running it:**

```
/piv-run-full-loop .agents/features/2026-10-08-csv-export        # next ready ticket
/piv-run-full-loop .agents/features/2026-10-08-csv-export/prd.md # same
/piv-run-full-loop .agents/features/2026-10-08-csv-export T-3    # a specific ticket
/piv-run-full-loop ACC-30                                        # tracker ticket
/piv-run-full-loop Add pagination to GET /orders                 # free text
```

**It stops if:**

- you are not in a git repository;
- you are on another ticket's branch;
- there are uncommitted changes on the base branch outside `.agents/features/`;
- the folder only has `prd.md` (it suggests `plan-architecture`);
- there is a *planned* ticket (it offers to resume it);
- no ticket is ready;
- the report ends up `PARTIAL` (it does not commit).

**Afterwards:** push → PR → merge → go back to the base branch and run the loop again.

### `rules-create-global`

**What it does:** writes a lean root `CLAUDE.md` (or `AGENTS.md`): code map, where new code goes, concrete conventions, commands and the working principles you choose (it asks in a GATE). It also creates `.claude/references/conventions.md` (with a verifiable `## validate` and, if needed, `## commit`) and context stubs. In greenfield it derives the rules from `architecture.md`; in brownfield, from the code (already primed with `prime-codebase`, from an analysis document, or from a minimal read).

**Running it:**

```
/rules-create-global                                     # latest architected feature, or the code
/rules-create-global .agents/features/2026-10-06-thread-grouping/
/rules-create-global docs/prd.md docs/architecture.md    # loose paths
/rules-create-global docs/codebase-analysis.md           # brownfield with an analysis document
```

**Next step:** `plan-slice-tickets` (greenfield) or the first PIV loop (brownfield). Later on, `rules-check-drift`.

### `rules-check-drift`

**What it does:** compares `CLAUDE.md`/`AGENTS.md` and the `## validate` in `conventions.md` against the diff. It reports only four kinds of problem: a rule that is now false, an outdated map entry, a `## validate` command that no longer exists, and a new invariant that deserves a line. If nothing really changed, it suggests nothing. It is advisory: it does not edit.

**Running it:**

```
/rules-check-drift                     # detects the base like piv-review-changes
/rules-check-drift main...HEAD
```

**Next step:** if you ask to apply the changes, they are committed with `piv-commit`.

---

## 7. Cheat sheet: "I want to…"

| I want to… | Run |
|---|---|
| Start a product or feature from an idea | `/plan-create-prd <idea>` |
| Decide how to build a PRD | `/plan-architecture <folder>` |
| Generate `CLAUDE.md` and the validation commands | `/rules-create-global [<folder>]` |
| Cut the feature into tickets | `/plan-slice-tickets <folder>` |
| Build the next ticket end to end | `/piv-run-full-loop <folder>` |
| Build a specific ticket | `/piv-run-full-loop <folder> T-n` |
| Fix something small without a PRD | `/piv-run-full-loop <description>` |
| Only plan and review the plan before building | `/piv-plan-implementation <folder> T-n` |
| Build an already-written plan | `/piv-implement <plan-path>` |
| Know whether everything is green | `/piv-validate [<plan>]` |
| A code review before the PR | `/piv-review-changes [<plan>]` |
| Commit what's pending without leaking secrets | `/piv-commit` |
| Check that `CLAUDE.md` is still true | `/rules-check-drift main...HEAD` |
| Have the agent get to know the repo | `/prime-codebase [jira-keys]` |

---

## 8. Complementary tools (original, optional)

They are not part of the flow, no v2 skill depends on them, and they are **not included in this repository** (`.agents/skills/` only contains the 12 flow skills). If you want them, install them separately in `<your-project>/.claude/skills/`.

- **`agent-browser`**: browser automation (Chrome over CDP) to navigate, fill forms, click, take screenshots or extract data. Before using it, load its real guide with `agent-browser skills get core`.
- **`ast-grep`**: structural code search by AST, for example "async functions without try/catch", with `ast-grep run` or `ast-grep scan`.
