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

`F-<DOMAIN>-<NNN>` — e.g. `F-HLD-015`. The domain prefix is yours to choose; keep it to three
letters and keep the list short.

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
- Given a Title I already have on Loan, when I view it, then I cannot place a Hold.

**Out of scope:**
- Holding a *specific* Copy — the queue is Title-level per D-001
- Inter-library loans (a Hold never spans branches in v1)
- Email notifications (in-app only for v1)
- Suspending a Hold while on holiday

**Dependencies:** F-CAT-001 (Catalogue), F-NOT-004 (Notifications)

**Notes:** "exactly one" in criterion 2 is deliberate — an early prototype fired twice when two
Copies were returned in the same minute. `HoldShelfWindowHours` is referenced by name per D-002; its
default lives in glossary §6.

---

## Active

*(rows here)*

## Shipped

*(rows here — keep them; they're the record of what was agreed)*
