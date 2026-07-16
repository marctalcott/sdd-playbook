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

- constrains future specs ("money is integer cents")
- would otherwise be re-argued every few months ("no SMS before M12")
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

Delete these once you have real ones.

### D-001 — Money is integer minor units

**Status:** Accepted
**Date:** 2026-01-15
**Deciders:** Tech Lead, Feature Manager

**Context:**
Amounts appear in specs, the API, the UI, and the database. Floating-point and decimal both
invite rounding drift across that many boundaries, and rounding drift in money is the kind of bug
you find in an audit rather than a test.

**Decision:**
All monetary amounts are stored, transmitted, and computed as **integers in minor units** (cents).
Never a float. Never a decimal. Formatting to a display string happens at the UI edge only.

**Rationale:**
Integers are exact. Every rounding decision becomes explicit and deliberate rather than emergent.
The cost is a small amount of ceremony at the display boundary, which is cheap and local.

**Consequences:**
- Every API contract expresses money as an integer, with the unit in the field name.
- A spec that writes an amount as `49.99` is a defect. It's `4999`.
- Currency conversion needs explicit rounding rules, decided per case.

**Affects:** every payment feature; glossary §7 (Units and types).

---

### D-002 — Business values live in configuration

**Status:** Accepted
**Date:** 2026-01-15
**Deciders:** Tech Lead

**Context:**
Thresholds, caps, fees, and time windows change for business reasons, on business timescales,
often urgently. Baked into code, each change is a deploy. Baked into a **spec**, they're worse:
the literal propagates into code, and then nobody knows which `5000` was the payment cap and which
was a coincidence.

**Decision:**
Business and tuning values are defined in configuration and referenced **by name** — in specs, in
plans, and in code. **A literal business value in a spec is a defect.**

**Rationale:**
Named values are self-documenting and greppable. Changing one is a config change, not a release.
The spec stays true across the change.

**Consequences:**
- Specs say "capped at `MaxCardAmount`", never "capped at 5000".
- Every such key is listed in glossary §6.
- Tests reference the config, not a hard-coded expectation.

**Affects:** every spec; every repo constitution.

---

## Decisions

*(append here — newest last)*
