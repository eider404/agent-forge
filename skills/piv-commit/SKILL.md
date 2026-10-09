---
name: piv-commit
description: 'Creates one atomic git commit with a conventional-commit message (`<tag>: <description>`) for all uncommitted changes, keeping secrets (`.env`, private keys, credentials) and build output or oversized files out of history and reporting what it left out. Use when work is complete and ready to be committed, or when asked to "commit", "commit this", "commit my changes" or "save this work to git". Does not push.'
---

# Commit: Create a New Commit

Create one commit for all of our uncommitted changes, without letting a secret or a build artifact into history.

## Process

0. **Read this project's conventions.** If `.claude/references/conventions.md` exists, read its `## commit`
   section and follow it — those rules win over the defaults below. That file is where a project's specifics
   live; this skill stays general.
1. **See what changed.** Run `git status && git diff HEAD && git status --porcelain`. Untracked files do not
   appear in the diff: open the new ones you need to understand, both to describe them and to check them for
   secrets (step 2).
2. **Stage everything, then take out what must not be committed.** Run `git add -A`, then review what is
   staged (`git diff --cached --name-status`). A commit is permanent and shared: a pushed secret has to be
   rotated, and a binary stays in every clone. So hold back, with `git reset -q -- <path>` (the file stays on
   disk; the command also works before the first commit), any added or modified file that is:
   - **Secret-bearing by name**: `.env` and `.env.*` (not `.env.example`, `.env.sample`, `.env.template`),
     `*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.keystore`, `id_rsa*`, `id_ed25519*`, `credentials*`,
     `service-account*.json`, `*.tfstate`.
   - **Secret-bearing by content**: a private-key block, a cloud or API token (`AKIA…`, `ghp_…`, `sk-…`), a
     password inside a URL, or any other credential hard-coded in the staged diff.
   - **Build or dependency output, or large**: anything under `node_modules/`, `dist/`, `build/`, `.venv/` or
     `__pycache__/`, a `*.log`, or a file over about 5 MB. Find the large ones with
     `git diff --cached --name-only -z --diff-filter=AM | xargs -0 -r du -k | awk '$1>5120'`.

   Deleting such a file is fine to commit; only additions and modifications are held back. If nothing is left
   staged, do not commit: say so and stop. Don't edit `.gitignore` on your own; suggest it in the report.
3. **Write the message.** Subject `<tag>: <description>`: tag one of `feat`, `fix`, `docs`, `refactor`, `test`,
   `chore`, etc. that reflects the work; imperative, about 72 characters. Add a body only for what the subject
   cannot carry. A single commit is the loop's contract (one ticket, one commit), so don't split the work. If the
   diff clearly mixes unrelated work, name the main change in the subject and list the rest in the body.
4. **Commit.** Pass the message through a heredoc so line breaks survive: `git commit -F - <<'EOF'` … `EOF`.

## Output

A single commit with the staged changes and a conventional-commit-style message
(`<tag>: <atomic description>`) that accurately reflects the work done.

After the commit succeeds, start with its short hash and subject (`git log -1 --oneline`), then print these
summaries:

### What Changed
One short paragraph (3–6 sentences) describing the feature/fix/refactor that was committed — what problem it solves and what files were the key touch points. Write for a developer skimming the git log. If the commit mixes unrelated work, say so here so the user can split it.

### Left Out of the Commit
Only include this section if step 2 held something back. List each file with the reason (secret by name, secret in content, build output, size) and what to do about it (rotate the credential, add it to `.gitignore`, commit it by hand if it was intended). If nothing was held back, omit this section entirely.

### AI Layer Changes
Only include this section if any files under `.claude/` were modified or added (CLAUDE.md, `.claude/references/`, `.claude/skills/`, `.claude/agents/`, etc.).

List each changed AI-layer file with a one-line note on what evolved and why. If nothing in `.claude/` changed, omit this section entirely.
