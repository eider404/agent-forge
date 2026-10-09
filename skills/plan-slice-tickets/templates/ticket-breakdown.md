# Ticket Breakdown — <product / PRD name>

Source: <path (and tracker URL, if any) of the PRD> · <path (and tracker URL, if any) of the architecture>

<!-- Content (titles, scope, criteria, context) in the PRD's language. Headings, field labels, IDs, Type values
     and the "TBD — needs validation" marker stay in English, verbatim: piv-plan-implementation and
     piv-run-full-loop read them by name. Skip nothing silently: write "TBD — needs validation" for unknowns. -->

## Feature summary
Goal in 2–3 lines (from the PRD's problem and thesis), scoped to the MVP.

## Tickets

### T-1 — <title> <(tracker key/URL, once published)>
- **Type:** build | spike
- **Scope:** one testable concern.
- **Acceptance criteria:**
  - AC-1 — <verifiable criterion>
  - AC-2 — <verifiable criterion> (a spike's criterion is its decision rule: go with X if signal / Y if counter-signal)
- **Architecture decisions:** the sections of `architecture.md` that bind this ticket
  (e.g. "Key decisions › Data model · Missing pieces › source adapter").
- **Per-ticket context:** the PRD sections, guides and seams this ticket needs
  (e.g. "PRD › MVP · seam: adapter interface").
- **Blocked by open questions / spikes:** <none / named question or spike ticket>
- **Files touched (estimate) · size:** <files> · ~500–1500 lines incl. tests
- **Depends on:** <none / T-x>

### T-2 — <title>
(same fields)

## Dependency graph
Text or mermaid graph showing the order and the parallel groups.

## Suggested execution order
- Wave 1 (parallel): T-1, T-3
- Wave 2: T-2 (after T-1 is implemented)
