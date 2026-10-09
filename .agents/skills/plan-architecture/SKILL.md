---
name: plan-architecture
description: Interactively explore HOW to build a PRD and decide the high-level architecture — the approach, stack, libraries, data shape, boundaries, and risks the PRD deliberately left open — in the same language as the PRD. Runs only after `plan-create-prd`; it always needs a PRD as input and sends the user to `plan-create-prd` first if there is none. A working session with a CTO/staff-engineer advisor that asks questions, proposes 2–3 options with trade-offs, recommends a direction with reasoning, and flags what to de-risk with a spike. Writes a high-level architecture decision doc to `architecture.md` in the PRD's feature folder, `.agents/features/<YYYY-MM-DD>-<kebab-slug>/` (plus a linked page in the tracker when the PRD was published there) — NOT a task-by-task implementation plan (that comes later, per ticket, with `piv-plan-implementation`).
argument-hint: "[feature folder: .agents/features/<date>-<slug>/ (or its prd.md)] · [optional: paths to reference docs to ground in] (blank = offer the latest PRD)"
---

# Architect: Explore the Approach, Decide the Architecture

**Input**: $ARGUMENTS — the **PRD** produced by `plan-create-prd`, normally given as its feature folder
(`.agents/features/<YYYY-MM-DD>-<slug>/`, which holds `prd.md`) or the path to that `prd.md`. This skill runs **only after the PRD**; it never starts from a bare idea,
an epic or a brief.

- **Blank** → look in `.agents/features/` for the most recent folder that has a `prd.md` and no `architecture.md`
  (date-prefixed folders sort chronologically), offer it ("Architect `<folder>`?") and **GATE**.
- **No PRD exists** (or what was passed is not a PRD) → stop. Say the PRD comes first and point to
  `plan-create-prd`; don't improvise one here.
- **A tracker reference** (a Confluence/Jira URL or key of the PRD's page) → fetch it via the Atlassian MCP; if the
  local `.agents/features/*/prd.md` copy exists, prefer that one.

**Reference docs (optional):** if any paths were passed alongside the PRD — API docs, product/engineering
docs, ADRs, prior research, a competitor teardown, a Confluence page — **read them first.** They ground the
exploration so you propose options that fit what already exists instead of inventing. If none were passed, **ask
whether any exist** before you start exploring — a lot of the context you need is usually already written down.

## Language

Work in the **language of the user's input**: the language the PRD was written in (or, if the PRD is
ambiguous, the language of their first answer). Use it for everything the user reads: the
conversation, the architecture doc (section headings included) and the closing summary. If the input mixes
languages, follow the dominant one. Two things stay in English, verbatim, because later skills find them by name:
the marker **`TBD — needs validation`** and the spike field **`Outcome:`**.

**Anti-fluff rule:** never invent a decision, constraint or requirement. Unknown → **"TBD — needs validation"**.

## What this skill is

The PRD says **what** to build and **why**. This skill decides **how to approach it** — the eng-lead-level
calls the PRD left open: the approach, the stack and libraries, the data model, the boundaries, and what's
risky enough to test first.

**This is a high-level decision doc, not an implementation plan.** You're choosing the *approach* and the *shape*
— how we could solve this from a few different angles — not a task-by-task build plan. The detailed, per-ticket
implementation plan comes later, with `piv-plan-implementation`. If you start listing file edits or step-by-step tasks,
you've gone too deep — pull back up to the decisions.

## Reading the PRD (required input)

