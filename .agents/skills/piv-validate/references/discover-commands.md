# Discovering a project's validation commands

Read this when the project has no `## validate` section in `.claude/references/conventions.md`. The goal is the
commands the project itself already trusts, not a guess at the stack's usual tools.

## The `## validate` section (what a project can write to skip discovery)

One bullet per check, in the order they should run. The directory goes in parentheses when it isn't the root:

```markdown
## validate

- backend tests: `uv run pytest` (backend/)
- backend types: `uv run mypy .` (backend/)
- backend lint: `uv run ruff check .` (backend/)
- frontend tests: `pnpm test --run` (frontend/)
- frontend types: `pnpm tsc --noEmit` (frontend/)
- frontend lint: `pnpm lint` (frontend/)
- smoke: `make smoke`
```

Anything listed here is used as written. Anything not listed falls through to discovery.

## Where to look, by source

| Source | What to read |
|---|---|
| CI workflow | The `run:` steps of the job that gates merges. Skip deploy, release and publish jobs. |
| `package.json` | `scripts`: `test`, `typecheck` / `type-check` / `tsc`, `lint`, `format:check`. The package manager is the one whose lockfile exists. |
| `pyproject.toml` / `tox.ini` / `noxfile.py` | `[tool.pytest]`, `[tool.mypy]`, `[tool.ruff]`, `[tool.pyright]`; the runner is `uv run` / `poetry run` / plain, whichever the lockfile and CI use. |
| `Makefile` / `justfile` | Targets named `test`, `check`, `lint`, `typecheck`, `ci`, `verify`. A single `ci` / `check` target that bundles the rest can replace several rows; say so in the table. |
| `go.mod` | `go test ./...`, `go vet ./...`, plus `golangci-lint run` if configured. |
| `Cargo.toml` | `cargo test`, `cargo clippy -- -D warnings`, `cargo fmt --check`. |
| README / CONTRIBUTING | Only to confirm a command found elsewhere, or when nothing else answers. |

A **monorepo** has one surface per package or workspace directory that carries its own manifest. Resolve each one;
don't assume the root `test` script covers them all (check what it does).

## Traps

- **Watch and interactive modes.** `jest --watch`, a bare `vitest`, `ng test` without `--watch=false`, and `npm test`
  scripts that open a watcher never return. Use the CI form (`vitest run`, `--ci`, `--watchAll=false`).
- **Fix modes.** Anything with `--fix`, `--write`, `-w` or `format` (not `format:check`) edits the tree. Use the check
  variant, or skip the check and say why.
- **Wrong directory.** If a tool's config sits in a subdirectory, running from the root either fails or silently
  checks nothing. The directory goes in the table.
- **Exits 0 on error.** Some wrappers swallow the exit code (`cmd || true`, a script that only echoes). If a command
  you resolved does that, report it as a gap rather than counting it as ✅.
- **Two candidates for one check.** Prefer the one CI runs; mention the other in the table's source column.
- **POSIX-only idioms** (`lsof`, `python3`, background-then-`kill`) break for teammates on Windows. Prefer the
  project's own task runner.
