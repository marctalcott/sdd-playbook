# 07 — Pitfalls

> **Read this if** something feels off, or before you run your first real feature. Every rule here was
> paid for by someone. None of them are theoretical.

---

## The five that will actually get you

Skip the rest if you must. Not these.

### Synonym drift

**The single most common cause of failure in spec-driven development.**

The spec says `order`. The glossary says `transaction`. Nobody notices, because both words are
obviously fine. A developer writes `OrderService`. Six weeks later, half the codebase says order,
half says transaction, and every conversation carries a translation. The AI happily generates both,
because it learned from a world where both exist.

**Why it's so dangerous:** it never announces itself. Nothing breaks. It just gets 5% more expensive
to talk about your own system, every week, forever.

**The rule:** *Names are law.* One glossary. One name per concept. If a spec needs a word that isn't
in `08-glossary.md`, that is a **signal to add the term deliberately, with the team** — not a licence
to coin a synonym. Reject any generated artifact that invents one; put the glossary rows in the prompt
and regenerate.

### The false green

QA runs the gate with four of five services up. The fifth "isn't related to this feature." It passes.
Ship it.

Except the feature reads from a projected view, the projection worker was the one that was down, and
the test asserted against stale data that happened to look right.

**Why it's the worst thing in this playbook:** it doesn't just miss one bug. It teaches everyone that
green means "probably." Every green after it is worth less. You cannot un-ring this bell — once a team
has seen a green gate lie, they start manually re-checking, and the gate has become theatre with extra
steps.

**The rule:** *the whole system, or it's a lie.* Every service, every worker, health checks polled
before a single assertion. If a service genuinely isn't needed, remove it from the `compose` recipe
**deliberately, in the manifest**, where everyone can see the decision. And: **zero tests matched is a
BLOCK, not a pass.** Zero failures is not success.

### Absorbing UAT ideas into the current feature

A customer tries it and says "could it also sort by date?" It's small. It's obviously good. The
developer's already in the file. Everyone agrees. In it goes.

**What just happened:** the shipped feature is no longer the feature anyone signed. The estimate is
now fiction. And — the expensive part — the customer just learned that the signoff gate was theatre.
Next time they won't read the spec carefully, because they know they can add things later. You have
destroyed the thing that made stage 3 valuable, in exchange for a sort order.

**The rule:** *bugs come back; ideas go to the catalog.* A good idea loses nothing by being a catalog
row — it gets specified, signed, and tested, which is more than it'd get as a quiet addition. **The
catalog is not where ideas go to die. It's where they go to be taken seriously.** Say that out loud
the first few times; it's the difference between the rule reading as process and reading as respect.

### Building the next story while you're in there

Story 2's endpoint is twenty minutes and you have the whole feature in your head.

**What just happened:** story 1 is no longer independently shippable, because its branch now carries
story 2's half-finished work. The gate for story 1 no longer means "this could ship." You are doing
horizontal slicing while believing you're doing vertical slicing — which is worse than doing it
honestly, because you've lost the ability to tell the difference.

**The rule:** *one story, end to end, then stop.* Take the twenty minutes back next story.

### Hand-editing a generated spec

The generated spec has a small mistake. You fix it in `spec.md`. Done in ten seconds.

**What just happened:** the next regeneration silently reverts it. And the property that makes the
whole approach work — *the spec is reproducible from the docs* — is now false, in a way that nothing
will detect.

**The rule:** *fix the input, regenerate.* If the fix belongs in the catalog row, fix the row. If it
belongs in the glossary, fix the glossary. The generated artifact is an output; treat it like build
output.

---

## By stage

### Intake and Specify

| Pitfall | How it shows up | Fix |
|---|---|---|
| **Thin acceptance criteria** | "It should work well" | Reject. Given/When/Then or it isn't a criterion. These become the tests. |
| **Invented persona** | Spec references a persona not in `02-personas-and-jobs.md` | Reject. Personas aren't created in spec land. |
| **Principle skipped** | Spec quietly violates a linked principle | Put the principle's **full text** in the prompt, regenerate. |
| **Out-of-scope quietly added** | The spec has a feature you didn't ask for | Reject. Models are enthusiastic. Move it to the catalog as its own row. |
| **Hard-coded business values** | Spec says `5000` for a cap | Reject. Reference config by name. A literal in a spec becomes a literal in production. |
| **Guessed instead of flagged** | An ambiguity got resolved by the model, silently | Anything unresolved is an open question. Run `/speckit.clarify`. |
| **No test plan** | Section is one line | Reject. If QA can't test it from the spec, the spec isn't done. |

### Spec signoff

| Pitfall | How it shows up | Fix |
|---|---|---|
| **Signoff by silence** | Emailed the spec, nobody replied, called it approved | Not a signoff. A signoff is a person saying yes. |
| **Signing raw markdown** | Customer skims a document they don't understand | Read the criteria out loud. Given/When/Then is almost English — use that. |
| **Out-of-scope not reviewed** | Customer signs the what, never sees the what-not | Review the out-of-scope list explicitly. It's the half that causes arguments. |
| **Nobody available to sign** | "They're busy, let's proceed" | That's information, not an obstacle. Escalate. Don't proceed on a shrug. |

### Plan and Tasks

