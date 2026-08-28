---
name: "feature-ship"
description: "The closing beat: confirm UAT passed with no open bugs, merge each in-scope repo's PR in dependency order (api before ui), deploy to production, and confirm every UAT request became a catalog row."
argument-hint: "<NNN-slug>"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.ship.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 10 — Ship (Developer)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.ship.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

OUTWARD-FACING AND HARD TO REVERSE. Every merge and every deploy is opt-in and pre-checked — confirm with a human before each. Never merge a red PR; never ship with open UAT bugs. Know what merging actually triggers in THIS workspace before you merge — in many setups a merge to the default branch deploys to a test environment and production is a separate manual step. Confirm which it is; do not assume the merge is the deploy, or that it isn't.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
