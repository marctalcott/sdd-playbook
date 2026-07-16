# 03 — Structure

> **Read this if** you need to set up the folders, or you're trying to work out where a given file
> belongs.

---

## Three layers

Everything in this playbook lives in one of three layers, and the layer decides who owns it.

```
┌─ LAYER 1 — Product docs ────────────────────────────────────┐
│  <product>-docs/            no code, ever                    │
│  Vision, personas, principles, glossary, decisions, catalog  │
│  Owner: Feature Manager                                      │
│  Answers: WHY, and WHAT WORDS MEAN                           │
└──────────────────────────────────────────────────────────────┘
                              │
┌─ LAYER 2 — The feature layer ───────────────────────────────┐
│  <product>-docs/features/NNN-slug/                           │
│  feature.md      — one spec for the whole feature            │
│  manifest.yaml   — the machine-readable spine                │
│  Owner: Feature Manager (spec) + Tech Lead (manifest)        │
│  Answers: WHAT this feature is, ACROSS the repos             │
└──────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┴─────────────────────┐
┌─ LAYER 3 — The repos ───────────────────────────────────────┐
│  <product>-api/          <product>-ui/                       │
│  .specify/memory/constitution.md  ← this repo's rules        │
│  .specify/specs/NNN-slug-api/     ← spec, plan, tasks        │
│  src/                             ← the code                 │
│  Owner: Tech Lead (plan) + Developers (code)                 │
│  Answers: HOW, in this repo's idiom                          │
└──────────────────────────────────────────────────────────────┘
```

Layer 2 is the one stock Spec Kit doesn't have. It is thin on purpose — **two files per feature** —
because every line you put there is a line that can drift from the repos below it.

---

## Layer 1: the product docs repo

One repo. No application code. It holds the vocabulary and the intent that every other repo consumes.
A numbered set of documents works well because the numbers imply the reading order:

```
<product>-docs/
├── 01-vision.md                 product vision
├── 02-personas-and-jobs.md      personas and jobs-to-be-done
├── 03-product-principles.md     principles (P-01, P-02, …)
├── 04-current-state.md
├── 05-target-state.md
├── 06-roadmap.md                sequencing
├── 07-feature-catalog.md   ★    the backlog — every feature request
├── 08-glossary.md          ★    THE canonical vocabulary
├── 09-decisions.md         ★    product decisions (D-001, D-002, …)
├── 10-handoff.md                how docs become specs (this process, project-specific)
├── 11-walkthroughs.md           end-to-end narratives
└── features/                    ← layer 2 lives here
    ├── _template/manifest.yaml
    └── NNN-slug/
```

The three starred files do the daily work.

**`07-feature-catalog.md` — the backlog.** Every feature request, in a fixed shape. A row must be
complete before it can be specified: user story, ≥3 Given/When/Then acceptance criteria, linked
principles, linked personas, out-of-scope, dependencies, status. Statuses run
`Idea → Ready-for-spec → Spec'd → In-progress → Shipped`.

**`08-glossary.md` — the vocabulary.** The most important file in the system. Every domain term,
every enum, every identifier, every state transition. **One name per concept, and no synonyms.**

Why this file matters more than it looks: an AI generating a spec will happily write `order` where
your glossary says `transaction`, then a developer will write an `OrderService`, and now your
codebase has two words for one thing and every future conversation carries a translation cost. It
compounds. This is the most common way spec-driven development quietly fails, and the glossary is the
only defence.

**`09-decisions.md` — the decisions.** Numbered, dated, with rationale: *money is stored as integer
cents* (D-011), *no SMS before M12* (D-015). Specs cite them. When a decision changes, you can find
every spec that assumed it.

> **Why a separate repo?** Because it makes "the docs are the source of truth" true rather than
> aspirational. Docs in a code repo become that repo's docs and quietly start describing that repo.
> Docs in their own repo can be cited with a commit SHA — `source: <product>-docs@a1b2c3d` — which
> means a spec can name the exact version of the truth it was generated from. That's what makes
> regeneration reproducible.

---

## Layer 2: the feature folder

```
<product>-docs/features/015-watch-list/
├── feature.md       ← ONE spec, for the whole feature, across all repos
└── manifest.yaml    ← the spine
```

