---
name: "feature-spec-signoff"
description: "Record a real customer accepting the WHAT before any code is written. Presents the spec so a non-technical person can disagree with it, then records spec_signoff. Blocks feature.plan until filled."
argument-hint: "<NNN-slug>"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.spec-signoff.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 3 — Spec signoff (the CUSTOMER decides)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.spec-signoff.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

GATE 1. `spec_signoff.by` is a real person's name and org — never "the team", never an AI, never a timestamp you invented. If the customer has not actually answered, this stage ends `blocked`.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
