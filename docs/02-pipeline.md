# 02 — The pipeline

> **Read this if** you want to know what actually happens, in order, and who is holding the pen at
> each moment. This is the heart of the playbook.

---

## The shape: two loops

A feature moves through an **outer loop**. Inside it, each user story moves through an **inner loop**.

The outer loop runs **once per feature**: agree what it is, agree it's right, work out how to build
it, cut it into stories. Then, at the end, prove it on real users and ship it.

The inner loop runs **once per user story**: build it, test it, accept it. Then the next story
starts. This is where the "always shippable" property comes from — after the inner loop closes on
story 1, story 1 works, end to end, on its own.

```
OUTER (per feature)          1. Intake          Feature Manager
                             2. Specify         Feature Manager
                             3. SPEC SIGNOFF ◀── GATE — the customer
                             4. Plan            Tech Lead
                             5. Tasks           Tech Lead
                             ├─ 5a. Analyze     Tech Lead (optional, recommended)
                             │
INNER (per user story) ──────┤  6. Implement    Developer
                             │  7. VERIFY   ◀── GATE — QA
                             │  8. SIGNOFF  ◀── GATE — QA
                             │  └─ repeat for the next story
                             │
OUTER resumes                9. UAT         ◀── GATE — real customers
                            10. Ship            Developer
                                 └─ new ideas from UAT ──▶ back to 1
```

Ten stages. Four gates. Below, each one in detail.

---

# The outer loop, part one: deciding what to build

## 1. Intake — Feature Manager

**A request arrives.** From a customer, a support ticket, a sales call, a UAT session on a previous
feature. It is prose, and it is probably vague.

Your job is not to write a spec yet. It is to turn the request into a **row in the feature catalog**
(`<product>-docs/07-feature-catalog.md`) that is complete enough to specify from. A row is complete
when it has:

- A single user story: *As a `<persona>`, I want `<capability>`, so that `<outcome>`*
- **At least three acceptance criteria, every one of them Given/When/Then**
- At least one linked product principle
- At least one linked persona — one that already exists in `02-personas-and-jobs.md`
- An explicit out-of-scope list (write "(none)" rather than omit it)
- An explicit dependencies list (same)
- Status: **Ready-for-spec**

The Given/When/Then discipline is not a formatting preference. Those acceptance criteria are going to
travel, unchanged, all the way down to the automated tests QA runs in stage 7. Write them badly here
and QA inherits an untestable feature four weeks later. This is the single highest-leverage writing
in the whole pipeline.

If you can't fill the row, **you are not ready and no amount of AI will fix it**. Go back to the
person who asked.