**`feature.md`** is the cross-repo specification. Prioritised user stories with Given/When/Then, the
functional requirements, the key entities, the success criteria, the out-of-scope list, and the open
questions. It is assembled from layer 1 — the catalog row plus the *full text* of every linked
principle, persona, glossary row, and decision.

This replaces the "two sibling specs you keep in sync by hand" model, which does not work. One spec,
projected down.

**`manifest.yaml`** is the artifact polyrepo was missing.

---

## The manifest, field by field

This is the one file worth understanding completely. Everything else is documents; this is the thing
that makes the pipeline mechanical.

```yaml
feature: 015-watch-list
title: Watch List
catalog_ref: F-LST-015                # the row in 07-feature-catalog.md
source: <product>-docs@a1b2c3d        # the exact docs version this was specified from

status: draft                         # draft → spec-signed-off → in-progress
                                      #       → signed-off → uat → shipped
spec: feature.md

# ─── The customer signed the WHAT. Gate 1. ───────────────────────────
spec_signoff:
  by: null                            # a real person's name
  at: null                            # ISO timestamp
  note: null                          # what they accepted, incl. what's out of scope

# ─── Which repos participate, and where each one stands ──────────────
repos:
  api:
    path: <product>-api
    spec: .specify/specs/015-watch-list-api
    constitution: .specify/memory/constitution.md
    branch: feat/015-watch-list-api
    state: new                        # new | specified | planned | in-progress | merged
    pr: null
  ui:
    path: <product>-ui
    spec: .specify/specs/015-watch-list-ui
    constitution: .specify/memory/constitution.md
    branch: feat/015-watch-list-ui
    state: new
    pr: null

# ─── Stories, slices, and the per-story gate ─────────────────────────
phases:
  - id: P1
    title: Watching and viewing
    status: pending
    stories:
      - id: US1
        title: As a buyer, I want to watch a listing, so that I can find it again
        priority: P1
        slices:                       # THE WORK ORDER. api before ui, always.
          - { repo: api, tasks: T001-T009 }
          - { repo: ui,  tasks: T001-T006 }
        done_gate:
          status: pending             # pending → green → signed-off
          e2e: <product>-ui/e2e/015-watch-list-us1.spec.ts
          e2e_tag: "@015-us1"         # the test selector
          last_run: null
        signoff:
          by: null
          at: null
          note: null

# ─── How to stand up the WHOLE system for the gate ───────────────────
compose:
  api:
    cwd: <product>-api
    cmd: "<your API run command>"
    health: "http://localhost:PORT/health"   # poll until 200 BEFORE testing
  workers:                            # background services — no port to poll
    - { name: projections, cwd: <product>-api, cmd: "<your worker command>" }
  ui:
    cwd: <product>-ui
    cmd: "<your UI run command>"
    url: "http://localhost:3000"
  e2e:
    cwd: <product>-ui
    cmd: "<your e2e command> --grep '@015'"

# ─── Append-only history. Every stage writes one line. ───────────────
log:
  - { at: "2026-07-01T10:00:00Z", phase: specify, by: feature.specify, note: "scaffolded" }
```

### The four fields that do the heavy lifting

**`state`** — per repo, and it's a dispatcher. It tells every later stage what to do with this repo:

| `state` | Meaning | Effect |
|---|---|---|
| `new` / `specified` | Needs planning | Plan it |
| `planned` / `in-progress` | Already has plan artifacts | Reconcile against it, **don't regenerate** |
| `merged` | **Shipped. Contracts are FROZEN.** | Read-only authority. Other repos conform to it. |

Getting this wrong either clobbers shipped work or duplicates a plan that already exists.

**`slices`** — the work order, api-before-ui. This is what makes a story vertical. Everything in the
inner loop reads exactly this.

**`done_gate`** — the per-story test gate. `e2e_tag` is the thread that ties an acceptance criterion
to a test. `status` moves `pending → green` (a machine, on a real run) `→ signed-off` (a human).

**`compose`** — the reproducible "stand up everything and test it" recipe. This exists because a
read-side feature is invisible to a test unless its background workers are running, and "it worked on
my machine because I happened to have the worker up" is not a process.

### Why the log matters

Every stage appends one line. It gives you a readable history without diffing anything:

