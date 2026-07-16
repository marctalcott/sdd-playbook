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

`F-<DOMAIN>-<NNN>` — e.g. `F-LST-015`. The domain prefix is yours to choose; keep it to three
letters and keep the list short.

---

## Template — copy this

```markdown
### F-XXX-NNN — Short name

**Status:** Idea | Ready-for-spec | Spec'd | In-progress | Shipped | Superseded
**Personas:** P-XXX
**Principles applied:** P-01, P-07
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

Delete this once you have real rows. It's here so the shape is unambiguous.

### F-LST-015 — Watch List

**Status:** Ready-for-spec
**Personas:** P-BUY (Buyer)
**Principles applied:** P-02 (buyer confidence), P-07 (no dark patterns)
**Requested by:** Dana Ortiz (Acme Ops), 2026-06-28
**Source:** customer conversation

**User story:**
As a Buyer, I want a list of the Listings I'm tracking, so that I can come back to them without
searching again.

**Acceptance criteria:**
- Given I am viewing a Listing, when I tap Watch, then it appears in my Watch List.
- Given a Listing in my Watch List, when the Seller reduces the price, then I receive exactly one
  notification within 24 hours.
- Given a Listing in my Watch List, when it is Sold to another Buyer, then it is removed from my
  Watch List and I am notified once.
- Given a Listing I own, when I view it, then I cannot Watch it.

**Out of scope:**
- A public "N people watching" badge
- Sharing a Watch List with another User
- Email notifications (in-app only for v1)

**Dependencies:** F-LST-001 (Listings), F-NTF-001 (Notifications)

**Notes:** "exactly one" in criterion 2 is deliberate — an early prototype double-fired on a single
price change.

---

## Active

*(rows here)*

## Shipped

*(rows here — keep them; they're the record of what was agreed)*