> **Vocabulary check.** Every noun in your row must already exist in `08-glossary.md`. If the feature
> needs a word the product doesn't have yet, that's a real signal: stop and add the term to the
> glossary as a deliberate act, with the team. Do not let a synonym in. See
> [07 — Pitfalls](07-pitfalls.md#synonym-drift).

**Done when:** the row is Ready-for-spec and the glossary covers its vocabulary.

---

## 2. Specify — Feature Manager

**You now write the WHAT.** Not the how. No class names, no file paths, no database tables, no
mention of React or Postgres. If a developer reading your spec learns *how you'd build it*, you have
written the wrong document.

You run one command:

```
@feature.specify F-HLD-015
```

It reads your catalog row and, crucially, **assembles the context around it**: the full text of every
principle you linked, every persona you referenced, the relevant glossary rows, and any product
decisions that constrain the feature. That assembly is why the output is good — a spec generated from
a bare ticket is a bare ticket with more words.

It produces:

| Artifact | Where | What it is |
|---|---|---|
| `feature.md` | `<product>-docs/features/NNN-slug/` | **One spec for the whole feature**, across every repo |
| `manifest.yaml` | same folder | The machine-readable spine — see [03](03-structure.md) |
| `spec.md` | in each code repo, on a new branch | The repo-local projection of `feature.md` |

Note the shape: **one** feature spec, **projected** into each repo. Not two specs that you keep in
sync by hand — that is a job nobody has ever done successfully.

**Then you review it, and this is real work.** Check, in order:

1. **Names.** Every term matches the glossary. If the AI invented a synonym, that's a defect.
2. **Acceptance criteria survived.** Every Given/When/Then from your row is in the spec.
3. **Out-of-scope survived.** Nothing got quietly added. AI models are enthusiastic; they will
   helpfully spec a feature you didn't ask for.
4. **No hard-coded business values.** Thresholds, caps, fees, and time windows are referenced *by
   name*, never written as literals. `5000` in a spec becomes `5000` in code, and then it is in
   production and nobody knows why.
5. **Open questions are marked, not guessed.** Anything unresolved should be flagged, not invented.

If something is wrong, **fix the input and regenerate — don't hand-edit the spec.** A hand-edit is
lost the next time anyone regenerates, and it breaks the property that makes this work: that the spec
is reproducible from the docs.

Run `/speckit.clarify` to work through the ambiguities the model flagged. It will ask you questions.
Answer them; that's the point.

**Done when:** the spec has no unresolved ambiguities and you'd be comfortable showing it to the
customer. Which is convenient, because that's next.

---

## 3. SPEC SIGNOFF — the customer 🚦 **GATE**

**This is the most valuable gate in the pipeline and the one teams skip first.**

You take the spec to the person who asked for the feature and they say yes or no *to the what*,
before anyone writes a line of code.

```
@feature.spec-signoff 015-hold-queue --by "Dana Ortiz (Athenaeum Ops)" --note "..."
```

Practically: don't email them raw markdown. Walk them through the user stories and read the
acceptance criteria out loud. "Given you're first in the queue, when a copy comes back, then it's
put aside for you and you're told once — is that right?" Given/When/Then is unusually good at this,
because it's almost English and it's specific enough to disagree with. A customer cannot
meaningfully review "the system shall support hold notifications." They can absolutely tell you that
three days on the shelf is too long, because they're the one who has to find space for it.

Record what they accept, **including what they accept being out of scope**. That list is the thing
you point at in six weeks.

**The rule:** no signoff, no plan. If the customer won't engage, that is information — either the
feature doesn't matter enough to them, or you have the wrong customer in the room. Escalate; don't
proceed on a shrug.

**Done when:** the manifest records `spec_signoff: {by, at, note}` and the feature status is
`spec-signed-off`.

---

# The outer loop, part two: deciding how to build it

## 4. Plan — Tech Lead

**The spec is now frozen as intent.** You do not get to relitigate the what. You add the how.

```
@feature.plan 015-hold-queue
```

Two things happen, and the second one is the reason this stage exists at the feature level.

**First, per-repo planning.** For each repo, a sub-agent reads that repo's `spec.md` and that repo's
**constitution**, and produces the technical artifacts: `plan.md`, `research.md`, `data-model.md`,
and `contracts/`. It works inside that repo's rules, because that's where those rules live.

**Second — and this is the part no per-repo tool can do — contract reconciliation.** You compare what
the API says it *exposes* against what the UI says it *consumes*: endpoint paths, field names, enum
values, status codes, error shapes. Every divergence is drift, and drift found here is free. The same
drift found in stage 7 costs a day, and found in production costs a customer.

> **The authority rule:** if one repo has already shipped and merged its half, **the merged contract
> wins.** It is a fixed input. You do not edit shipped code to match a plan; you change the plan. This
> comes up constantly and the answer is always the same.

This is also where you apply **technical strategy** — the thing the Feature Manager's spec is
deliberately silent about. Does this fit the architecture? Does it need a new service, and if so is
that a decision that should be written down as an ADR before it's built? Is there an existing pattern
this should follow? If the answer changes what gets built, record it as a decision, not a code
comment.

**What you may not do:** rewrite `spec.md`. If planning reveals a genuine functional gap, mark it and
raise it — the spec belongs to the Feature Manager and the customer signed it. Changing what the
customer agreed to, quietly, at plan time, is exactly the failure the signoff gate exists to prevent.

**Done when:** every repo has plan artifacts and the cross-repo contracts agree.

---

## 5. Tasks — Tech Lead

Each repo's Spec Kit can now generate a `tasks.md`: T001, T002, … a dependency-ordered list. But
remember Problem 2 — that list is **horizontal**. It has no idea which of those tasks finish story 1.

```
@feature.tasks 015-hold-queue
```

**Your job is the mapping — turning a flat list into vertical slices:**

| Story | API slice | UI slice |
|---|---|---|
| US1 | T001–T009 | T001–T006 |
| US2 | T010–T015 | T007–T012 |
| US3 | done (already merged) | T013–T018 |

The API slice is ordered before the UI slice, always, because the UI consumes the API. That table
goes into the manifest and it is the work order for the entire inner loop. `@feature.implement US1`
reads exactly this.

**Two rules that are not negotiable:**

**Never lose a task.** Every task ID in every repo must end up in a story slice, in the shared setup
block, or in an explicitly-excluded block. The check is arithmetic: *sum of slices + setup + excluded
== total tasks*. Orphans are how a horizontal list quietly fails to become vertical while looking
like it succeeded.

**Excluded stays excluded.** If a story is out of scope, its tasks are recorded as out of scope, not
quietly absorbed into an in-scope story. This is the most common way scope creep re-enters after the
customer signed it out.

**Done when:** every in-scope story has a non-empty slice in every repo that serves it, and the
arithmetic balances.

---

## 5a. Analyze — Tech Lead *(optional, but do it)*

```
@feature.analyze 015-hold-queue
```

A read-only audit across everything: does each acceptance criterion map to a task *and* to a test?
Do the contracts still agree? Are the cited decisions still current? Is the manifest telling the
truth?

It writes nothing and fixes nothing — it reports, ranked by severity, and routes each finding to
whoever owns the repair. Run it here (before code is written, when findings are cheap) and again
before shipping.

**A status lie is the highest-severity finding it can produce**: a gate marked green with no test
run, a signoff with no signer. Those erode the manifest's entire purpose, which is to be the one
thing you can trust without re-checking.

---

# The inner loop: one story at a time

Everything above happened once. Everything below happens **per user story**, and then repeats.

## 6. Implement — Developer

```
@feature.implement 015-hold-queue US1
```

You build **one story**, all the way across, backend first, then frontend. Not two stories. Not
"while I'm in here." One.

This will feel wrong. You are in the backend code with the whole feature in your head and building
story 2's endpoint would take twenty minutes. **Don't.** The moment you do, story 1 is no longer
independently shippable, the gate no longer means anything, and you are back to horizontal slicing
wearing a vertical costume.

The actual code is written inside each repo's own constitution — its TDD rules, its API-client
pattern, its component library, its accessibility bar. You are not re-litigating those here.

Three rules:

- **Trim excluded payload.** Excluded work hides inside shared tasks. A shared routing task might add
  both an in-scope screen and an out-of-scope one. Build only the in-scope half.
- **Config, not literals.** Business values come from configuration, referenced by name. Always.
- **Don't invent contracts.** If the API has shipped, build against it exactly as it shipped. A gap
  is a question you raise, not a name you make up.

**Done when:** the build is green and unit tests pass, locally, committed to the feature branch. Not
pushed. Green build is *not* "done" — it means "ready to be tested."

---

## 7. VERIFY — QA 🚦 **GATE**

```
@feature.verify 015-hold-queue US1
```

**This is where QA gets dramatically more automated, and it's worth understanding why.**

Look at what happened to a single acceptance criterion. The Feature Manager wrote it in the catalog
as Given/When/Then. It was carried into `feature.md`, projected into each repo's `spec.md`, mapped to
a task in `tasks.md`, and tagged to a test as `@015-us1`. **The same sentence, unchanged, through
every artifact.**

So QA doesn't sit down with a finished feature and invent a test plan by reverse-engineering what the
developer probably meant. The test is a *transcription* of a sentence the customer already signed. The
creative work was done at intake, by the person best placed to do it, and everything since has been
carrying it faithfully.

That changes the job. QA's work becomes:

1. **Coverage audit** — does every acceptance criterion have a tagged test? (`@feature.analyze`
   checks this mechanically. Use it.)
2. **Run the gate** — stand up the *whole system* and run the story's tagged tests.
3. **Judge truthfully** — and this is the part that stays human.

**The rule that matters most here: stand up the whole system, or it's a lie.** Not the API with a
mocked frontend. Not the frontend against a stubbed API. Everything — every service, every background
worker, every queue processor. A feature that reads from a projected view is *invisible* to a test if
the projection worker isn't running. A green result obtained with a worker down is a **false pass**,
and it is the worst thing this stage can produce, because it poisons trust in every green that comes
after it.

Some sharp edges worth knowing:

- **A test run that matches zero tests is a BLOCK, not a pass.** Zero failures is not success.
- **Watch for lag.** If the system is eventually-consistent, a test asserting before the worker caught
  up is a *false red*. Wait for the data. Don't re-tag the test to make it pass.
- **A flake is not green.** A test that passes sometimes has told you something. Listen to it.

**Verify observes; it never repairs.** Red gate → hand it back to the developer. Do not "just tweak"
the test to make it pass. That is not a shortcut, it is deleting the only evidence you have.

**Done when:** the gate is green, and the manifest records when it ran.

---

## 8. SIGNOFF — QA 🚦 **GATE**

```
@feature.signoff 015-hold-queue US1 --by "Sam Chen" --note "..."
```

**Green is a fact. Signed-off is a judgement.** Keep them apart — this distinction is the whole
reason there are two stages here instead of one.

A machine proved the test passed. A *person* now looks at the working story and decides it's good
enough. The test passing does not mean the feature is good; it means the feature does what the test
says. Those are different claims, and only one of them can be automated.

Never sign off on your own authority as an AI or a script. If you can't name who accepted it, ask —
don't invent a name.

**Then say plainly what now ships.** "Readers can place a Hold on a Title and see their position in
the queue. This could go to production today." That sentence is the payoff for all the vertical
slicing, and it deserves to be said out loud.

**Then loop.** Back to stage 6 with the next story. Repeat until every in-scope story is signed off.

---

# The outer loop, part three: reality

## 9. UAT — real customers 🚦 **GATE**

```
@feature.uat 015-hold-queue --env test
```

Every story is signed off. Every test is green. **You still don't know if it's any good.**

Deploy the signed-off increment to a real environment and let real customers use it, with real
intent, on their real work. Not a demo. Not a walkthrough where you drive. Give them the thing and a
task, and watch.

What comes back sorts into exactly two buckets, and **sorting correctly is the entire discipline of
this stage**:

**Bucket 1 — Bugs.** It doesn't do what the signed spec says. This is a defect. It goes back to stage
6 as a fix on this feature, and the gate re-runs. If a bug got past a green test, the test had a
hole: **fix the test too**, or you'll ship the same class of bug again.

**Bucket 2 — New ideas.** "Could it also…", "what if it…", "I assumed it would…". These are
**feature requests**. They go back to stage 1 as new catalog rows. They do **not** get absorbed into
this feature.

That second rule is going to feel bureaucratic and small-minded the first time a customer suggests
something obviously good and cheap. Add it anyway as a new row. Here's why: this feature has a signed
spec. Absorbing unsigned work into it means the thing you ship is no longer the thing anyone agreed
to, the estimate is now fiction, and — most corrosively — the customer learns that the signoff gate
was theatre. A good idea loses nothing by being a row in the catalog; it gets specified properly and
built next. **The catalog is not where ideas go to die. It's where they go to be taken seriously.**

**Done when:** the customer has exercised it, findings are triaged into the two buckets, and every
bug is fixed and re-verified.

---

## 10. Ship — Developer

No open bugs. Customer's had their hands on it. Ship it.

```
@feature.ship 015-hold-queue --to prod
```

Merge in dependency order — API before UI, because the UI consumes the API and the reverse order
means a window where the frontend is calling an endpoint that doesn't exist yet. Then deploy, then
watch it. Then mark the feature `shipped` in the manifest and set the catalog row to Shipped.

Then close the loop: **the bucket-2 ideas from UAT are now sitting in the catalog as new requests,**
and the Feature Manager picks the best one up at stage 1. That's not the process restarting. That's
the process working — a customer used the software, told you what they actually wanted, and that
went in the front door with everything else.

---

## The map, one more time

| # | Stage | Who | Artifact it produces | Gate? |
|---|---|---|---|---|
| 1 | Intake | Feature Manager | Catalog row, Ready-for-spec | |
| 2 | Specify | Feature Manager | `feature.md`, `manifest.yaml`, per-repo `spec.md` | |
| 3 | **Spec signoff** | **Customer** | `spec_signoff: {by, at, note}` | 🚦 |
| 4 | Plan | Tech Lead | `plan.md`, `data-model.md`, `contracts/` + reconciliation | |
| 5 | Tasks | Tech Lead | Per-story slices in the manifest | |
| 5a | Analyze | Tech Lead | Findings, routed | |
| 6 | Implement | Developer | Code, green build, local commit | |
| 7 | **Verify** | **QA** | `done_gate: green` + timestamp | 🚦 |
| 8 | **Signoff** | **QA** | `signoff: {by, at, note}` | 🚦 |
| 9 | **UAT** | **Customer** | Triaged findings | 🚦 |
| 10 | Ship | Developer | Merged, deployed, `status: shipped` | |

---

Next: [03 — Structure](03-structure.md) · or jump to your own runbook in [04 — Roles](04-roles.md)
