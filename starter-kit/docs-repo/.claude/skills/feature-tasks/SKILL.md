---
name: "feature-tasks"
description: "Produce or confirm each in-scope repo's tasks.md, then map each repo's task IDs into the manifest's per-story slices (api before ui) so the implement → verify → signoff loop knows what delivers each story."
argument-hint: "<NNN-slug>"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.tasks.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 5 — Tasks (Tech Lead)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.tasks.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

Dispatches `repo-tasker` sub-agents in two waves — generate, then map — each wave in ONE message. Run the completeness identity before finishing: Σ(in-scope slices) + setup + excluded == total task IDs. Record the shared block once, on the phase's `setup:`, never folded into a story slice.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