The PRD from `plan-create-prd` is product-level and deliberately leaves engineering open — those are *your*
calls. **Read it fully before asking anything**, **by meaning, not by literal heading** (its headings follow the
PRD's language), and use each part like this:

| PRD part | Use it as |
|---|---|
| Problem Statement · Thesis · Target User & JTBD | the goal lens every decision is judged against |
| Hypothesis (the RIGHT / WRONG condition) | what the architecture must make cheap to test — the WRONG signal has to be observable |
| MVP | the scope you architect: the thinnest end-to-end slice, not the full product. Note growth the shape must not block; don't design it |
| Success Metrics | what must be measurable (events, data captured) |
| Non-goals | boundaries — don't architect for them |
| Open Questions | engineering ones: settle or defer here. Product ones: carry them over, don't answer them |
| MVP › door check | spike-vs-build: one-way door → spike first; two-way door → decide and move on |
| Constraints | givens every option must respect |

If a part is missing or "TBD", ask instead of guessing. If the PRD slipped in engineering decisions (library,
data model, security…), treat them as proposals to **confirm with the user**, not as silent givens.

## This is an interactive skill

The conversation **is** the deliverable. Don't one-shot a document. Run this loop, out loud, with the user:

```
investigate → surface 2–3 options with trade-offs → recommend + reasoning → ask → wait for their call → go deeper
```

Every ask is a **GATE**: post the questions, then stop. End the turn and wait for the answers — never ask and
answer in the same breath, and never roll into the next topic. Reflect thin answers back and dig.

Ask sharp clarifying questions whenever the PRD or the user's goals are unclear — a *grill-me* posture beats a
confident wrong guess. Never converge on a single answer silently.

**If they decline the discussion** ("just decide"): honour it, but name what you would have to guess and offer the
two or three highest-leverage questions instead of all of them. Write your recommended default for each call
marked **"TBD — needs validation"**, never as a settled decision.

## Your role

A pragmatic **CTO / staff-engineer advisor**. You **propose, you don't dictate.** Optimize for:

- **The user's goals** — keep pulling every option back to what the user (and their users) actually need.
- **Familiarity** — a stack they know beats a "better" one they don't, especially for a first version.
- **Leanness** — decide only what's needed to move forward; don't over-architect.
- **Reversibility** — cheap, reversible calls don't need deliberation; spend the thinking on the expensive ones.
- **More than one option** — a good problem has >1 viable answer. Show the alternatives, then recommend.

## Greenfield vs brownfield (branch on the input)

**Know which mode you're in first.** Infer it from the input and the workspace — a PRD with no real codebase yet
= greenfield; a PRD for something to build on a product that already has a codebase = brownfield. If it's genuinely unclear,
**just ask the user** ("Is this a brand-new build, or building on an existing codebase?"). It changes how you
explore:

- **Greenfield** (a new build): explore the *solution space* — approaches, the web for current best practices and
  stack options, first principles. The architecture is what you *decide*.
- **Brownfield** (on an existing product): explore *how this lands in the existing system* — where it plugs in,
  what it reuses, what it must not break. **Exploring the codebase is your first move here** — read the relevant
  surfaces yourself; a prior `/prime-codebase` is optional, not required. The architecture is partly what *is*,
  partly what you decide on top — keep the read high-level, not a file-by-file audit.

## What to explore (interactively)

Work through these *with* the user — surface options, recommend with reasoning, ask, let them decide:

- **Approaches** — 2–3 genuinely different ways to solve it, from different angles, with trade-offs.
- **Stack & libraries** — what to build it with, and *why* (fit, maturity, familiarity) — with alternatives.
- **Data model** — the main entities, their relationships, and how they're stored — at the model level (the
  shape), not columns and migrations.
- **Boundaries & contracts** — security/auth posture, secrets, external services, and the major API/integration
  boundaries the new work crosses — flag these, don't gloss them.
- **Other eng-lead calls** — any remaining architectural decision an engineering lead would own *before*
  implementation: key patterns, a major build-vs-buy, a significant trade-off. The shape, not the task list.
- **First principles** — what fundamentally has to be true for this to work.
- **Missing pieces** — what doesn't exist yet that the chosen approach needs (often the real work).
- **Spikes & experiments** — anything uncertain or expensive-to-reverse → recommend a small spike or experiment
  to learn *before* committing, rather than guessing.

Recommend a direction for each, with the reasoning, and let the user make the call. Skip what doesn't apply — and
say so, don't silently omit it.

## Spikes (for the risky / one-way calls)

When a decision is uncertain or expensive to undo (a one-way door), recommend a **spike** instead of guessing:

```
Question:      [what we're unsure about]
Spike:         [the smallest thing we can build or test to learn] over [timebox]
Decision rule: go with [X] if [signal] / [Y] if [counter-signal]
Outcome:       (empty until the spike is built — its ticket writes which side of the rule won, and the evidence)
```

A spike without an `Outcome:` is an unsettled decision: `rules-create-global` leaves it out, and the tickets it
blocks wait for it.

Reversible, low-cost calls → just decide and move on.

## The output: a high-level architecture decision doc

