---
name: "feature-plan"
description: "Drive the per-repo technical HOW (plan.md, research.md, data-model.md, contracts/) for a customer-signed feature, then reconcile the cross-repo contracts — the one thing per-repo Spec Kit structurally cannot do."
argument-hint: "<NNN-slug>"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.plan.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 4 — Plan (Tech Lead)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.plan.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

Dispatches `repo-planner` sub-agents — all in ONE message, in parallel. Refuses to run until Gate 1 `spec_signoff` is filled by a real person. The merged contract always wins: a merged repo is a fixed input, and the unmerged side's plan conforms to it.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
