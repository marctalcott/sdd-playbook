# 07 — Feature Catalog

| Field | Value |
|---|---|
| Owner | Feature Manager |
| Last reviewed | *(date)* |
| Status | Living document |

> **The backlog.** Every feature request lives here, in a fixed shape, before it becomes a spec.
>
> This is also where UAT ideas land, where "could it also…" goes, and where the Feature Manager
> starts every morning. **The catalog is not where ideas go to die — it's where they go to be taken
> seriously.**

---

## Statuses

| Status | Meaning | Who moves it |
|---|---|---|
| `Idea` | Captured. Not yet worth the work of a full row. | Anyone |
| `Ready-for-spec` | The row is **complete** (see below). `@feature.specify` will run on it. | Feature Manager |
| `Spec'd` | A spec exists and **the customer signed it**. | `@feature.spec-signoff` |
| `In-progress` | Being built. | `@feature.implement` |
| `Shipped` | In production. | `@feature.ship` |
| `Superseded` | Replaced. Say by what. | Feature Manager |

## The completeness bar

**`@feature.specify` will refuse a row that isn't complete, and it's right to.** A row is
Ready-for-spec only with all of:

- [ ] A single user story: *As a `<persona>`, I want `<capability>`, so that `<outcome>`*
- [ ] **≥3 acceptance criteria, every one Given/When/Then**
- [ ] ≥1 linked principle (`P-XX`)
- [ ] ≥1 linked persona — one that already **exists** in `02-personas-and-jobs.md`
- [ ] An out-of-scope list (write `(none)` rather than omit it)
- [ ] A dependencies list (same)

> **Why the bar is here and not in a code review:** those Given/When/Then sentences travel
> unchanged into the spec, into the tasks, and into the automated test that gates production. This
> is the highest-leverage writing anyone does in the whole process. A vague criterion here is a
> hole in production later — and by then it's twenty times more expensive to close.
>
> If you can't fill the row, **you don't understand the request yet, and no amount of AI will fix
> that.** Go back to the person who asked.

## ID convention

`F-<DOMAIN>-<NNN>` — e.g. `F-HLD-015`. The number is unique across the whole catalog, not per
domain, so an ID never needs its prefix to be unambiguous. The domain prefix is yours to choose;
keep it to three letters and keep the list short — one per part of the product a Reader would
recognise, not one per service.

The example product ("Athenaeum", a public library) uses six:

| Prefix | Domain |
|---|---|
| `F-CAT` | Catalogue — Titles, search, availability |
| `F-LON` | Loans — borrowing, returning, renewing |
| `F-HLD` | Holds — the queue for a Title whose Copies are out |
| `F-NOT` | Notifications — how the library tells a Reader anything |
| `F-ACC` | Reader accounts — sign-in, cards, preferences |
| `F-STF` | Staff tools — what Librarians use at the desk |

---

## Template — copy this

```markdown
### F-XXX-NNN — Short name

**Status:** Idea | Ready-for-spec | Spec'd | In-progress | Shipped | Superseded
**Personas:** P-XXX
**Principles applied:** P-01, P-03
**Requested by:** {who asked, and when}
**Source:** {customer conversation | support ticket | UAT of F-XXX-NNN}

**User story:**
As a {persona}, I want {capability}, so that {outcome}.

**Acceptance criteria:**
- Given {context}, when {action}, then {observable outcome}.
- Given {context}, when {action}, then {observable outcome}.
- Given {context}, when {action}, then {observable outcome}.

**Out of scope:**
- {thing we are explicitly not building}
- {thing we are explicitly not building}

**Dependencies:** F-XXX-NNN ({why}), or (none)

**Notes:** {anything a spec author needs that doesn't fit above}
```

---

## Worked example

From the fictional library product used throughout this playbook. **Delete it once you have real
rows** — it's here so the shape and the level of precision are unambiguous.

### F-HLD-015 — Hold Queue

**Status:** Ready-for-spec
**Personas:** P-RDR (Reader)
**Principles applied:** P-01 (reader privacy), P-03 (equitable access)
**Requested by:** Dana Ortiz (Riverside Library), 2026-06-28
**Source:** customer conversation

**User story:**
As a Reader, I want to place a Hold on a Title whose Copies are all on loan, so that I get the next
one returned without having to keep checking back.

**Acceptance criteria:**
- Given a Title with no Copy available, when I place a Hold, then I join the Hold Queue and see my
  position in it.
- Given I am first in the Hold Queue, when a Copy is returned, then that Copy is moved to the Shelf
  for me and I am notified exactly once.
- Given a Copy is on the Shelf for me, when `HoldShelfWindowHours` elapses without collection, then
  my Hold expires and the Copy passes to the next Reader in the queue.
- Given a Title I already have on Loan, when I view it, then I cannot place a Hold (D-005).

