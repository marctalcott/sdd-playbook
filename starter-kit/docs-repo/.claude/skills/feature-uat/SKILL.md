---
name: "feature-uat"
description: "Deploy the signed-off increment to a real environment, record who exercised it, and triage what comes back into exactly two buckets — bugs back to this feature, requests as NEW catalog rows."
argument-hint: "<NNN-slug>"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.uat.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 9 — UAT (real CUSTOMERS decide)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.uat.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

THE TWO-BUCKET RULE IS ABSOLUTE: a new idea becomes a new row in feature-catalog.md and is never absorbed into a feature that is already signed. Never close a bug without closing the test gap that let it through. Deploying is outward-facing — confirm before acting.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
