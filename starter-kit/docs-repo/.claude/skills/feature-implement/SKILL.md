---
name: "feature-implement"
description: "Carry ONE user story end to end across the repos: read that story's slices from the manifest, dispatch a per-repo implementer inside each repo's own constitution, tick tasks.md, build to green, and make a LOCAL commit on the feature branch."
argument-hint: "<NNN-slug> [US{n}]"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.implement.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 6 — Implement (Developer)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.implement.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

Dispatches `repo-implementer` sub-agents — API FIRST, awaited, THEN UI. Two blocks, deliberately; do NOT parallelize across the seam. The first stage that writes src/. Never touches a merged repo, the default branch, or a remote. If any repo is red or blocked, STOP and leave the work in place.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
