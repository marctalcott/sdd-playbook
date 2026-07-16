# 09 — Decisions

| Field | Value |
|---|---|
| Owner | Tech Lead + Feature Manager |
| Last reviewed | *(date)* |
| Status | Append-only |

> **Product-level decisions, numbered, dated, with the reasoning.** Specs cite them. Agents check
> them. When one changes, this file tells you what else is now stale.
>
> **Append-only.** You never delete a decision — you supersede it. The record of what you used to
> think, and why you changed your mind, is the value. A decisions file you can rewrite is a
> decisions file nobody can trust.

---

## Why this file exists

Six months from now, someone will look at a line of code and ask "why on earth is it done that
way?" There are three possible answers:

1. Nobody knows. *(You now have code nobody dares change.)*
2. Someone remembers. *(Until they leave.)*
3. **`D-001`.** *(You can read the reasoning and decide whether it still holds.)*

This file is how you get the third one.

There's a second, sharper use. When a decision is superseded, every spec that cited it may now be
wrong. `@feature.analyze` checks exactly this — it flags a feature citing a decision that has since
been superseded, so a story built on an obsolete premise gets caught before it ships rather than
after.

## Statuses

| Status | Meaning |
|---|---|
| `Accepted` | In force. Specs must honour it. |
| `Superseded` | Replaced. **Say by which D-XXX.** Never delete the original. |
| `Proposed` | Under discussion. Not yet binding. Don't cite it as though it is. |

## When to write one

Write a decision when a choice:

- constrains future specs
- would otherwise be re-argued every few months
- is non-obvious enough that a future reader would ask why
- was expensive to arrive at

**Don't** write one for a choice with an obvious answer, or one that lives naturally in code. If
it's an architectural decision internal to one repo, an ADR in that repo is the better home. This
file is for decisions that cross the seam or constrain the product.

---

## Template

```markdown
### D-XXX — Short title

**Status:** Accepted | Superseded by D-YYY | Proposed
**Date:** YYYY-MM-DD
**Deciders:** {names}

**Context:**
{What forced a decision. What we knew at the time. What we didn't.}

**Decision:**
{What we decided, in one or two sentences. Specific enough to check a spec against.}

**Rationale:**
{Why. Including what we traded away.}

**Consequences:**
{What this makes easy. What it makes hard. What it forecloses.}

**Affects:** {specs, features, or glossary terms that must honour this}
```

---

## Worked examples

> **These are from a fictional library product ("Athenaeum"), used as the running example
> throughout this playbook. Delete them and write your own.**

### D-001 — A Hold is placed on a Title, never a Copy

**Status:** Accepted
**Date:** 2026-02-03
**Deciders:** Tech Lead, Feature Manager

**Context:**
Readers say "can you reserve that copy for me", so the first prototype let a Hold name a specific
Copy. Two problems surfaced within a fortnight. A named Copy could be damaged, lost, or mis-shelved,
and the Hold then pointed at nothing — with no rule for what should happen next. And Readers holding
the same Title could be served out of order depending on which Copy came back, which nobody could
explain to the person who had waited longer.

**Decision:**
A **Hold names a Title**. The system assigns whichever Copy is returned first. A Reader cannot
choose a Copy.

**Rationale:**
The queue stays strictly first-come-first-served, which is the property Readers actually care about
and the one we can defend at the desk. Copy lifecycle events can never invalidate a Hold. We give up
the ability to request a particular physical Copy — genuinely wanted by a handful of Readers, and not
worth an unexplainable queue.

**Consequences:**
- Hold Queue position is meaningful and can be shown to the Reader.
- "I want the large-print edition" is not a Copy choice — large-print is a **separate Title**.
- **A spec that gives a Hold a `CopyId` is a defect.** A `Copy` is assigned at `Ready`, not at
  `Queued`.
- Damaged and lost Copies need no Hold-reassignment logic at all.

**Affects:** every hold feature (F-HLD-*); glossary §1 (Hold, Copy, Title), §4 (`HoldState`).

---

### D-002 — Policy values live in configuration

**Status:** Accepted
**Date:** 2026-02-03
**Deciders:** Tech Lead

**Context:**
Loan periods, shelf windows, and hold caps change for library-policy reasons, on policy timescales,
often at short notice — term time, holidays, a branch closure. Baked into code, each change is a
deploy. Baked into a **spec**, it's worse: the literal propagates into the code, and six months
later nobody can tell which `72` was the shelf window and which was a coincidence.

**Decision:**
Policy and tuning values are defined in configuration and referenced **by name** — in specs, in
plans, and in code. **A literal policy value in a spec is a defect.**

**Rationale:**
Named values are self-documenting and greppable. Changing one becomes a config change rather than a
release. Crucially, the spec stays *true* across the change — which is the whole premise of
generating from specs.

**Consequences:**
- Specs say "expires after `HoldShelfWindowHours`", never "expires after 72 hours".
- Every such key is listed in glossary §6, with its default.
- Tests reference the config, not a hard-coded expectation.

**Affects:** every spec; every repo constitution.

---

### D-003 — Timestamps are UTC instants

**Status:** Accepted
**Date:** 2026-02-10
**Deciders:** Tech Lead

**Context:**
Branches sit in one timezone today, but the shelf window is measured in hours and spans overnight
and daylight-saving boundaries. A Hold that expires an hour early because the clocks moved is a
Reader losing a book they queued three weeks for.

**Decision:**
All timestamps are stored, transmitted, and computed as **UTC ISO-8601 instants**. Converting to
library-local time is a display concern, done at the UI edge only.

**Rationale:**
Instants are unambiguous and arithmetic on them is safe across DST. The cost is formatting ceremony
at the boundary, which is cheap and local.

**Consequences:**
- Every API contract expresses a time as a UTC instant.
- "Expires at end of day" is not expressible without a stated timezone — say which.
- Durations are whole hours, never "until tomorrow".

**Affects:** glossary §7 (Units and types); every feature with a deadline.

---

## Decisions

*(append here — newest last)*
