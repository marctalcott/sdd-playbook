# Running a `feature.*` stage from Claude Code

Every `feature-*` skill in `.claude/skills/` is a **thin wrapper**. It holds no copy of its
stage's rules. The rules live in exactly one place —
`<product>-docs/.github/agents/feature.<stage>.agent.md` — and this file is the only translation
layer between how that file is written (for GitHub Copilot) and how you execute it (in Claude
Code).

**Why it is built this way.** `AGENTS.md` says two copies of a rule become two different rules
within a month. A stage that exists as both a Copilot agent and a Claude skill is exactly that
hazard, so there is one body and two front-ends. If you are tempted to "just fix it here"
because the `.agent.md` reads awkwardly — don't. Fix the `.agent.md`; both runners get it.

---

## 1. Resolve the workspace

Walk up from the cwd until you find a folder holding `<product>-docs/` and the code repos as
**siblings**. That is `<workspace>` (for example `~/src/<product>`). If a sibling is
missing, stop and report which.

Every path in a stage file is relative to `<workspace>` unless it says otherwise.

## 2. Read the stage file — it *is* your instructions

Read `<workspace>/<product>-docs/.github/agents/feature.<stage>.agent.md` in full before acting.
Its frontmatter is Copilot metadata; **the body is the spec you execute.** Ignore only the
frontmatter keys translated below.

Read the artifacts it tells you to read — most stages point at
`<product>-docs/features/_template/manifest.yaml`, whose inline `[stage]` brackets are authoritative
about which fields your stage may write. Read it rather than assuming.

## 3. Translate the frontmatter

| Copilot frontmatter | What it means for you |
|---|---|
| `tools: ['codebase', 'search']` | Read, Glob, Grep |
| `tools: [… 'editFiles']` | Edit, Write — **only** within the stage's stated write scope |
| `tools: [… 'runCommands']` | Bash |
| `tools: [… 'agent']` | the Agent tool (see §4) |
| **absence** of `editFiles` | the stage is read-only. Do not write. |
| **absence** of `runCommands` | the stage does not run commands. Read and edit only. |
| `agents: ['repo-x']` | dispatch `subagent_type: "repo-x"` — defined in `.claude/agents/` |
| `handoffs:` | **not automatic here** (see §5) |

A stage's write scope is stated in its own body, usually under *Operating constraints*. That
prose is narrower than the tool list and it **wins**. `feature.analyze` carries `editFiles` and
is still read-only apart from one log line.

## 4. Sub-agents

Where a stage says "dispatch a `repo-x` sub-agent", call the Agent tool with
`subagent_type: "repo-x"`. The six `repo-*` agents in `.claude/agents/` exist for this and
nothing else.

- **"in a single message so they run in parallel"** — put every Agent call in **one** block.
  Separate blocks run them in sequence and waste the parallelism the stage asked for.
- **"api first, await it, then ui"** (`feature.implement`) — this is the opposite instruction.
  Two blocks, deliberately. The UI consumes the API; parallelising across the seam produces a
  rebuild, not a slice.
- **Every prompt must be self-contained.** Pass the `NNN` and slug **you** resolved, the repo
  path, and the context pulled from `<product>-docs/`. A sub-agent that re-derives a number will
  derive a different one.
- Sub-agents report back; **you** write the coordination-layer artifacts. Never let a sub-agent
  write `<product>-docs/features/`.

## 5. Handoffs are printed, not taken

Copilot chains stages automatically. **Here, you stop.** Finish your stage, emit its Output
block, and print the next command as a suggestion:

```
Next: /feature-spec-signoff 015-hold-queue
```

Running the next stage yourself collapses the gates the pipeline exists to enforce. Wait to be
asked.

## 6. The rules that do not bend

- **Human gates are human.** `spec_signoff` and `signoff` carry a real person's name. Never
  fill one, never invent a timestamp for one, never sign "the team" or an AI.
- **Green is a fact.** `done_gate.status: green` is set off a real run whose output you saw. A
  stage that could not run its gate reports that; it does not assume.
- **Names are law.** Terms come from `glossary.md`, `personas-and-jobs.md`, `decisions.md`. A
  missing term is an Open Question, never a coined synonym.
- **A verifier never repairs.** `feature.verify`, `feature.analyze` and `feature.signoff` route
  a red finding back to the owning stage. They do not fix it in passing.
- **`status: blocked` is a real outcome.** A stage that cannot proceed says so and stops. It
  does not improvise past a missing precondition.

## 7. Arguments

Stages accept the flags their own body documents — commonly `--number NNN`, `--slug <kebab>`,
`--repos api,ui`. Parse them from the skill invocation and pass them through. An explicit flag
always beats a derived value; never re-derive a number the caller supplied.

## 8. Report honestly

End with the stage's own Output block verbatim — including its `{ status: … }` return line
where it specifies one. If you skipped a step, say which and why. A stage that reports success
it did not achieve is worse than one that reports `blocked`.
