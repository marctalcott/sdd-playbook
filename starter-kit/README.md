# Starter kit

Copy-paste implementation of the [playbook](../README.md). Everything here is generic — replace
`<product>` with your own name.

**Read [02 — The pipeline](../docs/02-pipeline.md) first.** These files won't make sense as a
process until you know what the ten stages are.

---

## What goes where

| Folder | Copy into | Contains |
|---|---|---|
| **`docs-repo/`** | `<product>-docs/` | The 11 stage definitions, **both front-ends** (`.github/agents/` for Copilot, `.claude/` for Claude Code), the manifest template, the three docs that do the daily work, and the multi-root workspace file |
| **`code-repo/`** | **each** code repo | The Copilot instructions template |
| **`workspace-root/`** | `<workspace>/` (the bare container folder) | A local, untracked `README.md` + `AGENTS.md` + `CLAUDE.md` — the operating model for whoever opens that folder directly instead of the workspace file |

```bash
# from <workspace>/, with athenaeum-docs, athenaeum-api, athenaeum-ui as siblings
cp -r sdd-playbook/starter-kit/docs-repo/.       athenaeum-docs/
cp -r sdd-playbook/starter-kit/code-repo/.       athenaeum-api/
cp -r sdd-playbook/starter-kit/code-repo/.       athenaeum-ui/
cp -r sdd-playbook/starter-kit/workspace-root/.  .
```

Then find-and-replace `<product>` → `athenaeum` everywhere (including inside `athenaeum-docs/`,
where `example.code-workspace` should be renamed `athenaeum.code-workspace`) and open
`athenaeum-docs/athenaeum.code-workspace`.

