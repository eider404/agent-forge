---
name: rules-check-drift
description: "Checks whether the project's rules file (CLAUDE.md or AGENTS.md) and the `## validate` commands in `.claude/references/conventions.md` still match the codebase after recent changes. Reports stale or now-false rules, drifted architecture-map entries, validate commands that no longer exist, and any new invariant worth adding, each with the minimal edit. Advisory and anti-bloat: keeps the rules true, never longer. Use before a merge, after `piv-review-changes`, or when asked to \"check the rules for drift\", \"is CLAUDE.md still accurate\" or \"update the rules after this change\"."
argument-hint: "[optional diff range, e.g. main...HEAD]"
---

# /rules-check-drift — keep your rules file true, not longer

Your rules file — **`CLAUDE.md`** or **`AGENTS.md`** — is a **steering document, not documentation**: your ground
rules, your conventions, and a current **map of where things live**. Its only failure mode that matters is being
**wrong**: a stale rule or a drifted map actively misleads the agent on every future run. This skill checks the
rules file against what just changed and proposes the **smallest** edit that keeps it true.

> **Wrong rules are worse than missing rules. A longer rules file is worse than a lean one.** Most changes
> need *no* edit at all — adding a wrong or verbose line makes it worse.

## Input
- `$ARGUMENTS` — optional diff range. Default: uncommitted + staged (`git diff HEAD`); if the tree is clean (e.g.
  `piv-run-full-loop` already committed the ticket), fall back to `<base>...HEAD`, the base being the repo's default
  branch, detected as `piv-review-changes` does (fallback `main`).
- **Scope: the project's rules file(s)** — `CLAUDE.md` and/or `AGENTS.md`, the root file + any package-level
  ones. (If `CLAUDE.md` is just a `@AGENTS.md` import, check `AGENTS.md`.) Plus the **`## validate` section of
  `.claude/references/conventions.md`**, if it exists: `piv-validate` and the planner run those commands as written,
  so a renamed script or a moved config breaks every loop. Ignore README, `docs/`, the rest of `.claude/references/`
  and `.claude/` agent/command/skill files. This skill exists to keep the *rules* honest, nothing else.

## Process

### 1. See what changed
`git diff <range>` + `git status`. Note: moved/renamed/removed files, new modules, changed conventions,
and any new invariant the change establishes.

### 2. Read the rules file as it is now
Load the project's rules file — `CLAUDE.md` or `AGENTS.md` (and any package-scoped ones). Hold each claim against the change set.

### 3. Flag ONLY these four things
1. **A stated rule or fact is now false** — e.g. "routes live in `src/routes/`" but they moved. → fix it.
2. **The architecture map drifted** — a path or "where things live" pointer no longer matches reality.
   → fix the wrong entry (don't catalog every new file).
3. **A `## validate` command no longer exists** — the script, target or config directory it names was renamed or
   removed (check the manifests and CI the change touched). → fix the one bullet.
4. **A new durable invariant must hold going forward** — the change introduces a rule that must stay true
   (e.g. "never call the DB from handlers — go through `repository/`"). → add it as **one line**.

Everything else, leave alone. Do **not** suggest an edit to *record that a feature was added* (that's a
changelog — the codebase is the source of truth), to restate what the code already makes obvious, or to add
background/rationale/prose that doesn't steer future work.

### 4. Write each suggestion the way CLAUDE.md should read
- **One bullet, not a paragraph.** A rule is a line, not an essay.
- **Keep the map current — don't grow it.** Fix the wrong path; don't enumerate the new ones.
- **State rules in natural language; reference the codebase, never paste code.** Copied code goes stale; the
  codebase stays true. Good: "follow the error pattern in `src/core/errors/`." Bad: pasting the class.

## Output
```
## Rules-file drift check — range: <range>

### Fix (now false)
| Where | What's wrong | Minimal fix |
|-------|--------------|-------------|
| "Architecture" map | routes moved `src/routes/` → `src/api/routes/` | update the one path |
| `conventions.md › ## validate` | `pnpm typecheck` script renamed to `pnpm check:types` | update the one bullet |

### Add (new invariant only)
- <one-line rule> — established by <the change that made it durable>

### Checked, still true — no edit
- <areas you verified need no change>
```
If nothing drifted: **"The rules file is still accurate for these changes — no edits needed."**

## Rules
- **Advisory.** Report the drift; only apply the edits (or commit them via `piv-commit`) if the caller explicitly asks.
- **Rules file and `## validate` only** (`CLAUDE.md` / `AGENTS.md`, `conventions.md › ## validate`). Not README, not docs.
- **Lean by default.** When in doubt, suggest nothing.
- **Run it before every merge** (after `/piv-review-changes`, on the same range) so your rules never drift behind the code.