| Pitfall | How it shows up | Fix |
|---|---|---|
| **Contract drift** | API returns `listingId`, UI expects `id` | Caught at plan time it's free. Caught in the gate it's a day. **The merged side always wins.** |
| **Editing shipped code to match a plan** | "The UI plan is nicer, let's change the API" | No. A merged contract is a fixed input. Change the plan. |
| **Rewriting `spec.md` at plan time** | The what changed quietly, after the customer signed | Flag a gap as an open question. Never edit the signed spec. |
| **Orphan tasks** | Slices + setup + excluded ≠ total | Every task is mapped, in setup, or explicitly excluded. **Silent truncation reads as "fully sliced" when it isn't.** |
| **Excluded payload absorbed** | An out-of-scope story's tasks end up in an in-scope slice | The signature failure of this layer. Check it every time. |
| **Regenerating an existing `tasks.md`** | Someone's hand-ordered list gets rewritten | An existing task list is shipped intent. **Map it, don't regenerate it.** |

### Implement

| Pitfall | How it shows up | Fix |
|---|---|---|
| **Untrimmed shared task** | A shared routing task adds an in-scope AND an out-of-scope screen | Build only the in-scope half. Excluded work **hides inside shared tasks**. |
| **Invented contract** | UI calls an endpoint that doesn't exist | Build against the API as it shipped. A gap is a question, not a guess. |
| **Literals instead of config** | `if (amount > 5000)` | Config, by name. Always. |
| **Pushing at implement time** | Branch is on the remote before QA saw it | Implement ends at a local commit. |

### Verify and Signoff

| Pitfall | How it shows up | Fix |
|---|---|---|
| **Green with a service down** | See [the false green](#the-false-green) | The worst outcome available. Whole system or nothing. |
| **Zero tests matched, called green** | `--grep` typo, 0 tests ran, 0 failed | A run that matches zero specs is a **BLOCK**. |
| **False red from lag** | Test asserts before the worker caught up | Wait for the data. **Don't re-tag the test.** |
| **Flake accepted** | "It passes if you run it again" | A flake is not green. It's telling you something. |
| **Verify repairs the code** | The verifier fixes the bug it found | Then you have no gate. Red goes back to the developer. |
| **Green treated as signed-off** | Nobody ever judged it | Green is a **fact**; signed-off is a **judgement**. Two stages, two claims. |
| **Fabricated approval** | `signoff.by` is "the team", or an AI | A real name or no signoff. If you can't attribute it, ask. |

### UAT and Ship

| Pitfall | How it shows up | Fix |
|---|---|---|
| **UAT as a demo** | You drive, they watch, everyone's impressed | Give them the thing and a real task. Then be quiet. |
| **Findings not triaged** | One list of "UAT feedback" | Two buckets. Bug or new request. The sort **is** the work. |
| **Bug fixed, test not fixed** | It got past a green gate; the fix doesn't cover the hole | If a bug survived the gate, the gate had a hole. **Fix the test too.** |
| **Shipping UI before API** | Frontend calls an endpoint that isn't deployed | API merges first. The UI consumes the API. |

---

## Manifest hygiene

The manifest is only useful if it's true. These are the ways it stops being true.

| Pitfall | How it shows up | Fix |
|---|---|---|
| **A status lie** | `done_gate: green` with no `last_run`; `signed-off` with no signer | **The highest-severity finding the audit can produce.** It erodes the manifest's only purpose. |
| **Unquoted timestamps** | YAML won't parse | Quote ISO timestamps in `{ }` flow-maps. The colons in `10:00:00` break it. |
| **Wrong `state`** | A `merged` repo marked `planned`, and gets replanned | `state` is the dispatcher. Wrong `state` either clobbers shipped work or duplicates a plan. |
| **The branch-base trap** | The spec exists only on the feature branch, not `main` | Check what's actually on the branch you record. This one is subtle and it bites. |
| **Nested-repo path confusion** | Spec Kit is at `ui/App/.specify/` but branches cut at `ui/` | Different paths. Write the real ones in the manifest and let it be the authority. |
| **A thick feature layer** | The manifest now duplicates the spec, the contract, and a summary | Two files per feature. Duplicates diverge within a month. |
| **Stale `source` SHA** | Spec generated from docs that have since changed | Compare `source:` against the docs' HEAD. If the glossary moved, the spec may be stale. |

---

## The meta-pitfall: trusting it because it's been fine

The agents are **prompts, not programs**. They'll be right for a month and then confidently do
something wrong. This is why:

- Every agent states its **write scope** and an explicit **Forbidden** list in its body.
- Every gate is a **human**.
- The audit checks for **status lies** specifically.

All three of those will look like paranoia after a few clean features. Every one of them is there
because the alternative was discovered the hard way. The moment you remove a safeguard because it's
never fired is the moment you find out what it was catching.

---

## The rules, on one card

Print this.

> **Names are law.** One glossary, no synonyms.
> **Sign the spec, then sign the build.** Two signatures, two people, two moments.
> **The whole system, or it's a lie.** Every service, every worker.
> **Green is a fact; signed-off is a judgement.** Never collapse them.
> **One story, end to end, then stop.**
> **The merged contract wins.** Always.
> **Never lose a task.** Slices + setup + excluded = total.
> **Excluded stays excluded.**
> **Config, not literals.**
> **Fix the input, regenerate.** Never hand-edit an output.
> **Bugs come back; ideas go to the catalog.**
> **Verify observes; it never repairs.**

---

Back to the [README](../README.md) · or the [starter kit](../starter-kit/)