**The workspace file lives inside the docs repo, not at the top of `<workspace>/`.** That top
folder is a plain container — it has no `.git` of its own, so anything dropped directly into it
(including the `workspace-root/` copy above) is never committed or shared. Putting the workspace
file in `athenaeum-docs/` instead means it's versioned and arrives automatically for anyone who
clones that repo. See [03 — Structure](../docs/03-structure.md#the-workspace-on-disk) for the
diagram.

**Using Claude Code?** Two more steps, both in
[06 — Claude Code setup](../docs/06-claude-code-setup.md): symlink the skills and sub-agents into
the container so `/feature-*` resolves from wherever you launch, and make `CLAUDE.md` a one-line
`@AGENTS.md` import rather than a second operating model.

```bash
# from <workspace>/ — the container has no .git, so this is per-developer
mkdir -p .claude
ln -s ../athenaeum-docs/.claude/skills .claude/skills
ln -s ../athenaeum-docs/.claude/agents .claude/agents
```

**Spec Kit is NOT in this kit** — install it per code repo with
`specify init . --integration copilot`. See [05 — Setup](../docs/05-copilot-setup.md).

---

## The 11 stages

In `docs-repo/.github/agents/`. They live in the **docs repo** because that's the coordination
layer — their whole job is to reach *down* into the code repos.

**A stage has one definition and two front-ends.** The `.agent.md` file *is* the stage. Copilot
reads it natively as `@feature.plan`; Claude Code reads the same file as `/feature-plan`, via an
eleven-skill wrapper layer in `docs-repo/.claude/` that holds no rules of its own. Neither runner
owns a stage's rules — if you find yourself fixing a wrapper, fix the `.agent.md` instead and both
runners get it.

| Agent | Stage | Role | Gate? |
|---|---|---|---|
| `feature.specify` | 2 | Feature Manager | |
| `feature.spec-signoff` | 3 | **Customer** | 🚦 |
| `feature.plan` | 4 | Tech Lead | |
| `feature.tasks` | 5 | Tech Lead | |
| `feature.analyze` | 5a | Tech Lead | |
| `feature.implement` | 6 | Developer | |
| `feature.verify` | 7 | **QA** | 🚦 |
| `feature.signoff` | 8 | **QA** | 🚦 |
| `feature.uat` | 9 | **Customer** | 🚦 |
| `feature.ship` | 10 | Developer | |
| `feature.adopt` | — | Tech Lead | *In-flight entry point. See below.* |

### Already have work in flight?

You almost certainly do. **`feature.adopt` is your real on-ramp** — it detects what each repo has
already got (spec? plan? tasks? merged code?) and wraps an existing feature in a manifest that
reflects reality, without pretending it started greenfield.

It will not fabricate a `spec_signoff` for a feature that predates the gate — and neither should
you. Either leave it null with a logged warning, or take the reconstructed spec to the customer now
and get a real signature for the remaining stories.

---

## The manifest is the spine

`docs-repo/features/_template/manifest.yaml` — **read it before anything else here.** It's heavily
commented, and its inline `[stage]` annotations are the authoritative answer to "which stage owns
this field?"

Copy it to `<product>-docs/features/NNN-slug/` per feature. The agents read and mutate it; it's how
the pipeline stays mechanical instead of remembered.

---

## The three docs that do the daily work

| File | Do this |
|---|---|
| **`glossary.md`** | **Fill this in FIRST, as a team, before your first spec.** Most spec-driven development failures are vocabulary failures. |
| `feature-catalog.md` | Your backlog. Has the completeness bar `@feature.specify` enforces, plus a worked example and a full example backlog. |
| `decisions.md` | **The reason the docs repo exists.** Append-only — supersede, never delete. Five worked examples including a superseded pair; delete them once you have real ones. |

`docs-repo/README.md` ships too — it carries the reading order and explains the four kinds of file.
That's where reading order belongs, rather than encoded in filename prefixes: **number the things you
have many of, name the things you have one of.** There's one glossary, so it's `glossary.md`. There
are many features, so they're `features/015-hold-queue/` — and *that* number is a real identifier,
shared across both repos' spec folders, both branch names, and the `@015-us1` test tag. See
[03 — Structure](../docs/03-structure.md#why-the-docs-have-no-numbers-and-the-features-do).

The other four aren't templated — they're too project-specific to fake usefully. Create them as plain
markdown, but know that two of them the **agents will actually read**:

- **`personas-and-jobs.md`** and **`product-principles.md`** — a referenced persona and a
  linked principle get assembled into every spec *verbatim*. Write these properly; they end up in
  generated output.
- **`vision.md`** and **`walkthroughs.md`** — humans only. Useful for onboarding, but no agent
  loads them, so a beautiful vision statement won't improve what the model generates.

**Seven docs, and nothing else.** There's no `current-state`, `target-state`, or `roadmap` here on
purpose. Nothing in the pipeline reads them, and **a document nothing references is a document nobody
updates** — a stale doc in a source-of-truth repo is worse than no doc at all. Add one the day
something actually needs it.

---

## How the agents are wired

Two frontmatter fields carry the whole architecture:

**`handoffs`** — the pipeline chain. Each stage suggests the next with a pre-filled prompt, so a new
team member can follow the process from inside VS Code without reading this repo.

**`agents`** — per-repo dispatch. An orchestrator sends work *down* into a repo (`repo-planner`,
`repo-implementer`, …) where it runs against that repo's own constitution, then reconciles what
comes back. That's the up/down rule from [01](../docs/01-why.md) in two lines of YAML.

All six sub-agents ship, **for both front-ends**:

| Sub-agent | Dispatched by | Writes |
|---|---|---|
| `repo-scaffolder` | `feature.specify` | the branch + an empty `spec.md` |
| `repo-detector` | `feature.adopt` | **nothing — read-only** |
| `repo-planner` | `feature.plan` | `plan.md`, `research.md`, `data-model.md`, `contracts/` |
| `repo-tasker` | `feature.tasks` | `tasks.md` (GENERATE mode only) |
| `repo-implementer` | `feature.implement` | `src/` — the only one that does |
| `repo-auditor` | `feature.analyze` | **nothing — read-only** |

Copilot reads `docs-repo/.github/agents/repo-*.agent.md`; Claude Code reads
`docs-repo/.claude/agents/repo-*.md`. Each carries `user-invocable: false` so it stays out of the
dropdown — they're pipeline machinery, dispatched by the stage that owns them, never picked by
hand.

They're the most project-shaped files in the kit, so read them before your first real feature and
adjust them to your stack. **The write scopes are the part to leave alone** — `repo-detector` and
`repo-auditor` are read-only for the same reason `feature.verify` is.

### Two front-ends, one body

The `feature.*` stages have **one** definition and two front-ends: the `.agent.md` is the stage,
and Claude Code's skills read that same file rather than restating it. Nothing is duplicated.

**The `repo-*` sub-agents are the one exception.** They exist twice — same body, byte for byte,
with different frontmatter — because a sub-agent's body *is* its prompt, and there's no
shared-include mechanism both runners read. Each file carries an HTML comment naming its twin.

**Most teams should delete the front-end they don't use**, and then this exception evaporates:

```bash
rm -rf <product>-docs/.claude                      # Copilot-only shop
rm  <product>-docs/.github/agents/repo-*.agent.md  # Claude-Code-only shop
```

If you genuinely run both, editing one and not the other is the failure mode to watch for. It's
the same hazard [08 — Pitfalls](../docs/08-pitfalls.md) describes everywhere else; here it's
accepted deliberately rather than by drift.

---

## Before you trust it

The write-scope rules in each agent are **load-bearing, not ceremony**. `feature.verify` cannot
write `src/` because *a verifier that can fix things will fix things*, and then you have no gate.
`feature.analyze` can write exactly one log line because an auditor that edits is not an auditor.

Two things to keep in mind:

- **These are prompts, not programs.** They'll be right for a month and then confidently do
  something wrong. That's why every write scope is also stated as an explicit **Forbidden** list in
  the body, and why every gate is a human. Don't remove either safeguard because it's never fired —
  that's the moment you find out what it was catching.
- **Tune the `tools` list to your setup.** The lists here are a starting point. Narrower is safer.

---

## Adapt these

Everything here is a starting point, not scripture. The parts worth keeping as-is:

- The **two-signature model** — the customer signs the spec, QA signs the build.
- The **two-bucket UAT triage** — bugs come back, ideas go to the catalog.
- The **`state` dispatcher** and "the merged contract wins".
- **Never lose a task** and the completeness arithmetic.
- **Green is a fact; signed-off is a judgement.**

The parts that are yours to change: the number of repos, the `compose` recipe, the ID conventions,
the tools lists, how many stages one person covers.

If you drop something, drop it deliberately — [08 — Pitfalls](../docs/08-pitfalls.md) says what each
rule was protecting you from.