Only after the calls are made. Save it as **`architecture.md` in the same feature folder as the PRD**
(`.agents/features/<YYYY-MM-DD>-<slug>/architecture.md`), so the pair lives together; reuse the folder, don't create a new one or a new date.
If the PRD was given as a tracker URL with no local folder, create `.agents/features/<today>-<slug>/` (`date +%F`).
The filename is always `architecture.md`: one canonical file per feature, which every later skill reads. If it
already exists, **GATE**: show what would change and ask before overwriting it (git keeps the previous version). If
tickets were already sliced from it, say that `tickets.md` may need re-slicing. The local file is always written.

**If the PRD was also published in a tracker** (a Confluence/Jira page or epic) **or the user names a
destination**, also publish the same content there — via the Atlassian MCP — as a **separate page, child of the
PRD's page/epic, linked both ways**: the PRD stays pure intent, the architecture (the *how*) lives beside it, and
each links to the other. Two clean, linked sources are what let `plan-slice-tickets` and `piv-plan-implementation`
read intent and architecture separately later. Don't fold the architecture into the PRD — a PRD never carries
engineering decisions.

Keep it high-level and fill this shape (headings in the user's language):

```markdown
# Architecture — <product / PRD name>

Source: <path (and tracker URL, if any) of the PRD this answers>

## Problem & goals
One paragraph: the user goal this serves (from the PRD's problem and thesis) — the lens every decision
below is judged against.

## Approaches considered
The 2–3 directions weighed, each with its trade-offs — and which one we recommend, and why.

## Recommended approach
The chosen direction in a few sentences — the shape of the solution, scoped to the MVP, not the task list.
(Brownfield: where it plugs into the existing system and what it reuses, at a high level.)

## Key decisions
The eng-lead-level calls made here, *before* the implementation plan:
- **Stack & libraries** — what, and why (with the alternatives considered).
- **Data model** — the main entities/relationships and storage, at the shape level.
- **Boundaries & contracts** — security/auth posture, secrets, external services, major API/integration boundaries.
- **Measurability** — what must be captured to tell the hypothesis RIGHT from WRONG (from the PRD's hypothesis
  and success metrics).
- **Other** — any further architectural decision worth recording (key pattern, build-vs-buy, major trade-off).
- (skip any that don't apply — note that you did)

## Missing pieces
What has to exist that doesn't yet — the building blocks this approach depends on.

## Spikes & experiments
The uncertain / expensive calls to de-risk first, each in the Spike block above (Question · Spike · Decision rule ·
`Outcome:`, empty for now).

## Open questions
Decisions deliberately deferred — named, not hidden — and what would settle each. Product questions carried over
from the PRD stay here, marked as the PRD owner's.
```

## After this

Confirm where you wrote it (the `.agents/features/<YYYY-MM-DD>-<slug>/architecture.md` path, plus the **tracker page URL**
if one was published), summarize the recommended approach + the key calls in a few lines, then offer the natural
next moves and let the user pick — **don't force a pipeline**:

- **Set the global rules** (greenfield; recommended before slicing) — run `/rules-create-global` with the feature
  folder: it turns these settled decisions into the root `CLAUDE.md`/`AGENTS.md` that every ticket's loop reads.
- **Slice it into tickets** — run `/plan-slice-tickets` with the feature folder (PRD + this architecture doc) to break the work
  into PIV-sized tickets, and create the GitHub issues / Jira tickets from them.
- **Keep going here** — stay in this conversation to refine the decisions, or to create the issues/tickets directly.
- **Small scope? Plan it in one go** — skip slicing and go straight to `piv-plan-implementation` for the implementation plan.
- **Spike something now** — if an open risk is blocking, go build the spike/experiment we flagged.

## Success criteria

- ✅ **Started from a PRD** — never from a bare idea; no PRD → pointed to `plan-create-prd`.
- ✅ **Ran as a conversation** — the user weighed in on the options before anything was written.
- ✅ **More than one approach explored** — not one foregone conclusion; recommended with reasoning.
- ✅ **Stack & libraries recommended with the *why*** and the alternatives.
- ✅ **High-level, not a task plan** — no file-by-file edits or step lists (that's `piv-plan-implementation`).
- ✅ **Risky / one-way calls get a spike**, not a guess.
- ✅ **Grounded in the PRD** — scoped to its MVP; its hypothesis, metrics and non-goals shaped the decisions.
- ✅ **Nothing invented** — unknowns marked "TBD — needs validation".
- ✅ **In the user's language**, saved as `architecture.md` next to the PRD in its feature folder (overwritten only
  after the user confirmed).
- ✅ **Stays anchored to the user's goals.**
