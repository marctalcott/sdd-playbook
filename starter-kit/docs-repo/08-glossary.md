# 08 — Glossary

| Field | Value |
|---|---|
| Owner | *(name)* |
| Last reviewed | *(date)* |
| Status | Living document |

> **This is the most important file in the system.**
>
> It is the canonical vocabulary for the entire product — the docs, every code repo, every spec,
> every test, every conversation. **One name per concept. No synonyms. Ever.**
>
> Every downstream artifact inherits this vocabulary. A term you get wrong here becomes a class
> name, then a table name, then an API field, then a test fixture, and renaming it later costs a
> week. Write this file *before* you write your first spec, and write it as a team.
>
> **The rule the agents enforce:** if a spec needs a term that isn't in this file, that is a signal
> to add the term deliberately — not a licence to coin a synonym. Missing term → Open Question →
> a conversation. Never an invention.

---

## How to use this file

- **Adding a term** is a deliberate act. Propose it, agree it, add it. It is not a side effect of
  writing a spec.
- **Renaming a term** means every spec that used it is now stale. Add a decision in
  [09-decisions.md](09-decisions.md) recording why, and list the affected specs in the PR.
- **The `Avoid` column is load-bearing.** Naming the synonyms you rejected is how you stop them
  coming back. Every one of them is a word someone reasonable will reach for.

> **The examples below are from a fictional library product ("Athenaeum"), used throughout this
> playbook. Delete them and write your own.** They're here so the shape and the level of precision
> are unambiguous.

---

## 1. Core entities

| Term | Definition | Avoid |
|---|---|---|
| **Title** | A work the library holds, independent of how many physical copies exist. | Book, Item, Record |
| **Copy** | One physical instance of a Title, individually barcoded. | Item, Volume, Book |
| **Hold** | A Reader's claim on the next available Copy of a Title. | Reservation, Request, Booking |
| **Loan** | A Copy currently checked out to a Reader. | Checkout, Issue, Rental |
| **Shelf** | Where a Copy waits after being set aside for a Reader with a Ready Hold. | Holdshelf, Pickup area |

## 2. Actors and personas

Personas live in full in [02-personas-and-jobs.md](02-personas-and-jobs.md). This table is the
naming authority only.

| Term | Definition | Avoid |
|---|---|---|
| **Reader** | A library member entitled to borrow. | Patron, Member, Customer, Borrower, User |
| **Librarian** | Staff who manage the catalogue, the Shelf, and Loans. | Admin, Staff, Clerk |

## 3. Actions

Name actions the way a user would say them, and keep the verb aligned with its past-tense event.

| Term | Definition | Event name | Avoid |
|---|---|---|---|
| **Place a Hold** | A Reader joins the Hold Queue for a Title. | `HoldPlaced` | Reserve, Request, Queue, Book |
| **Collect** | A Reader takes a Ready Copy from the Shelf. | `HoldCollected` | Pick up, Claim, Retrieve |
| **Return** | A Reader gives a Copy back, ending a Loan. | `CopyReturned` | Check in, Give back |

## 4. States and enums

**Every enum value is a term.** These bite hardest when they drift, because they end up in a
database and a URL.

### `HoldState`

| Value | Meaning | Valid transitions to |
|---|---|---|
| `Queued` | Waiting in the Hold Queue. | `Ready`, `Cancelled` |
| `Ready` | A Copy is on the Shelf for this Reader. | `Collected`, `Expired`, `Cancelled` |
| `Collected` | The Reader took the Copy. Terminal. | — |
| `Expired` | The shelf window elapsed uncollected. Terminal. | — |
| `Cancelled` | The Reader withdrew the Hold. Terminal. | — |

> **Rule:** a spec that introduces a state transition not in this table is a defect. Either add the
> transition here — with a decision explaining it — or rework the spec.

## 5. Identifiers

| Term | Format | Example | Notes |
|---|---|---|---|
| **TitleId** | ISBN-13 where one exists, else an internal `T-` id | `9780441013593` | Not the Copy barcode. |
| **CopyBarcode** | 8 digits, physically on the Copy | `30412887` | Unique across branches. |

## 6. Configuration values

**Policy values live here by NAME, and in config by value. Never as a literal in a spec or in
code.**

This section is what lets a spec say "expires after `HoldShelfWindowHours`" instead of "expires
after 72 hours". The first survives a policy change; the second becomes a magic number nobody dares
touch — and nobody can later tell which `72` was the shelf window and which was a coincidence.

| Config key | What it controls | Default | Set in |
|---|---|---|---|
| `HoldShelfWindowHours` | How long a Ready Copy waits on the Shelf before the Hold expires | `72` | `config/holds` |
| `MaxHoldsPerReader` | Cap on a Reader's simultaneous active Holds | `20` | `config/holds` |
| `LoanPeriodDays` | Standard loan length | `21` | `config/loans` |

## 7. Units and types

The boring section that prevents the expensive bugs.

| Concept | Type | Rule |
|---|---|---|
| **Timestamps** | ISO 8601, UTC instants | Never library-local time. Local time is a display concern. See D-003. |
| **Durations** | Whole hours (integer) | Never a fractional hour. |
| **Queue position** | 1-based integer | Position 1 is next to be served. Never 0-based in a contract. |

---

## Rejected synonyms

The graveyard. When someone asks "why isn't it called X?", this is the answer — and having it
written down saves the argument being had twice.

| Rejected | Use instead | Why |
|---|---|---|
| Reservation | Hold | "Reservation" implies a specific Copy. The queue is Title-level — see D-001. |
| Patron | Reader | Reader says what they do. Patron is institutional jargon our members don't use about themselves. |
| Item | Title *or* Copy | "Item" collapses the two most important entities in the system. Always say which one you mean. |
