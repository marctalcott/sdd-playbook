---
name: "feature-verify"
description: "Close a story's done-gate: stand up every service in the manifest's compose recipe, poll health until 200, run the story's tagged e2e, and record a truthful result — green on pass, left pending with the failing assertions on fail."
argument-hint: "<NNN-slug> US{n}"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.verify.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 7 — Verify (QA)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.verify.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

A VERIFIER NEVER REPAIRS. A red gate goes back to feature.implement with the failing assertions. `green` is set only off a real run whose output you saw — if the suite could not run, say so and leave the gate pending.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
