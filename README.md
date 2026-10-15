# agent-forge

> Forge production-quality code with AI agents.

**agent-forge** is a skills kit for Claude Code that takes a feature from PRD and architecture through tickets, plans, implementation, review and commits, with a repeatable flow and rules that never drift out of sync with the code.

## How it works

There are 12 skills chained into two loops. Each one leaves a file that the next one reads, so you can stop the flow at any point and resume it later.

```
Outer loop (once per feature)
  PRD → architecture → project rules → tickets

Inner loop (once per ticket)
  prime → plan → implement → validate → review → commit
```

- **What to build:** `plan-create-prd`, `plan-architecture`, `rules-create-global`, `plan-slice-tickets`.
- **How to build it (PIV: Plan → Implement → Validate):** `prime-codebase`, `piv-plan-implementation`, `piv-implement`, `piv-validate`, `piv-review-changes`, `piv-commit`, or all together with `piv-run-full-loop`.
- **Rules kept current:** `rules-check-drift` verifies that `CLAUDE.md` and the validation commands are still true.

## Quick start

```bash
mkdir -p <your-project>/.claude/skills
cp -r .agents/skills/* <your-project>/.claude/skills/
```

Open Claude Code at the root of your project and run, for example:

```
/plan-create-prd
/piv-run-full-loop .agents/features/<date>-<slug> T-1
```

## Documentation

The full guide (installation, workflows and usage of each skill) is in [GUIA.md](GUIA.md).
