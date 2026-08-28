---
name: "feature-specify"
description: "Promote a Ready-for-spec row from feature-catalog.md into ONE cross-repo feature spec + manifest under <product>-docs/features/NNN-slug/, cut the per-repo branches, and write each in-scope repo's functional spec.md."
argument-hint: "<F-CATALOG-ID> [--number NNN] [--slug kebab-name] [--repos api,ui]"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.specify.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 2 — Specify (Feature Manager)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.specify.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

Dispatches `repo-scaffolder` sub-agents — all in ONE message, in parallel. Numbering is owned by THIS stage: do not use Spec Kit's `create-new-feature.sh`, which derives its own number and will fight you. Writes `spec.md` only — never plan.md, tasks.md, or src/.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
