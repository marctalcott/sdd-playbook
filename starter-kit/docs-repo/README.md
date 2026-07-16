# `<product>`-docs

**The product source of truth. No application code lives here, ever.**

This repo holds what the product *is*, what its words *mean*, and why it works the way it does. Every
code repo consumes this one. Nothing here consumes them.

It also hosts the orchestration layer — `features/` — because coordinating work across repos is a job
that can only be done from above them.

---

## Read these in this order

New to the project? About an hour.

| # | Document | What it gives you |
|---|---|---|
| 1 | [vision.md](vision.md) | Why the product exists. Read once. |
| 2 | [personas-and-jobs.md](personas-and-jobs.md) | Who it's for, and the jobs they're trying to do. |
| 3 | [product-principles.md](product-principles.md) | What we'd say no to a customer over (`P-01`, `P-02`, …). |
| 4 | [walkthroughs.md](walkthroughs.md) | End-to-end narratives. The fastest way to see how the pieces fit. |
| 5 | [glossary.md](glossary.md) | **The vocabulary.** Skim it now; you'll live in it later. |
| 6 | [decisions.md](decisions.md) | **Why the system is the way it is.** Read the ones that surprise you. |
| 7 | [feature-catalog.md](feature-catalog.md) | The backlog. What's shipped, what's next. |

**The order above is a reading order, not a work order.** When you're *setting a project up*, you
write [glossary.md](glossary.md) first and everything else after — see the playbook's rollout guide.

---

## The four kinds of file here

They behave completely differently, and it's worth knowing which is which:

| Kind | Files | How it behaves |
|---|---|---|
| **Narrative** | vision, personas, principles | Read once. Changes a few times a year. |
| **Live** | feature-catalog | Changes daily. The only one that does. |
| **Reference** | glossary, decisions | Never read cover to cover. Looked up constantly, by people *and* by agents. |
| **Examples** | walkthroughs | Read when you're lost. |
| **Orchestration** | `features/` | One folder per feature. See below. |

---

## The two files that do the most work

### `glossary.md` — the vocabulary

One name per concept. **No synonyms, ever.** If you need a word that isn't in here, that's a
conversation with the team — not a word you invent in a spec.

This is the most-loaded file in the system: the AI agents read it on nearly every run. A sloppy
entry here becomes a sloppy spec, then sloppy code, then two names for one thing in three repos.

### `decisions.md` — why the system is like this

Numbered (`D-001`, `D-002`, …), dated, and carrying **the reasoning, not just the ruling**.

When you find a rule that looks arbitrary — *why can't a reader do X? why is that value in config?*
— the answer is in here, with what forced it and what it traded away. That's what lets you argue
with a rule on the merits instead of nervously working around it.

**Append-only. You supersede; you never delete.** A superseded decision stays put, because the
reasoning that killed it is the valuable part — otherwise the next person proposes the same idea
again in good faith and nobody can say why it was rejected.

Specs cite decisions by number, so when one is superseded you can find every spec that assumed it.

---

## `features/` — the orchestration layer

One folder per feature, and **this is where numbers are load-bearing**:

```
features/
├── _template/manifest.yaml    ← copy this per feature
└── 015-hold-queue/
    ├── feature.md             ← ONE spec for the whole feature, across every repo
    └── manifest.yaml          ← the spine: repos, branches, story slices, gates
```

That `015` is a **cross-repo correlation key**. The same number appears in the API's spec folder, the
UI's spec folder, both branch names, and the `@015-us1` test tag the done-gate runs. Grep `015` and
you find every artifact belonging to this feature, in every repo.

The documents above have no numbers because there's only ever one of each — **number the things you
have many of, name the things you have one of.**

---

## Rules

- **Names are law.** A term means the same thing here, in the API, in the UI, and in the tests.
- **No implementation detail.** This repo says *what* and *why*. The code repos say *how*.
- **No business values as literals.** They live in config and are referenced by name — see the
  glossary's config section.
- **Nothing here is generated.** Every word was written by a person and agreed. That's the point.
