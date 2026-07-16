# 04 — Roles

> **Read this if** you know the pipeline and you want your own runbook. Find your role, read that
> section. [02 — The pipeline](02-pipeline.md) is the narrative; this is the checklist.

Four roles plus the customer. On a small team one person wears several hats — that's fine, and common.
What is **not** fine is one person wearing two hats *on the same feature at the same gate*. If you
wrote the spec, you should not be the one who signs it off. The gates work because they're a second
pair of eyes; a gate you pass to yourself is a comment.

| Role | Owns | Gates they hold |
|---|---|---|
| [Feature Manager](#feature-manager) | The WHAT | — (facilitates the spec signoff) |
| [Tech Lead](#tech-lead) | The HOW, and the seam between repos | — |
| [Developer](#developer) | The code, and the deploy | — |
| [QA](#qa) | The evidence | Verify, Signoff |
| [The customer](#the-customer) | Reality | Spec signoff, UAT |

---

## Feature Manager

**You own the WHAT.** You are the first person to define it and the last person who can change it
without a conversation.

### Your day

**A request arrives.** Someone wants something. Your first move is not to write a spec — it's to
work out whether you understand the request well enough to write a catalog row.

**Write the catalog row** in `07-feature-catalog.md`:

```markdown
### F-LST-015 — Watch List

**Status:** Ready-for-spec
**Personas:** Buyer (P-BUY)
**Principles applied:** P-02 (buyer confidence), P-07 (no dark patterns)

**User story:**
As a Buyer, I want a list of the Listings I'm tracking, so that I can come back to
them without searching again.

**Acceptance criteria:**
- Given I am viewing a Listing, when I tap Watch, then it appears in my Watch List.
- Given a Listing in my Watch List, when the seller reduces the price, then I receive
  a notification within 24 hours.
- Given a Listing in my Watch List, when it is Sold to another Buyer, then it is
  removed from my Watch List and I am notified once.

**Out of scope:**
- Public "N people watching" badge
- Sharing a Watch List with another Buyer

**Dependencies:** F-LST-001 (Listings), F-NTF-001 (Notifications)
```

**Then specify:**

```
@feature.specify F-LST-015
```

**Then review the output properly.** Names against the glossary. Every acceptance criterion present.
Nothing added. No hard-coded numbers. Open questions flagged rather than guessed.

**Then `/speckit.clarify`** and answer its questions.

**Then take it to the customer.** Read the acceptance criteria out loud. Get a yes.

```
@feature.spec-signoff 015-watch-list --by "Dana Ortiz (Acme Ops)"
```

**Then hand off to the Tech Lead** and get out of the way. Your next involvement is triaging UAT
findings in stage 9.

### Your rules

- **Given/When/Then, every time.** Those sentences become the automated tests. This is the highest-
  leverage writing anyone does in this process. A vague acceptance criterion is a hole in production.
- **Never invent a word.** If the feature needs a term that isn't in the glossary, that's a
  conversation with the team, not a synonym you coin in a spec.
- **No implementation.** No tables, no endpoints, no libraries. If you find yourself writing "we'll
  store this in a new column", stop — you're doing the Tech Lead's job and doing it with less
  information than they'll have.
- **Never hand-edit a generated spec.** Fix the input, regenerate. A hand-edit is a lie waiting for
  the next regeneration.
- **Out-of-scope is a deliverable.** The list of what you're *not* building is the thing you point at
  when someone asks why you didn't build it.

### You will be tempted to

**Skip the customer signoff because they're busy and you're confident you know what they want.** You
have been right about this many times, which is exactly why it's dangerous. The one time you're wrong
costs more than every signoff meeting you've ever sat through. If they won't engage, that's data —
escalate it, don't route around it.

---

## Tech Lead

**You own the HOW, and — uniquely — the seam between the repos.** Nobody else in the process can see
both sides at once. That's your job.

### Your day

**The spec is signed.** It is now frozen intent. You do not get to relitigate the what. If you think
it's wrong, that's a conversation with the Feature Manager and the customer — not an edit.

**Plan:**

```
@feature.plan 015-watch-list
```

Per-repo sub-agents produce `plan.md`, `research.md`, `data-model.md`, and `contracts/`, each inside
its own repo's constitution. Then **you reconcile the contracts** — API exposes vs UI consumes,
field by field, enum by enum, status code by status code.

**Apply technical strategy here.** Does this fit the architecture? Does it need a pattern we don't
have yet? Is there a decision that should be written down before it's built rather than discovered in
a code review? This is the stage where a feature either fits the system or quietly bends it.

**Slice the tasks:**

```
@feature.tasks 015-watch-list
```

Map each repo's flat task list into per-story slices, api before ui. Check the arithmetic.

**Audit:**

```
@feature.analyze 015-watch-list
```

**Then hand the work order to the developers.** Your job during the inner loop is to unblock, not to
drive.

### Your rules

- **The merged contract always wins.** If a repo has shipped its half, its contract is a fixed input.
  Change the plan, never the shipped code. This will come up on nearly every feature and the answer
  never changes.
- **Never rewrite `spec.md`.** It's the Feature Manager's, and the customer signed it. Found a gap?
  Flag it as an open question and raise it.
- **Never lose a task.** Slices + setup + excluded must equal the total. Orphans are how a horizontal
  list fails to become vertical while looking like it worked.
- **Excluded stays excluded.** An out-of-scope story's tasks are recorded as out of scope. Never
  quietly absorbed.
- **The constitution governs how; you govern what and in what order.** Don't re-litigate a repo's TDD
  or style rules from up here.

### You will be tempted to

**Fix the contract drift yourself, in the code, because you can see exactly what's wrong and it's a
two-line change.** Don't. Flag it and route it. The moment the coordinating layer starts editing code
it stops being trustworthy as a coordinator, and you've made yourself the bottleneck for every
feature.

---

## Developer

**You own the code, and eventually the deploy.**

### Your day

**Pick up one story.** Not a feature — a story.

```
@feature.implement 015-watch-list US1
```

The manifest tells you exactly which task IDs in which repos, in which order. API slice first. Then
UI slice. The code gets written inside each repo's constitution.

**Build to green.** Compile, unit tests, lint. Commit to the feature branch. **Don't push.**

**Hand to QA.** Green build is not "done" — it means "ready to be tested."

**When the gate goes red, it comes back to you.** Fix it, hand it back. Do not argue with the test;
the test is a transcription of something the customer signed.

**When everything's signed off and UAT is clean, you ship it:**

```
@feature.ship 015-watch-list --to prod
```

API merges before UI. Then deploy. Then watch it.

### Your rules

- **One story, end to end, then stop.** This is the discipline the whole process rests on.
- **Trim excluded payload.** Out-of-scope work hides inside shared tasks — a routing task that adds
  two screens where only one is in scope. Build the in-scope half only.
- **Config, not literals.** Business values come from configuration, by name. Every time.
- **Don't invent contracts or names.** Build against the API as it actually is. Use the glossary's
  words. A gap is a question, not a guess.
- **Green build ≠ verified.** You stop at compile and unit tests. The gate is QA's.

### You will be tempted to

**Build story 2's endpoint while you're already in the file.** It's twenty minutes. You have the whole
feature in your head. It is *obviously* more efficient.

It isn't, and here's the specific reason: the moment story 2's code exists, story 1 is no longer
independently shippable, because you can't deploy story 1's branch without story 2's half-finished
work riding along. The gate stops meaning anything. You are now doing horizontal slicing while
believing you're doing vertical slicing, which is worse than doing it honestly, because you've lost
the ability to tell.

Resist it. Finish the story. Take the twenty minutes back on the next one.

---

## QA

**You own the evidence.** This role changes the most under this process, and mostly for the better.

### What changed

You used to receive a finished feature and reverse-engineer a test plan from it — reading the code,
asking the developer what it was meant to do, guessing at the edges. Most of your effort went into
*working out what should be true*, and very little into *checking whether it was*.

Now, watch what happened to one sentence:

```
Feature Manager writes, at intake:
  "Given a Listing in my Watch List, when the seller reduces the price,
   then I receive a notification within 24 hours."
        │
        ├──▶ carried into feature.md          (unchanged)
        ├──▶ projected into spec.md           (unchanged)
        ├──▶ mapped to tasks T004, T011       (traceable)
        └──▶ tagged to a test  @015-us1       (executable)
```

**The same sentence, all the way down.** The customer signed it in stage 3. The test is a
transcription of it, not an invention. The creative work was done by the person best placed to do it,
at the moment they had the most information, and everything since has carried it faithfully.

So your job shifts from *inventing what should be true* to **auditing coverage and judging truthfully**
— which is a better use of a QA engineer and a much more automatable one.

### Your day

**Audit coverage first.** Does every acceptance criterion have a tagged test?

```
@feature.analyze 015-watch-list
```

It checks the mapping mechanically: criterion → task → test. Gaps come back as findings. This is
the automation lever — use it before you touch the system.

**Generate the quality checklists** with `/speckit.checklist` — it validates requirements
completeness against the spec.

**Then run the gate:**

```
@feature.verify 015-watch-list US1
```

It stands up the whole system per the manifest's `compose` recipe, waits for health, runs
`--grep '@015-us1'`, and records the result.

**Then judge:**

```
@feature.signoff 015-watch-list US1 --by "Sam Chen"
```

### Your rules

- **The whole system, or it's a lie.** Every service, every worker. Not the API with a mocked UI, not
  the UI against a stub. A feature that reads a projected view is invisible if the projection worker
  is down — and a green obtained that way is a **false pass**, the single worst thing this stage can
  produce. It doesn't just miss one bug; it makes every future green untrustworthy.
- **Zero tests matched is a BLOCK, not a pass.** Zero failures is not success.
- **A flake is not green.** A test that passes sometimes has told you something. Find out what.
- **Mind the lag.** In an eventually-consistent system, asserting before the worker caught up is a
  *false red*. Wait for the data — don't re-tag the test.
- **You observe; you never repair.** Red gate goes back to the developer. Editing the test to make it
  pass deletes the only evidence you have.
- **Green is a fact; signed-off is a judgement.** Two stages because they're two different claims.
  Never sign off on a machine's authority, and never sign off in someone else's name.

### You will be tempted to

**Mark it green when four of five services are up and the fifth "isn't related to this feature."**
Sometimes you'll be right. But you've just made every green in the system mean "probably", and the
whole value of the gate was that it meant "definitely." If a service genuinely isn't needed, take it
out of the `compose` recipe deliberately, in the manifest, where everyone can see the decision.

---

## The customer

You hold two gates. They are the only two that check the process against reality.

### Gate 1 — Spec signoff (stage 3)

Someone reads you the acceptance criteria before anyone writes code. Your job is to disagree.

You are not reviewing a technical document. You're answering: *"Given you're watching an item, when
the seller drops the price, then you get a notification — is that right?"* You can absolutely answer
that. You can also say "yes, but not at 3am", and that sentence, said now, is worth a fortnight.

**Also review what's out of scope.** That list is what you're agreeing *not* to get. It's easier to
argue about now than to be surprised by later.

### Gate 2 — UAT (stage 9)

Every test is green. You get the real thing, in a real environment, and you use it for real work.

Everything you find sorts into two buckets:

- **"It doesn't do what we agreed"** → a bug. It gets fixed before this ships.
- **"Could it also…"** → a new feature request. It goes in the catalog and gets specified properly.

The second bucket will feel like a brush-off. It isn't. A good idea absorbed quietly into a
signed-off feature gets built without a spec, without acceptance criteria, and without tests. The
same idea as a catalog row gets all three. **The catalog is not where ideas go to die — it's where
they go to be taken seriously.**

---

## The handoffs, in one table

| Handoff | What passes | The rule |
|---|---|---|
| Request → Feature Manager | Prose | Not startable until the catalog row is complete |
| Feature Manager → Customer | The spec | **Gate.** No signoff, no plan. |
| Feature Manager → Tech Lead | Signed spec | The what is frozen. It's not up for renegotiation. |
| Tech Lead → Developer | The slice map | One story at a time, api before ui. |
| Developer → QA | Green build | Green build ≠ done. |
| QA → Customer | Signed-off increment | **Gate.** Every story signed. |
| Customer → Developer | Triaged findings | Bugs come back. Ideas go to the catalog. |
| Developer → production | Shipped | API before UI. |

---

Next: [05 — VS Code + Copilot setup](05-copilot-setup.md)
