# 06 — Claude Code setup

> **Read this if** you want to run the same pipeline from [Claude Code](https://claude.com/claude-code)
> instead of — or alongside — VS Code and Copilot. It assumes you have already done
> [05 — VS Code + Copilot setup](05-copilot-setup.md) through step 4: Spec Kit installed per code
> repo, a constitution in each, and the docs repo seeded.

---

## The one idea to hold on to

**A stage has one definition and two front-ends.**

The definition is `<product>-docs/.github/agents/feature.<stage>.agent.md`. That file *is* the
stage. Claude Code does not get its own copy of the rules — it gets an eleven-skill front-end
whose entire job is to read that same file and execute it.

```
                  .github/agents/feature.plan.agent.md
                        the stage — one definition
                          ▲                  ▲
                          │                  │
              reads it natively        reads it via
                          │           .claude/pipeline-runner.md
                          │                  │
                  @feature.plan        /feature-plan
                  GitHub Copilot        Claude Code
```

This is not neatness for its own sake. The rule this playbook repeats everywhere — *two copies of
a rule become two different rules within a month* — applies hardest here, because a stage's rules
**are** its write scopes and its gates. A Claude front-end that restated them would drift, and the
drift would show up as a gate that stopped holding.

So: **if a wrapper starts accumulating instructions of its own, that is the bug.** Fix the
`.agent.md` and both runners get it.

---

## What you need

| | |
|---|---|
| **Claude Code** | Any recent version. CLI, desktop app, or the VS Code / JetBrains extension — all read the same project files. |
| **The Spec Kit CLI** | Same install as [05](05-copilot-setup.md#step-1--install-the-spec-kit-cli). Per **code** repo, not the docs repo. |
| **A constitution per code repo** | [05, step 4](05-copilot-setup.md#step-4--write-each-repos-constitution). The sub-agents build strictly inside it, and without one you get generic code. |

You do **not** need Copilot. The `.agent.md` files are the stage definitions whether or not
anything ever reads them as Copilot agents.

---

## Step 1 — Copy the `.claude/` layer into the docs repo

```bash
cp -r sdd-playbook/starter-kit/docs-repo/.claude  <product>-docs/
```

That gives you three things:

```
<product>-docs/.claude/
├── pipeline-runner.md              ← the ONLY Copilot→Claude translation layer
├── skills/
│   ├── feature-specify/SKILL.md    ← 11 thin wrappers, one per stage
│   ├── feature-plan/SKILL.md
│   └── …
└── agents/
    ├── repo-scaffolder.md          ← 6 per-repo sub-agents
    ├── repo-planner.md
    └── …
```

Then find-and-replace `<product>` as you did for the rest of the starter kit.

**It goes in the docs repo, for the same reason the agents and the workspace file do** — the docs
repo is the coordination layer, it is git-tracked, and it arrives automatically for anyone who
clones it. A `.claude/` folder dropped loose in the container is on one person's laptop and
nowhere else.

---

## Step 2 — Make the skills resolve from wherever you launch

**Here's the problem.** Claude Code discovers project skills and sub-agents from `.claude/` in the
directory you launch it in. Your `.claude/` is inside `<product>-docs/`. The container folder — the
one holding all the repos as siblings, the one you actually want to be in so the model can see
across the seam — has no `.claude/` at all, so `/feature-plan` simply won't exist there.

**The fix is a symlink per subfolder**, not a symlink of the whole `.claude/`:

```bash
cd <workspace>                       # the plain container holding the repos as siblings
mkdir -p .claude
ln -s ../<product>-docs/.claude/skills .claude/skills
ln -s ../<product>-docs/.claude/agents .claude/agents
```

Symlinking the two subfolders rather than `.claude` itself leaves room for a real, untracked
`.claude/settings.local.json` beside them — that file holds machine-specific permissions and
belongs to one developer, not to the repo.

> **The container has no `.git`, so these symlinks are not shared.** Like the `README.md` and
> `AGENTS.md` that live there (see [03 — Structure](03-structure.md#the-container-folder-isnt-entirely-bare-though)),
> each person creates them once. Put the two `ln -s` lines in your docs repo's README so the next
> person doesn't have to work out why `/feature-specify` isn't there.

**Launching from inside a code repo still won't see them.** That's a feature, not a bug: a stage
that can only see one repo cannot reconcile a contract across the seam, and will silently do half
a job. Launch from `<workspace>`.

---

## Step 3 — Point `CLAUDE.md` at `AGENTS.md`

Claude Code reads `CLAUDE.md`, not `AGENTS.md`. Do **not** solve that by keeping two operating-model
documents. Make `CLAUDE.md` a one-line import and nothing else:

```markdown
@AGENTS.md
```

Put that pair — the real `AGENTS.md` from
[`starter-kit/workspace-root/`](../starter-kit/workspace-root/) and its one-line `CLAUDE.md` — in
**both** the docs repo and the container folder.

**Why both.** Claude Code loads `CLAUDE.md` from the working directory *and every directory above
it*. A copy at the container root is therefore in context no matter which of your repos you launch
from, and each repo's own `CLAUDE.md` then layers on top. Workspace-wide rules go in the container
copy; repo-specific rules go in that repo's own file.

---

## Step 4 — Read `pipeline-runner.md` once, yourself

It's 100 lines and it is the only thing standing between "Copilot frontmatter" and "Claude Code
behaviour". Four of its translations are worth knowing before you trust a run:

**`tools:` is the write scope.** `tools: ['codebase', 'search']` means read-only — Read, Glob,
Grep, and nothing else. Adding `editFiles` grants Edit and Write, `runCommands` grants Bash,
`agent` grants sub-agent dispatch. **The absence of a tool is a rule**, and the prose in the
stage's own *Operating constraints* is narrower still and wins over the frontmatter.

**"In a single message" means parallel.** Where a stage says to dispatch sub-agents in one
message, that is a real instruction — separate messages run them in sequence and waste the
parallelism. `feature.implement` says the opposite on purpose: **api first, awaited, then ui.**
Parallelising across the seam produces a rebuild, not a slice.

**Handoffs are printed, not taken.** This is the biggest behavioural difference from Copilot.
Copilot chains stages via `handoffs:`. Here the stage finishes, emits its Output block, prints the
next command as a suggestion — and **stops**:

```
Next: /feature-spec-signoff 015-hold-queue
```

Running the next stage yourself collapses the gates the pipeline exists to enforce. Four of those
gates are a named human saying yes. An agent that walks through them has removed the only part of
this process that isn't automatable.

**`status: blocked` is a real, successful outcome.** A stage that cannot proceed says so and stops.
It does not improvise past a missing precondition.

---

## Step 5 — The six sub-agents

`.claude/agents/` ships `repo-scaffolder`, `repo-detector`, `repo-planner`, `repo-tasker`,
`repo-implementer` and `repo-auditor`. Each is the per-repo worker that one stage fans out to:

| Sub-agent | Dispatched by | Does |
|---|---|---|
| `repo-scaffolder` | `feature.specify` | Cuts the branch, creates the spec folder. Container only. |
| `repo-detector` | `feature.adopt` | **Read-only.** Reports what a repo actually contains. |
| `repo-planner` | `feature.plan` | `plan.md`, `research.md`, `data-model.md`, `contracts/`. |
| `repo-tasker` | `feature.tasks` | Generates or maps `tasks.md`. |
| `repo-implementer` | `feature.implement` | The only one that writes `src/`. |
| `repo-auditor` | `feature.analyze` | **Read-only.** One analysis dimension, findings routed to the owning stage. |

**These are pipeline machinery, not general-purpose helpers.** Dispatch one only from the stage
that owns it. Spawning `repo-implementer` directly because you want some code written does not
satisfy a stage — it just writes code with none of the gates around it.

They are the most project-shaped files in the kit. Read them before your first real feature and
adjust the *Read first* and *Forbidden* sections to your stack; the write scopes are the part to
leave alone.

> **These six exist twice, once per front-end** — `.claude/agents/repo-*.md` here, and
> `.github/agents/repo-*.agent.md` for Copilot. The **bodies are byte-identical**; only the
> frontmatter differs, because the two runners name their tools differently. Each file carries an
> HTML comment naming its twin.
>
> This is the one place the kit ships two copies of the same words, and it is a deliberate
> exception to *two copies of a rule become two different rules* — a sub-agent's body **is** its
> prompt, and there is no shared-include mechanism both runners read. **Most teams should delete
> the front-end they don't use.** If you genuinely run both, editing one and not the other is the
> failure mode to watch for.

> **Your session may not be allowed to dispatch them.** Claude Code can be configured to withhold
> sub-agent dispatch unless you ask for it explicitly. When that's set, it wins over anything a
> markdown file says — a file cannot grant a session a capability its own configuration withholds.
> The stage should ask you rather than assume, and you say yes. Expect to be asked each session.

---

## Step 6 — Check the gates survived

Every skill ships with:

```yaml
user-invocable: true
disable-model-invocation: true
```

**That second line is load-bearing.** It means a stage runs because a human typed `/feature-plan`,
never because an assistant read the room and inferred that planning was next. It is the same
safeguard as `tools:`, aimed at a different failure: not *"the agent wrote something it shouldn't"*
but *"the agent advanced the pipeline without being asked."*

Leave it on. The moment a model can invoke `feature.spec-signoff`, Gate 1 is decorative.

---

## The finished layout

```
~/src/<product>/                          ← plain container, no .git of its own
├── README.md, CLAUDE.md, AGENTS.md       ← local copies; canonical versions are below
├── .claude/
│   ├── skills   → ../<product>-docs/.claude/skills    ← symlink
│   ├── agents   → ../<product>-docs/.claude/agents    ← symlink
│   └── settings.local.json               ← yours, untracked, machine-specific
│
├── <product>-docs/
│   ├── <product>.code-workspace          ← open THIS in VS Code
│   ├── CLAUDE.md                         ← one line: @AGENTS.md
│   ├── AGENTS.md                         ← canonical operating model, git-tracked
│   ├── .github/agents/                   ← the 11 feature.* stage definitions
│   │   └── repo-*.agent.md               ←   + the 6 sub-agents, Copilot front-end
│   ├── .claude/
│   │   ├── pipeline-runner.md            ← the translation layer
│   │   ├── skills/feature-*/SKILL.md     ← 11 thin wrappers
│   │   └── agents/repo-*.md              ←   the same 6, Claude front-end (delete one set)
│   └── features/NNN-slug/{feature.md, manifest.yaml}
│
├── <product>-api/    .specify/, .github/, src/
└── <product>-ui/     .specify/, .github/, e2e/, src/
```

---

## Check it works

```bash
cd ~/src/<product> && claude
```

- [ ] Typing `/feature-` lists the eleven stages.
- [ ] `/feature-specify` **without** typing it — ask *"what's the next stage for feature NNN?"* and
      confirm it **tells you the command rather than running it.** If it ran, check
      `disable-model-invocation`.
- [ ] Ask a question needing two repos at once — *"list the endpoint paths in `<product>-api/src`
      and every path `<product>-ui` calls"*. If it can only see one, you launched from the wrong
      folder.
- [ ] Run a real stage on a real catalog row and read what it produced.
- [ ] The generated spec uses **your glossary's words.** If it invented a synonym, your glossary is
      thin or the stage isn't reading it — fix that now, not later.

That last one is the real test, and it is the same last one as in
[05](05-copilot-setup.md#check-it-works). Everything else is plumbing.

---

## Honest notes on the tooling

- **Spec Kit's helper scripts resolve the active feature from a file, not from your branch.**
  `check-prerequisites.sh` reads `.specify/feature.json`. A repo where that file is stale points
  every `/speckit.*` command at the wrong feature's spec folder, and a repo without one errors out.
  Neither failure is loud. Pass the directory explicitly and stop guessing:

  ```bash
  SPECIFY_FEATURE_DIRECTORY=.specify/specs/NNN-slug-api \
    .specify/scripts/bash/check-prerequisites.sh --json --paths-only
  ```

- **A nested app is two paths, and they are easy to mix up.** If a repo keeps its application in a
  subfolder, the branch is still cut at the git root while `package.json`, `.specify/` and `e2e/`
  live one level down. The manifest records both (`path` and `app_root`) and is the authority —
  read it rather than inferring from what you find on disk.

- **The skills are prompts, not programs.** Same caveat as the agents in
  [05](05-copilot-setup.md#honest-notes-on-the-tooling). They'll be right for a month and then
  confidently do the wrong thing. That's why the write scopes are also stated as an explicit
  **Forbidden** list in each stage body, and why every gate is a human. Don't remove either
  safeguard because it's never fired.

- **Running both front-ends is fine, and the failure mode is social, not technical.** Nothing
  breaks if one person uses Copilot and another uses Claude Code on the same feature — they read
  the same `.agent.md` and mutate the same manifest. What breaks is two people running the same
  stage at once. The manifest's `log` is append-only and will show you both.

---

Next: [07 — Rollout](07-rollout.md)
