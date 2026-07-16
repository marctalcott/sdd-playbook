# The Spec-Driven Delivery Playbook

How a Product Manager, a Tech Lead, a Developer, a QA engineer, and a real customer take one
feature from "someone asked for this" to "it's in production" — using written specifications as
the thing that carries meaning between them, and GitHub Copilot as the thing that does the typing.

You need two tools: **VS Code** and **GitHub Copilot**. That's it.

---

## Who this is for

You are on a team that has heard of spec-driven development, maybe installed
[GitHub Spec Kit](https://github.com/github/spec-kit), and found that it helps a single developer
in a single repository — but doesn't obviously tell a *team* how to work. This playbook is the
missing part: the roles, the gates, and the folder structure that turn Spec Kit from a personal
productivity tool into a delivery process.

Read it in order. It is about 45 minutes end to end.

| Document | What it answers |
|---|---|
| [01 — Why this exists](docs/01-why.md) | What breaks without this layer, and why Spec Kit alone doesn't fix it |
| [02 — The pipeline](docs/02-pipeline.md) | The ten stages, the four gates, and the two loops. **The heart of the playbook.** |
| [03 — Structure](docs/03-structure.md) | The repos, the folders, and the manifest that ties them together |
| [04 — Roles](docs/04-roles.md) | A runbook per role: what you do, what you type, what "done" means for you |
| [05 — VS Code + Copilot setup](docs/05-copilot-setup.md) | How to actually build this with only VS Code and Copilot |
| [06 — Rollout](docs/06-rollout.md) | What a team that has never done this does in its first two weeks |
| [07 — Pitfalls](docs/07-pitfalls.md) | The failure modes, each with the rule that prevents it |

The [`starter-kit/`](starter-kit/) folder is a copy-paste implementation: the Copilot agent
definitions, the manifest template, and the docs skeleton. Clone it and you can run the pipeline
on day one.

---

## The idea in one page

Most teams write a ticket, and the ticket is the last time anyone writes down what the feature is
supposed to do. Everything after that — the design, the code, the tests, the release note — is a
lossy re-derivation of the ticket by a different person who wasn't in the room.

Spec-driven development inverts that. **The specification is the artifact that persists**, and code
is generated from it. That only works if the specification is (a) precise enough to generate from,
(b) agreed by the person who asked for the feature, and (c) carried intact through every downstream
step. This playbook is the machinery for (a), (b), and (c).

Concretely, one feature moves like this:

```
                          ┌──────────── outer loop: per feature ─────────────┐
                          │                                                  │
  request ──▶ Intake ──▶ Specify ──▶ [SPEC SIGNOFF] ──▶ Plan ──▶ Tasks ──▶  │
                          │                 ▲                                │
                          │            the customer                          │
                          │           signs the WHAT                         │
                          │                                                  │
                          │   ┌──── inner loop: per user story ────┐        │
                          │   │                                     │        │
                          └──▶│  Implement ──▶ [VERIFY] ──▶ [SIGN]  │──┐    │
                              │      ▲            ▲          ▲      │  │    │
                              │     Dev          QA         QA      │  │    │
                              │      └──── red gate sends back ─────┘  │    │
                              └─────────────────────────────────────┘  │    │
                                              next story ◀─────────────┘    │
                                                   │                        │
                          all stories signed ──────┘                        │
                                                   ▼                        │
                                             [UAT] ──▶ Ship ──▶ production   │
                                                │                            │
                                          real customers                     │
                                                │                            │
                          new ideas ────────────┘                            │
                                │                                            │
                                └──▶ back to Intake as new feature requests ─┘
```

Four things in square brackets are **gates** — a named person says yes or no, and the process stops
until they do. Everything else is work.

---

## The four ideas that make it work

**1. Sign the spec, then sign the build.** Two different signatures from two different people at two
different times. The customer signs the *what* before anyone writes code — that is what stops the
"this isn't what I asked for" conversation happening after four weeks of work. QA signs the *build*
after the tests are green. Neither signature substitutes for the other.

**2. The user story is the unit of delivery, not the feature.** A feature is decomposed into user
stories, and each story is carried all the way across the system — backend, frontend, test, signoff
— before the next story starts. This is the opposite of the natural instinct, which is to build all
the backend then all the frontend. The payoff: story 1 is shippable while story 3 doesn't exist yet.

**3. There is a layer above the repositories.** Spec Kit installs *into* a repo and can only see
that repo. A real feature crosses repos. So a thin coordination layer sits above them, and it owns
exactly two files per feature: a cross-repo spec, and a manifest. Everything else stays down in the
repos where it belongs.

**4. Names are law.** A `Listing` means the same thing in the product docs, in the API, in the UI,
and in the test. One glossary, and nobody is allowed to invent a synonym — not a person, and
especially not an AI. Most spec-driven development failures are, at bottom, vocabulary failures.

---

## What this is not

- **Not a Spec Kit manual.** It assumes you can run `/speckit.specify`. If you can't yet, do the
  [Spec Kit quickstart](https://github.github.com/spec-kit/quickstart.html) first — an hour, one repo,
  one toy feature — and come back.
- **Not for every change.** Copy fixes, dependency bumps, and no-behavior-change refactors bypass all
  of this. The rule for when to use it is in [01 — Why this exists](docs/01-why.md).
- **Not automatic.** Every gate is a person. The tooling makes the evidence easy to see; it does not
  make the decision.

---

## A note on the placeholders

This playbook was extracted from a working three-repo system and generalised. Wherever you see
`<product>`, substitute your own name:

| Placeholder | Meaning | Example |
|---|---|---|
| `<product>-docs` | The product docs repo — source of truth, no code | `acme-docs` |
| `<product>-api` | A backend code repo | `acme-api` |
| `<product>-ui` | A frontend code repo | `acme-web` |
| `<workspace>` | The folder holding all of the above as siblings | `~/src/acme` |
| `NNN-slug` | A feature's shared number and short name | `015-watch-list` |
| `US{n}` | A user story within a feature | `US1`, `US2` |

The pipeline does not care whether you have two repos or five. It cares that there is more than one,
and that no single one of them knows when the feature is done.
