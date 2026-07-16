# 06 — Rollout

> **Read this if** your team has never done this and you're wondering where to start.

---

## Don't build the whole thing

The layout in [03](03-structure.md) and the eleven agents in [05](05-copilot-setup.md) are where you
end up, not where you start. A team that stands up the full machinery in week one will spend week two
debugging YAML instead of learning the discipline — and the discipline is the part that actually
works.

**The tooling is not the hard part. Agreeing what words mean is the hard part.**

So roll out in the order of *value earned*, where each step makes you want the next one. Roughly five
weeks, and you get real benefit from week one.

---

## Week 0 — One person, one toy feature

**Who:** whoever is most curious. One person, not the team.
**Goal:** know what Spec Kit actually does before designing a process around it.

Do the [Spec Kit quickstart](https://github.github.com/spec-kit/quickstart.html) in a throwaway repo.
Run `/speckit.constitution`, `/speckit.specify`, `/speckit.plan`, `/speckit.tasks`,
`/speckit.implement` on something trivial. An hour.

**What you're learning:** the shape of the artifacts, and — more usefully — how much the quality of
the output depends on the quality of what you fed it. That observation is the whole reason the rest of
this playbook exists.

**Don't:** show anyone a demo yet. A toy feature demos beautifully and teaches the team the wrong
lesson, which is that this is easy.

---

## Week 1 — The glossary and the catalog. No tooling.

**Who:** the whole team, in a room.
**Goal:** agree what words mean.

This week has **no AI in it at all**, and it is the highest-value week of the rollout.

**Day 1–2: write `08-glossary.md`.** Every domain term, one name each. This will take longer than you
think and it will surface disagreements you didn't know you had — that's not the process going badly,
that's the process working. The fights you have this week are fights you'd otherwise have had in a
code review in month three, with code already written on both sides of the misunderstanding.

Expect to discover you have two words for one thing, or one word for two things. Both are common.
Both are expensive later.

**Day 3: write `03-product-principles.md`.** Five to ten, numbered. Things you'd say no to a customer
over.

**Day 4–5: write three catalog rows.** Full ones — user story, three Given/When/Then criteria, linked
principles and personas, out-of-scope, dependencies.

**The test for this week:** hand a catalog row to someone who wasn't in the room. Can they tell you
exactly what would and wouldn't be built? If not, the row isn't done — and no tooling will save it.

> **The temptation to skip this is enormous** and it's the mistake that kills rollouts. Writing a
> glossary feels like procrastinating. It isn't. Every downstream artifact — spec, plan, task, test —
> inherits this vocabulary, and renaming a concept once it's in three repos and forty tests is a week
> you don't get back.

---

## Week 2 — One repo, one real feature, one gate

**Who:** Feature Manager + one developer + one customer.
**Goal:** feel the spec signoff gate.

Pick your **simplest real feature that touches one repo only**. Not the interesting one. The boring
one.

1. Install Spec Kit in that repo (`specify init . --integration copilot`).
2. Write the constitution. Be strict.
3. Feature Manager runs `/speckit.specify` from a catalog row, reviews it against the glossary, runs
   `/speckit.clarify`.
4. **Take it to the customer. Read the acceptance criteria out loud. Get a yes.**
5. `/speckit.plan`, `/speckit.tasks`, `/speckit.implement`.

**No manifest. No feature folder. No custom agents.** One repo, stock Spec Kit, plus one gate.

**What you're learning:** what a customer says when you read them a spec. Nearly every team is
surprised here. The acceptance criteria you were sure about turn out to have an assumption baked in,
and finding that out on a Tuesday before any code exists is the entire pitch of this playbook,
delivered in week two for almost no setup cost.

---

## Week 3 — QA's gate

**Who:** add QA.
**Goal:** the criterion → test thread.

Same single repo. This time, before QA touches anything, they check: **does every acceptance criterion
have a test tagged to it?**

Adopt the `@NNN-us{n}` tag convention now. Tag each test with the story it proves. Then run the gate
by tag, and record the result somewhere — a file, a comment, anywhere. You don't have a manifest yet;
that's fine.

Use `/speckit.checklist` and `/speckit.analyze` to check coverage mechanically.

**What you're learning:** that QA's job just changed. The test is a transcription of a signed
sentence, not an invention. This is the moment the QA role clicks, and it's worth letting them
discover it rather than telling them.

---

## Week 4 — The second repo, and the seam

**Who:** add the Tech Lead.
**Goal:** feel Problem 1, then fix it.

Pick a feature that genuinely touches **two** repos. Run stock Spec Kit in both, independently, the
way you have been.

**It will go badly, and that's the exercise.** The API will plan against an imagined client. The UI
will plan against an imagined API. The field names won't match. You'll build all the backend and
have nothing to show. Somebody will ask "so is this done?" and nobody will be able to answer, because
there is no place where that question has an answer.

**Now add the layer, because now you want it:**

1. Create `<product>-docs/features/` and copy in `_template/manifest.yaml`.
2. Set up the multi-root `.code-workspace` — [05, step 5](05-copilot-setup.md#step-5--the-multi-root-workspace-this-is-the-important-one).
3. Copy in the `feature.*` agents from the starter kit.
4. Re-run the feature through `@feature.specify` → `@feature.plan` → `@feature.tasks`.
5. Watch `@feature.plan` catch the contract mismatch you already hit by hand.

**What you're learning:** why the manifest exists. Teams that are handed the manifest in week one
experience it as bureaucracy. Teams that spend a week without it experience it as relief. Same file.

---

## Week 5 — The inner loop and UAT

**Who:** everyone.
**Goal:** ship one story on its own.

Run one feature the whole way. Vertical slices. `@feature.implement US1` → `@feature.verify US1` →
`@feature.signoff US1`. Then **stop** and ask: could we ship this right now, on its own?

If the answer is yes, that's the payoff and you should say it out loud in the standup.

Then deploy the signed-off increment to a test environment and get a customer to use it for real
work. Triage what comes back into the two buckets. Notice that some of it is bugs and most of it is
new feature requests — and put the requests in the catalog rather than in this feature.

**What you're learning:** the loop closes. A customer used the software and their reaction went in the
front door with everything else.

---

## The order matters

Each week earns the next:

| Week | You feel | So you want |
|---|---|---|
| 1 | "We disagree about what a Title is" | A glossary |
| 2 | "The customer changed the spec in five minutes, before we'd built it" | The spec signoff gate |
| 3 | "The test is just the acceptance criterion" | The tag convention |
| 4 | "Two repos, no answer to 'is it done'" | The manifest |
| 5 | "Story 1 could ship today" | The inner loop |

Handing a team all five at once means they experience the whole thing as overhead, because they never
felt the problem any of it solves. **Let them feel the problem first. Then the fix reads as a fix.**

---

## How to tell it's working

Signs it's landing:

- Arguments happen at intake, in a document, instead of in code review.
- Somebody says "that's not in the glossary" without being prompted.
- A customer changes their mind at the spec gate and it costs an afternoon.
- Somebody notices a story could ship on its own.
- A UAT idea goes into the catalog and nobody argues about it.

Signs it isn't:

- The catalog rows are one-liners with a "TBD" acceptance criterion.
- Specs are being hand-edited rather than regenerated.
- The spec signoff is an email nobody replied to, recorded as approval.
- QA is writing test plans from scratch again.
- Gates are green but nobody can say when the test last ran.
- Every UAT finding becomes "a small addition to this feature."

Every one of those has a fix in [07 — Pitfalls](07-pitfalls.md).

---

## Two things that go wrong early

**Someone automates a gate.** It's the obvious next step — the pipeline is so mechanical, why is a
human clicking approve? Because the gates are the only places where the process touches reality. A
machine can prove a test passed; it cannot decide the feature is good. If you automate the judgement,
you have a very efficient pipeline for shipping the wrong thing.

**Someone makes the feature layer thick.** It starts with a status field, then a nice summary that
duplicates the spec, then a copy of the API contract "for convenience." Now you have two copies of
everything and they disagree within a month. Layer 2 is **two files per feature**, and the discipline
of keeping it there is what makes it trustworthy.

---

Next: [07 — Pitfalls](07-pitfalls.md)
