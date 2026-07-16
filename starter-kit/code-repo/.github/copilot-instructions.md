# <product>-api

<!--
  TEMPLATE — copy into EACH code repo's .github/ and fill in.
  Delete these comments.

  Keep this file SHORT. It is loaded into every Copilot request in this repo, and a long
  instructions file is a diluted one. Its job is to state the facts about this repo and to
  POINT AT the real authorities — not to restate them.

  Rule of thumb:
    copilot-instructions.md  → facts about this repo, and pointers
    *.instructions.md        → rules scoped to a folder (via applyTo)
    .specify/memory/constitution.md → THE LAW for how code gets written here

  Do NOT copy the constitution into this file. Two copies of a rule become two different
  rules within a month. Point at it.
-->

*(One or two sentences: what this service is and what it's built with.)*

## Non-negotiable

- **Domain vocabulary comes from `../<product>-docs/08-glossary.md`.** One name per concept.
  **Never invent a synonym.** If a term you need isn't there, that's an Open Question for the team,
  not a word you coin.
- **Business values come from configuration, by name.** Never a literal. See `D-002` and glossary §6.
- **The full rules are in `.specify/memory/constitution.md`. Read it before writing code.**

*(Add this repo's own two or three hard rules here — the ones that are true for every file. Keep
the list short enough that it's actually read.)*

## Structure

| Path | What's in it |
|---|---|
| `src/` | *(fill in)* |
| `.specify/memory/constitution.md` | This repo's law |
| `.specify/specs/NNN-slug-api/` | The spec, plan, and tasks for feature NNN |
| `e2e/` | *(UI repos)* End-to-end tests, tagged `@NNN-us{n}` |

## How work arrives here

This repo is one side of a cross-repo feature. The coordinating layer lives in
`../<product>-docs/features/NNN-slug/`:

- **`feature.md`** — the spec for the whole feature, across every repo.
- **`manifest.yaml`** — which branch, which task slices deliver which story, and which test gates it.

Work is picked up **one user story at a time**, and the manifest says exactly which task IDs in
this repo deliver it. Don't build the next story while you're in here — see the playbook.

## Testing

- Unit and component tests: this repo's business, run at implement time.
- **The cross-repo end-to-end gate is not run from here.** It runs against the *whole* system —
  every service in the manifest's `compose` recipe — and it is QA's gate, not the developer's.
  A green build here means "ready to be tested", not "done".