```yaml
log:
  - { at: "2026-07-01T10:00:00Z", phase: specify,  by: feature.specify,  note: "scaffolded; repos: api, ui" }
  - { at: "2026-07-02T14:30:00Z", phase: signoff,  by: feature.spec-signoff, note: "spec signed by Dana Ortiz" }
  - { at: "2026-07-03T09:15:00Z", phase: plan,     by: feature.plan,     note: "2 repos planned; 1 contract mismatch" }
  - { at: "2026-07-05T16:45:00Z", phase: verify,   by: feature.verify,   note: "US1 gate @015-us1 GREEN" }
```

Six months later, that's how you find out what happened.

> **A YAML trap that will bite you:** always **quote ISO timestamps** inside `{ }` flow-maps. The
> colons in `10:00:00` break the parse otherwise. Write `at: "2026-07-01T10:00:00Z"`, not
> `at: 2026-07-01T10:00:00Z`.

---

## Layer 3: the code repos

Each code repo gets **its own Spec Kit install** and **its own constitution**:

```
<product>-api/
├── .specify/
│   ├── memory/constitution.md         ← THIS repo's rules
│   ├── templates/                     spec-template.md, plan-template.md, tasks-template.md
│   └── specs/
│       └── 015-watch-list-api/
│           ├── spec.md         ← the WHAT (from the Feature Manager, stage 2)
│           ├── plan.md         ← the HOW (Tech Lead, stage 4)
│           ├── research.md
│           ├── data-model.md
│           ├── contracts/      ← what this repo EXPOSES
│           └── tasks.md        ← T001… (stage 5)
├── .github/
│   ├── copilot-instructions.md
│   └── prompts/                       ← Spec Kit's /speckit.* commands land here
└── src/
```

**The constitution is the repo's own law.** Test-driven development rules, naming conventions,
"never hard-code business values", "components must meet WCAG AA", "call the real API, never a
mock" — whatever this repo mandates. It is written once with `/speckit.constitution` and then it
governs every implementation in that repo.

This is the **DOWN** half of the organising rule from [01](01-why.md): the feature layer says *what*
and *in what order*; the constitution says *how*. Don't move constitution rules up — they're
different per repo, and that's correct.

### Naming conventions that pay for themselves

| Convention | Example | Why |
|---|---|---|
| Shared feature number across repos | `015-watch-list-api`, `015-watch-list-ui` | You can find every piece of feature 015 by grepping `015` |
| `-api` / `-ui` suffix | | Disambiguates the two installs at a glance |
| Branch = spec folder name | `feat/015-watch-list-api` | The branch and the spec can never be mismatched |
| Test tag `@NNN-us{n}` | `@015-us1` | **The thread from acceptance criterion to test.** The gate is literally `--grep '@015-us1'` |

That last one is the one to internalise. It's how a sentence the customer signed in stage 3 becomes a
test that gates production in stage 7.

---

## The whole thing on disk

```
<workspace>/                          ← the folder that holds everything
├── <product>.code-workspace          ← VS Code multi-root file. See doc 05.
│
├── <product>-docs/
│   ├── 01-vision.md … 11-walkthroughs.md
│   ├── .github/agents/               ← the feature.* Copilot agents live here
│   └── features/
│       ├── _template/manifest.yaml
│       └── 015-watch-list/
│           ├── feature.md
│           └── manifest.yaml
│
├── <product>-api/
│   ├── .specify/{memory,templates,specs}/
│   ├── .github/{copilot-instructions.md,prompts/}
│   └── src/
│
└── <product>-ui/
    ├── .specify/{memory,templates,specs}/
    ├── .github/{copilot-instructions.md,prompts/}
    ├── e2e/                          ← the gate tests
    └── src/
```

The repos are **siblings**. Every agent finds its bearings by walking up the tree until it sees them
all next to each other — that folder is `<workspace>`. Keep them flat.

> **Watch out if a repo is nested.** If your frontend app lives in a subfolder (`<product>-ui/App/`),
> then Spec Kit installs at `<product>-ui/App/.specify/` but **git branches are still cut at
> `<product>-ui/`**. Those two paths differ and mixing them up is a genuinely common, genuinely
> annoying bug. Write the real paths into the manifest and let it be the authority.

---

Next: [04 — Roles](04-roles.md)
