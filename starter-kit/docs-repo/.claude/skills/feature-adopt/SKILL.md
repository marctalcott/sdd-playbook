---
name: "feature-adopt"
description: "Wrap an ALREADY IN-FLIGHT feature in a cross-repo feature.md + manifest.yaml reflecting reality. Detects existing numbers, branches and per-repo state; never scaffolds, never mints a number."
argument-hint: "<F-CATALOG-ID or existing NNN-slug>"
compatibility: "Requires the multi-root workspace — <product>-docs/ and the code repos as siblings"
metadata:
  author: "<product>-docs"
  source: ".github/agents/feature.adopt.agent.md"
user-invocable: true
disable-model-invocation: true
---

# Stage 2b — Adopt (Tech Lead)

**This skill holds no rules of its own.** It is a thin wrapper so that the stage has exactly
one definition rather than a Copilot copy and a Claude copy that drift apart.

Resolve `<workspace>` first: walk up from the cwd until you find a folder holding
`<product>-docs/` and the code repos as **siblings**. If a sibling is missing, stop and report which.

Then read these two files, in this order, before doing anything else:

1. **`<workspace>/<product>-docs/.claude/pipeline-runner.md`** — how a `feature.*` stage is
   executed from Claude Code: the Copilot→Claude tool mapping, how to dispatch the `repo-*`
   sub-agents, and why handoffs are printed rather than taken.
2. **`<workspace>/<product>-docs/.github/agents/feature.adopt.agent.md`** — **that file is your
   instructions.** Its frontmatter is Copilot metadata; execute its body.

Pass the arguments this skill was invoked with into the stage's own *User input* section.
An explicit flag always beats a value you would otherwise derive.

Dispatches `repo-detector` sub-agents — all in ONE message, in parallel. This stage DETECTS; it never creates a branch and never mints a number. It will not fabricate a `spec_signoff` for a feature that predates the gate.

Finish with the stage's own Output block, then **stop** and print the next command as a
suggestion. Do not run the next stage yourself — the gates are the point.
