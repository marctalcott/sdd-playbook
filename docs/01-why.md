# 01 — Why this exists

> **Read this if** you are wondering why a Spec Kit install isn't already enough.

---

## The four problems

### Problem 1: Nobody can remember why

Start here, because this is the one that outlives everything else on the list.

Every project accumulates rules that look arbitrary from the outside:

> Why can't a reader place a hold on a book they already have out?
> Why is that number in a config file instead of just written in the code?
> Why is the large-print edition a separate record instead of a copy of the normal one?

Somebody had a good reason for each. It was probably a *hard-won* reason — the kind you only get by
shipping something, watching it fail in a way nobody predicted, and fixing it. But the reason was
never written down, because at the time it was obvious to everyone in the room.

Six months on, that reason exists in one person's head. When they go on holiday it's inaccessible.
When they leave it's gone, and what remains is **folklore**: code nobody dares change, and rules
nobody can justify — or repeal. New joiners work around the rules they don't understand, because
working around them is safer than asking. The rules calcify precisely because they're unexplained.

Then AI arrives and makes it sharper. A model generating a plan **cannot** intuit a decision your
team made a year ago. It will confidently generate something reasonable and wrong, in exactly the
way the decision existed to prevent — and it will do it fluently enough that a reviewer who wasn't
there won't catch it.

There is no test that fails when institutional memory is lost. It just gets quietly more expensive
to change anything, forever.

### Problem 2: Spec Kit can only see one repository

Spec Kit installs into a repo. Its `/speckit.specify`, `/speckit.plan`, and `/speckit.tasks`
commands read that repo and write into that repo. This is a sensible design and it is also the
source of the whole difficulty, because **a real feature is not shaped like a repo**.

Take a plain user story:

> As a reader, I want to join the queue for a book that's on loan, so I get the next one back.

Delivering that means a backend endpoint, a frontend screen, and a test that drives the screen
against the endpoint. Three repos, or at minimum two. Now run Spec Kit in each of them. The backend
install writes a plan for an API with an *imagined* client. The frontend install writes a plan for a
screen against an *imagined* API. Nobody planned the seam between them, because no install can see
both sides.

What comes out is **horizontal slicing**: all the backend, then all the frontend. Each repo's
`tasks.md` is one flat sequence with no notion of *which of these tasks finish user story 1 end to
end*. And a horizontal plan has a nasty property — you have nothing shippable until the last task
of the last repo lands. Story 1 is not done until story 5 is done.

### Problem 3: Polyrepo has no commit that means "this feature is done"

In a monorepo, a feature is a branch, and merging it means the feature shipped. In a polyrepo, a
feature is *two* branches in *two* repos that must both merge, in order, with a test that lives in
one of them exercising code that lives in the other.

There is no single artifact that says: these are the repos, this is the branch in each, this is the
task range in each that delivers story 2, and this is the test that proves it. That artifact doesn't
exist, so it lives in someone's head — and it is lost the moment they go on holiday.

### Problem 4: Specs drift from code, silently

Everyone has seen a spec that stopped being true. The mechanism is always the same: something was
learned during implementation, the code changed to match reality, and the spec didn't. It is nobody's
job to keep them in sync, and there is no test that fails when they diverge.

Under spec-driven development this is not an annoyance, it is fatal. If the spec is the artifact you
generate from, a stale spec regenerates stale code.

---

## What this layer adds

Four answers, one per problem.

**A separate repo for the documentation — and the decisions in it.** You create a new repository that
holds **no application code at all**. It holds the vision, the personas, the principles, the
vocabulary, the backlog, and the **decisions**: numbered, dated, and — the part that matters —
carrying the *reasoning*, not just the ruling.

That repo is the answer to Problem 1. Go back to the three questions it opened with — in the example
product, each one has an address:

| The question a newcomer asks | The answer |
|---|---|
| Why can't a reader hold a book they already have out? | **D-005** — they were parking at the front of the queue. D-004 allowed it; it was superseded, and D-004 is still there saying why. |
| Why is that number in config instead of the code? | **D-002** — policy changes on policy timescales, and a literal in a spec becomes a magic number nobody can identify later. |
| Why is large-print a separate record? | **D-001** — a Hold names a Title, never a Copy. That closed the door; linked editions (F-CAT-013) is the honest way through it. |

Nobody has to remember any of that. It works in two directions at once:

- **A human reads it to understand the system.** The newcomer gets `D-005` instead of a shrug: what
  the decision was, what forced it, what it traded away, and what it now forbids. They can then
  argue with it *on the merits* — which is the thing folklore never lets you do, because you can't
  contest a rule whose reasoning nobody can produce.
- **The AI reads it to generate correctly.** Every decision that constrains a feature is pulled into
  that feature's spec automatically and cited by number, so a plan generated today honours a choice
  made a year ago by someone who has since left. The model doesn't have to intuit your history — it
  is handed it.

Decisions are **append-only**: you supersede, never delete. The superseded entry stays with the
reasoning that killed it, because the alternative is that the next person proposes the same idea
again in good faith and nobody can explain why it was rejected. And when a decision *is* superseded,
the audit stage can find every spec that cited the old one — which is how you stop a story quietly
being built on a premise that stopped being true.

**Why does it have to be its own repo?** Two reasons. Docs living inside a code repo become *that
repo's* docs — they drift into describing that service, and the API's copy and the UI's copy of
"the truth" diverge inside a month. And docs in their own repo can be cited with a commit SHA
(`source: <product>-docs@a1b2c3d`), so a spec names the exact version of the truth it came from. That
is what makes regeneration reproducible rather than aspirational.

**A cross-repo spec and a manifest.** One feature gets one folder — **in that same docs repo** —
holding one specification written
for the *whole* feature — not one side of it — and one machine-readable manifest that names the
repos, the branches, the per-story task ranges in each repo, and the test that gates each story. The
manifest is the artifact polyrepo was missing. It is described in [03 — Structure](03-structure.md).

**Vertical story slices.** Instead of "all the backend then all the frontend", each user story gets a
*slice* in each repo — story 1 is API tasks T001–T009 plus UI tasks T001–T006 — and those slices are
built together, backend first, then frontend, then the test, then signoff. Then story 2 starts. This
is the single most important behaviour change in the playbook and the one that feels most unnatural
at first.

**Gates that produce evidence.** Each gate writes its result into the manifest: who signed the spec
and when, when the test last ran and whether it passed, who accepted the story. The manifest becomes
an audit trail that a human can read without diffing anything.

---

## The rule that organises it all

There is one architectural rule, and everything else follows from it:

> **Lift `specify`, `plan`, `tasks`, and `analyze` UP to the feature level.
> Keep the `constitution` and the `implement` loop DOWN in each repo.**

Read that again, because it is the whole design.

**Up** goes anything that must see across the seam. Specifying is up, because the feature spans repos.
Planning is up, because somebody has to make the API's exposed contract and the UI's consumed
contract agree — and only a layer above both can do that. Task slicing is up, because the slice map
is what makes a story vertical. Consistency analysis is up, because the interesting inconsistencies
are the cross-repo ones.

**Down** stays anything that is genuinely repo-local. Each repo keeps its own *constitution* — its
test-driven-development rules, its naming conventions, its component library mandate, its
accessibility bar. And each repo does its own implementation, inside its own constitution, because
how you write a C# handler has nothing to do with how you write a React hook and no coordinating
layer should pretend otherwise.

The coordinating layer says **what** and **in what order**. The repo says **how**. When someone asks
"should this go in the feature agent or the repo agent?", this is the question that answers it.

---

## When NOT to use this

The pipeline is heavy. It is meant to be — it is protecting you against building the wrong thing for
four weeks. But not every change deserves it. **Bypass all of this for:**

- Wording and content fixes
- Bug fixes that don't change a contract
- Internal refactors with no behaviour change
- Dependency upgrades

**Use it for** anything that changes user-visible behaviour, an API contract, a database schema, or
anything you'd have to explain to a regulator or an auditor.

The honest failure mode here is the opposite of what people expect. Teams don't reject this process
because it's too heavy for small changes — they discover that on their own within a week. They reject
it because they run it on a change that *did* deserve it, cut a corner at the spec signoff gate to
save an afternoon, and then spend three weeks discovering the customer meant something else. The
gates are load-bearing.

---

## What you should expect it to feel like

For about two weeks, slower. You are front-loading work that you used to do late and expensively:
arguing about what the feature means, agreeing the vocabulary, deciding what's out of scope. Doing
that in a document feels like bureaucracy right up until the first time a customer signs a spec and
you build exactly it.

The moment it clicks is usually the first time a story is signed off and someone realises it could go
to production *right now*, on its own, while the rest of the feature is still a sketch. That is the
thing the vertical slicing bought, and it is not obvious until you see it.

---

Next: [02 — The pipeline](02-pipeline.md)
