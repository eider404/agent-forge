---
name: prime-codebase
description: Primes the agent with deep codebase understanding by analyzing structure, documentation, key files and recent git activity, then reports a scannable summary. Use when the user says "prime the codebase", "orient me", "get up to speed on this repo", starts a new session, or needs a fast orientation before planning or implementing a ticket. Optionally loads external task context from Jira issues and Confluence pages first, via the Atlassian MCP.
argument-hint: "[jira-issue-keys] [confluence-page-ids]"
---

# Prime: Load Codebase Context

## Objective

Build comprehensive understanding of the codebase by analyzing structure, documentation, and key files. If external task references are provided, load them first so the codebase analysis is anchored to the actual work.

## Process

### 1. Load External Context (optional)

**Run this step BEFORE the codebase analysis.** It accepts optional arguments: `[jira-issue-keys] [confluence-page-ids]`.

- Jira keys may be a single key (`ACC-2`) or comma-separated (`ACC-2,ACC-3`).
- Confluence page ids are numeric page ids.

Use the Atlassian MCP for both.

**If Jira issue keys are provided:** get the Atlassian `cloudId`, then fetch each issue in markdown. Treat the issue summary, description, and acceptance criteria as the task context for everything that follows.

**If Confluence page ids are provided:** fetch each page in markdown (reusing the `cloudId` if already retrieved). Treat the page content as supporting context (specs, design docs, requirements).

**If no arguments are provided:** skip this step and go to step 2.

Briefly summarize any external context loaded before continuing — this frames the rest of the priming.

### 2. Analyze Project Structure

List tracked files (first 500):
!`(git ls-files 2>/dev/null || echo "not a git repository") | head -500`

Show directory structure (falls back to `find` where `tree` isn't installed):
!`tree -L 3 -I 'node_modules|__pycache__|.git|dist|build' 2>/dev/null || find . -maxdepth 3 -not -path '*/.git*' -not -path '*/node_modules*' -not -path '*/__pycache__*' 2>/dev/null | head -300`

If the list was cut at 500, widen only where the task needs it (`git ls-files <dir>`).

### 3. Read Core Documentation

- Read CLAUDE.md or AGENTS.md (the global rules file) and `.claude/references/conventions.md` if present
- Read README files at project root and major directories
- Read any architecture documentation

### 4. Identify Key Files

Based on the structure, identify and read:
- Main entry points (main.py, index.ts, app.py, etc.)
- Core configuration files (pyproject.toml, package.json, tsconfig.json)
- Key model/schema definitions
- Important service or controller files

### 5. Understand Current State

Check recent activity:
!`git log -10 --oneline 2>/dev/null || echo "no git history"`

Check current branch and status:
!`git status 2>/dev/null || echo "not a git repository"`

## Output Report

Provide a concise summary covering:

### External Task Context (if loaded)
- Jira issue(s): key, title, one-line goal, acceptance criteria
- Confluence page(s): title and what they specify

### Project Overview
- Purpose and type of application
- Primary technologies and frameworks
- Current version/state

### Architecture
- Overall structure and organization
- Key architectural patterns identified
- Important directories and their purposes

### Tech Stack
- Languages and versions
- Frameworks and major libraries
- Build tools and package managers
- Testing frameworks

### Core Principles
- Code style and conventions observed
- Documentation standards
- Testing approach

### Current State
- Active branch
- Recent changes or development focus
- Any immediate observations or concerns

**Make this summary easy to scan - use bullet points and clear headers.**
