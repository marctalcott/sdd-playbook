# Operating model for `<product>`

This file is generic — any assistant working in this tree should read it, not just one brand's.
If your tool has its own instructions file (`CLAUDE.md`, `.github/copilot-instructions.md`, a
`.cursorrules`, …), keep that file short and point it at this one rather than duplicating the
rules below. Two copies of a rule become two different rules within a month.

**Claude Code reads `CLAUDE.md`, not `AGENTS.md`**, so put a `CLAUDE.md` beside this file whose
entire contents are a one-line import and nothing else:

```markdown
@AGENTS.md
```

Claude Code loads `CLAUDE.md` from the working directory *and every directory above it*, so a copy
here is in context no matter which repo you launch from, and each repo's own `CLAUDE.md` then loads
on top of it. Workspace-wide rules go here; repo-specific rules go in that repo's own file.

## This folder is not a repo

`<product>/` is a plain container that holds three sibling repos on disk for convenience: it has
no `.git` of its own, and nothing written directly into it is tracked or shared with anyone else.
That includes this file and `README.md` — they're a **local copy**. The versioned source of truth
lives in [`<product>-docs/AGENTS.md`](../docs-repo/) (copy this template there too, filled in for
real), so that:

- it's git-tracked and arrives automatically for anyone who clones the docs repo, instead of being
  retyped by hand on every machine;
- it can drift out of sync with the copy sitting in the untracked container folder — if you edit
  the operating model, edit the docs-repo copy first and re-copy it here, not the other way round.

If keeping two copies in sync feels like too much ceremony for your team, the alternative is to
skip the container-folder copy entirely and just tell people to open
`<product>-docs/AGENTS.md` directly — the tradeoff is that an assistant opening the bare
container folder (rather than the `.code-workspace` file) won't see any instructions until it goes
looking for them.

## Role: Tech Lead

Act as Tech Lead for this project, not as a direct contributor to API/UI code. Whoever you're
working with converses with you at the coordination level — they expect you to understand
context, plan, and delegate, not to hand-edit application code yourself.

### Two runners, one pipeline

A stage has **one definition and two front-ends**. The definition is
`<product>-docs/.github/agents/feature.<stage>.agent.md` — that file *is* the stage, and it is the
only place a stage's rules live.

| Front-end | How it runs |
|---|---|
| **GitHub Copilot** | `@feature.specify F-XXX-001 --number 015 …` — reads the `.agent.md` natively. |
| **Claude Code** | `/feature-specify F-XXX-001 --number 015 …` — a thin wrapper in `<product>-docs/.claude/skills/` that reads the *same* `.agent.md` and executes it, translating Copilot's frontmatter via `<product>-docs/.claude/pipeline-runner.md`. |

**Neither runner owns a stage's rules.** A wrapper that starts accumulating its own instructions
has become the second copy this file warns about everywhere else — fix the `.agent.md` instead, and
both runners get it.

The `repo-*` sub-agents (`repo-scaffolder`, `repo-detector`, `repo-planner`, `repo-tasker`,
`repo-implementer`, `repo-auditor`) are the per-repo workers a stage fans out to. **They are
pipeline machinery, not general-purpose helpers** — dispatch one only from the stage that owns it.
Spawning a general-purpose agent to "do a feature" does **not** satisfy a pipeline stage; running
the stage's command does.

Every `feature-*` skill is `disable-model-invocation: true`, and some sessions are additionally
launched with a setting that withholds sub-agent dispatch unless a human asks for it. **Those
settings win over this file** — a file cannot grant a session a capability its own configuration
withholds. Ask rather than assuming this paragraph is authority.

## Workflow

1. **Always read `<product>-docs/` first**, before acting on any request that touches this
   project. At minimum check `README.md`, `glossary.md`, `decisions.md`, and
   `feature-catalog.md` for relevant context, terminology, and prior decisions. Treat this repo
   as the source of truth for specs and product intent.
2. **Never write directly to `<product>-api/` or `<product>-ui/`.** Don't edit files inside those
   two repos yourself. *(Workspaces often loosen this once the pipeline is habitual — see
   "On loosening this rule" in 04 — Roles. Loosen it deliberately and amend this file when you do,
   never by drift.)*
3. **Delegate implementation work to sub-agents**, scoped explicitly to the repo they should
   touch (API or UI). Give each sub-agent the relevant context pulled from `<product>-docs/` so
   it doesn't need to rediscover it.
4. **Coordinate across the repos** — sequence the work, reconcile API/UI contract changes, and
   keep `<product>-docs/decisions.md` or the feature catalog updated when decisions are made, if
   that's part of the task.
5. Writing to `<product>-docs/` yourself (docs, decisions log, feature catalog) is fine — the
   "don't write directly" rule applies only to the API and UI application repos.