**Out of scope:**
- Holding a *specific* Copy — the queue is Title-level per D-001
- Inter-library loans (a Hold never spans branches in v1)
- Email notifications — F-NOT-004 is in-app only; email is F-NOT-019
- Suspending a Hold while away → **F-HLD-021**
- Telling a Reader how long the wait will be → **F-HLD-022**

**Dependencies:** F-CAT-001 (Title catalogue — a Hold is placed from a Title page),
F-LON-002 (Borrow and return — the Hold Queue reacts to `CopyReturned`),
F-NOT-004 (Notification centre — where "notified exactly once" is delivered)

**Notes:** "exactly one" in criterion 2 is deliberate — an early prototype fired twice when two
Copies were returned in the same minute. `HoldShelfWindowHours` is referenced by name per D-002; its
default lives in glossary §6. Criterion 4 exists because D-004 was superseded by D-005 — Readers
were parking at the front of the queue by holding what they already had out.

> **Note the out-of-scope list.** Two of its four entries are real catalog rows, not vague
> deferrals. That is what "out of scope" should look like: the customer can see exactly where the
> thing they asked for went, and it is somewhere it will actually get built.

---

## Active

An `Idea` is allowed to be one line. **That's the point** — capture is cheap, and the completeness
bar only applies when you promote a row to `Ready-for-spec`. A backlog of half-filled "nearly ready"
rows is worse than a backlog of honest one-liners, because you can't tell which ones are actually
startable.

| ID | Status | One-line story | Waiting on |
|---|---|---|---|
| **F-HLD-015** | Ready-for-spec | Hold Queue — join the line for a Title whose Copies are all out. *(the worked example above)* | — |
| **F-HLD-021** | Idea | As a Reader, I want to suspend a Hold while I'm away, so that I keep my place without losing my turn. | F-HLD-015 |
| **F-HLD-022** | Idea | As a Reader, I want an estimate of how long I'll wait, so that I can decide whether to buy the book instead. | F-HLD-015 |
| **F-LON-018** | Idea | As a Reader, I want to renew a Loan I'm still reading, so that I don't make a trip to return and re-borrow. | F-HLD-015 |
| **F-CAT-013** | Idea | As a Reader, I want large-print and audio editions shown with the standard edition, so that I can pick one I can actually use. | — |
| **F-STF-009** | Idea | As a Librarian, I want to see what's on the Shelf and what expires today, so that I can clear it before the queue stalls. | F-HLD-015 |
| **F-NOT-019** | Idea | As a Reader, I want the library to email me as well as notify me in-app, so that I don't miss a Hold going Ready while I'm not in the app. | F-NOT-004 |

Four of those seven are waiting on F-HLD-015, which is what a dependency list is *for*: it says the
Hold Queue is the keystone and the rest of the domain is queued behind it. **F-LON-018 is the
sharpest case** — renewal has to refuse when another Reader holds the Title, so it cannot be
specified until `HoldState` exists. Specifying it now would mean inventing the very vocabulary
F-HLD-015 is about to define, which is exactly how two names for one concept get into a system.

**F-CAT-013 came out of a decision, not a customer.** D-001 settled that a Hold names a Title and
never a Copy — which means "I want the large-print one" isn't a Copy choice at all, and the only
honest way to give Readers that is to model editions as linked Titles. A decision that closes one
door usually opens a row.

**F-HLD-021 and F-HLD-022 came off F-HLD-015's out-of-scope list** when that row was written on
2026-06-28. Dana raised both in the intake conversation; both were scoped out; both got a row the
same afternoon. Note the order — the out-of-scope list is written at **intake**, before the spec
exists, which is why these rows predate F-HLD-015 even being specified. When Dana reaches the spec
signoff gate she'll be shown that list again and asked to confirm it, and because each entry points
at a row, "not now" reads as "scheduled" rather than "no". Nothing was dropped, and nothing was
smuggled into a feature nobody had agreed to yet.

## Shipped

Keep these. They're the record of what was agreed, and — more usefully day to day — they're where
the vocabulary and the events that everything else depends on were established.

| ID | Shipped | What it established |
|---|---|---|
| **F-CAT-001** | 2026-04-11 | Title catalogue and availability. **Where `Title` and `Copy` entered the glossary** — the distinction the whole system rests on. |
| **F-LON-002** | 2026-05-02 | Borrow and return a Copy. Emits **`CopyReturned`** — the event the Hold Queue listens for. This is the seam between the two features. |
| **F-ACC-003** | 2026-05-19 | Reader sign-in and library-card linking. Establishes who a `Reader` is. |
| **F-NOT-004** | 2026-06-06 | In-app notification centre. **In-app only, by design** — which is why "email notifications" is out of scope on every feature until F-NOT-019 ships. |

That last column is why the Shipped section earns its keep. When someone asks "why can't the Hold
Queue email me?", the answer isn't an opinion — F-NOT-004 shipped in-app only, and email is its own
row. The catalog remembers so nobody has to.
