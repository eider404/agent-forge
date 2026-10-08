---
name: plan-slice-tickets
description: Slice a feature (its PRD plus its architecture decisions) into PIV-sized tickets with acceptance criteria, per-ticket context and a dependency graph, then write them to `tickets.md` in the feature folder and, after the user confirms, create them in the tracker (Jira via the Atlassian MCP, or GitHub/Linear). Runs after `plan-architecture`; turns the strategic docs into the discrete units of work the PIV loop consumes, one `piv-plan-implementation` per ticket. Use when asked to "slice into tickets", "break the architecture into tickets", "split the epic", "create the backlog from the architecture" or "plan the waves".
argument-hint: "[feature folder: .agents/features/<date>-<slug>/ (or its architecture.md)] (blank = offer the latest architected feature)"
---

# Plan Slice Tickets — Slice a Feature into PIV-Sized Tickets

The bridge between the strategic docs and the PIV loop. The PRD and architecture are the destination; the PIV loop
is the unit of motion; **tickets are the bridge.** This skill does the slicing.

## Input

`$ARGUMENTS` — the **feature folder** `.agents/features/<YYYY-MM-DD>-<slug>/` written by `plan-create-prd` and
`plan-architecture` (it holds `prd.md` and `architecture.md`), or the path to either file.

- **Blank** → look in `.agents/features/` for the most recent folder that has an `architecture.md` and no
  `tickets.md`, offer it ("Slice `<folder>`?") and **GATE**.
- **No `architecture.md`** → stop. The architecture comes first; point to `plan-architecture`. Don't slice from a
  bare PRD or invent the architecture here.
- **A tracker reference** (Confluence/Jira URL or key of the PRD or architecture page) → fetch it via the
  Atlassian MCP (or the equivalent tracker tool available); if the local `.agents/features/*/` copy exists, prefer it.
- **Load-bearing input:** the architecture names the seams, data model, missing pieces and spikes the slices must
  respect.
- **Not prime-dependent.** A primed session helps, but isn't required (Step 2).

## Language

Work in the **language of the PRD**: the conversation, `tickets.md` (headings included) and the closing summary.
Keep the "TBD — needs validation" marker, translated, and never invent a requirement or acceptance criterion:
unknown → mark it TBD.

## Process

### Step 1 — Read the sources

Read both docs fully. From the PRD: MVP (the scope you slice), success metrics, non-goals, open questions. From
the architecture: recommended approach, key decisions, missing pieces, spikes, open questions. Slicing must
respect those calls.

### Step 2 — Orient on the code surface (if not already primed)

Slicing needs enough codebase awareness to judge what is independent vs dependent: file overlap, shared seams.
Skip if the session is oriented. Otherwise **explore it yourself**, starting from the architecture's named seams,
data model and missing pieces: read the relevant files/dirs to see what exists, what is reused, where new code
lands. Just enough to slice confidently, not a full re-derivation.

### Step 3 — Decompose into PIV-sized slices

**Scope for AI, not for a human backlog**: an agent loop carries a small-to-medium implementation *phase*,
~8–10 subtasks, often **500–1500 lines of change (20–50% tests)**. A small feature may be a single ticket. A
well-sized ticket:

- Is **one testable concern**: easy to test, review and prove on its own.
- Is a vertical slice of behavior, not a horizontal layer.
- Has acceptance criteria of its own.
- Fits in one focused loop without context rot.

Split by **dependency**, by **concern**, or as a **slim end-to-end slice** (prove the flow thinly, fatten it next
loop). If a slice is too big to test or review in one honest pass, split it further. The *planning detail* stays
high regardless; only the *scope* is larger.

Carry the architecture through so nothing it decided is lost:

- Each **spike** becomes its own ticket (type `spike`, with its decision rule as acceptance criterion), placed
  before the tickets that depend on its outcome.
- Each **missing piece** lands in at least one ticket.
- An **open question** that blocks a ticket is named in that ticket; don't answer product questions here.
- **Non-goals** and growth beyond the MVP are not ticketed.

### Step 4 — Slice for parallelizability

Map dependencies. **Independent tickets** (no shared files, no reliance on each other's output) can run in
parallel worktrees; mark which are independent and which form a chain. Slicing along vertical-slice seams
maximizes independence.

**Plan just-in-time:** a dependent ticket waits until its dependency is *implemented*, not just sliced; building
the dependency informs the dependent's plan. Independent tickets can be planned and run in parallel.

### Step 5 — Write `tickets.md`

Fill [templates/ticket-breakdown.md](templates/ticket-breakdown.md) and save it as **`tickets.md` in the same
feature folder** (`.agents/features/<YYYY-MM-DD>-<slug>/tickets.md`); reuse the folder, don't create a new one or
a new date. If the input was a tracker URL with no local folder, create `.agents/features/<today>-<slug>/`
(`date +%F`). If `tickets.md` already exists, don't overwrite it: add a numeric suffix (`tickets-2.md`). The local
file is always written.

Every ticket carries its own context, so a loop can pick it up later without re-reading the whole feature. The
`Source`, acceptance-criteria IDs and `Architecture decisions` fields are what `piv-plan-implementation` inherits
from when it plans the ticket.

### Step 6 — Confirm, then publish to the tracker

Creating tickets in a tracker is outward-facing and not reversible with a click. **GATE**: show the ticket list
(titles, dependencies, waves) and where `tickets.md` landed, then stop and wait for the user's OK. Skip this step
if the user has no tracker and wants only the local file.

After the OK, create the tickets via the Atlassian MCP (or the GitHub/Linear tool in use) as children of the
PRD's epic/page, linked to the architecture page. Write the created keys/URLs next to each ticket heading in
`tickets.md` so `Depends on` resolves.

## Output + hand off

Confirm where it landed (the `.agents/features/<YYYY-MM-DD>-<slug>/tickets.md` path, plus the **tracker keys/URLs**
if published); summarize the waves in a few lines. Then each ticket enters its own PIV loop: straight to
`/piv-plan-implementation` if it is well-scoped (it primes what it needs), or `/prime-codebase` first if it needs
more orientation. **Priming is optional**; the per-ticket context is what makes that possible. Wave 1 tickets can
start in parallel.

## Success criteria

- ✅ **Started from a feature folder with an architecture**; no architecture → pointed to `plan-architecture`.
- ✅ **Each ticket is one testable concern**, a vertical slice, one-loop sized, with its own numbered acceptance criteria.
- ✅ **Every ticket carries its context**: `Source`, architecture decisions that bind it, files, size, dependencies.
- ✅ **Architecture honored**: spikes are tickets, missing pieces are covered, blocking open questions are named.
- ✅ **Dependency graph and waves** show what runs in parallel and what waits.
- ✅ **Nothing invented**: unknowns marked "TBD — needs validation".
- ✅ **In the PRD's language**, saved as `tickets.md` next to the PRD and architecture.
- ✅ **Tracker tickets created only after the user confirmed.**

## Notes

- Issue management is **tool-agnostic**: Jira (Atlassian MCP), Linear, GitHub Issues, or just the local `tickets.md`.
  The goal is to **split the work just enough that each loop has the highest chance of one-shot success, so the
  loop can be automated.**
- Brownfield or greenfield, the same slicing applies; the PRD's MVP bounds what gets ticketed.
