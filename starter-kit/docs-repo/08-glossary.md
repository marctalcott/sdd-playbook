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

---

## 1. Core entities

| Term | Definition | Avoid |
|---|---|---|
| **Listing** | *(example — replace)* An item offered for sale by a Seller. | Item, Product, Ad, Post |
| **User** | *(example — replace)* Any authenticated account holder. | Member, Customer, Account |
| | | |

## 2. Actors and personas

Personas live in full in [02-personas-and-jobs.md](02-personas-and-jobs.md). This table is the
naming authority only.

| Term | Definition | Avoid |
|---|---|---|
| **Buyer** | *(example — replace)* A User in the act of purchasing. | Shopper, Customer |
| **Seller** | *(example — replace)* A User offering a Listing. | Vendor, Merchant |
| | | |

## 3. Actions

Name actions the way a user would say them, and keep the verb and its past-tense event aligned.

| Term | Definition | Event name | Avoid |
|---|---|---|---|
| **Watch** | *(example)* A User's deliberate flag marking a Listing to track. | `ListingWatched` | Favorite, Save, Bookmark, Follow |
| | | | |

## 4. States and enums

**Every enum value is a term.** These are the ones that bite hardest when they drift, because they
end up in a database and a URL.

### `<EnumName>` — *(example)*

| Value | Meaning | Valid transitions to |
|---|---|---|
| `Draft` | Created, not visible to Buyers | `Published`, `Removed` |
| `Published` | Visible and purchasable | `Reserved`, `Unpublished`, `Expired` |
| | | |

> **Rule:** a spec that introduces a state transition not in this table is a defect. Either add the
> transition here — with a decision explaining it — or rework the spec.

## 5. Identifiers

| Term | Format | Example | Notes |
|---|---|---|---|
| | | | |

## 6. Configuration values

**Business values live here by NAME, and in config by value. Never as a literal in a spec or in
code.**

This section is what lets a spec say "capped at `MaxCardAmount`" instead of "capped at 5000". The
first survives a pricing change; the second becomes a magic number nobody dares touch.

| Config key | What it controls | Default | Set in |
|---|---|---|---|
| `MaxCardAmount` | *(example)* Largest card payment accepted | `500000` (cents) | `appsettings` |
| | | | |

## 7. Units and types

The boring section that prevents the expensive bugs.

| Concept | Type | Rule |
|---|---|---|
| **Money** | *(example)* integer, minor units (cents) | Never a float or decimal. See D-011. |
| **Timestamps** | ISO 8601, UTC | Never local time. |
| | | |

---

## Rejected synonyms

The graveyard. When someone asks "why isn't it called X?", this is the answer, and having it
written down saves the argument being had twice.

| Rejected | Use instead | Why |
|---|---|---|
| *(example)* Favorite | Watch | Two names for one gesture; Watch reads as intent, Favorite as sentiment. |
| | | |
