# <Project name>

<One paragraph: what this is and who it is for. Then the stack in one line.>

## Architecture map

<The dirs and files that matter, one line each: what it is and why it lives there. Real paths only; each one must
exist (brownfield) or be named by the architecture (greenfield). Fix, don't enumerate.>

## Where new code goes

<The seams: where new work plugs in. One line each: "A new <thing> goes in `<path>`, following `<existing example>`".
Greenfield: designed from the architecture's recommended approach and missing pieces. Brownfield: where the code
already extends.>

## Ground rules

- <A specific choice, not a slogan: "derive types with `z.infer`", not "type safety matters". Brownfield: name the
  file that shows it. Greenfield: it is a settled decision in `architecture.md`.>

## Working principles

- <From the user's answers, only the ones that change behavior: planning posture, ask-vs-assume, scope discipline,
  what "done" means, the engineering primitives they hold.>

## Commands

- install: `<command>` · run: `<command>`
- Checks (tests, types, lint): `.claude/references/conventions.md` → `## validate`, or run `/piv-validate`.

## Load on demand

- <topic> → `.claude/references/<topic>.md`: read it when <the trigger>.

<!-- Delete any section with nothing real to put in it. Write the headings in the project's language. -->
