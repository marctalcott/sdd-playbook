# 05 — VS Code + Copilot setup

> **Read this if** you're the person actually building this. Everything here uses only VS Code and
> GitHub Copilot. No other tooling, no subscriptions beyond Copilot.

---

## What you need

| | |
|---|---|
| **VS Code** | Current release. Custom agents are a recent feature — if the agents dropdown isn't in your Chat view, update. |
| **GitHub Copilot** | Any plan with Copilot Chat. |
| **The Spec Kit CLI** | Installed once per machine, below. |
| **Git** | One repo per code project, one for the docs. |

---

## Step 1 — Install the Spec Kit CLI

Spec Kit ships as a Python CLI called `specify`. The current install path is via
[`uv`](https://docs.astral.sh/uv/):

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

Or run it without installing:

```bash
uvx --from git+https://github.com/github/spec-kit.git specify init --help
```

> **Check the flags yourself.** Spec Kit is moving quickly and its CLI flags have already changed
> once — `--ai copilot` was renamed to `--integration copilot`, with the old flag deprecated rather
> than removed. Run `specify init --help` and trust that over any document, including this one. The
> [Spec Kit repo](https://github.com/github/spec-kit) is the authority.

---

## Step 2 — Lay out the workspace

Repos as siblings. Flat.

```bash
mkdir -p ~/src/acme && cd ~/src/acme
git init acme-docs
git init acme-api
git init acme-ui
```

---

## Step 3 — Install Spec Kit into each **code** repo

Once per code repo. **Not** in the docs repo — the docs repo holds no code and generates no plans.

```bash
cd ~/src/acme/acme-api
specify init . --integration copilot
```

That creates:

```
acme-api/
├── .specify/
│   ├── memory/constitution.md          ← this repo's rules (empty for now)
│   ├── templates/
│   │   ├── spec-template.md
│   │   ├── plan-template.md
│   │   └── tasks-template.md
│   └── scripts/bash/
└── .github/
    ├── copilot-instructions.md         ← Copilot's context file for this repo
    └── prompts/                        ← the /speckit.* commands land HERE
        ├── speckit.specify.prompt.md
        ├── speckit.plan.prompt.md
        └── …
```

Repeat for `acme-ui`.

**Verify it worked:** open the repo in VS Code, open Copilot Chat, type `/speckit.` and you should see
the command list. If you don't, the prompts folder didn't land — check that `.github/prompts/` exists
and that you're not in a subfolder of the repo.

### The Spec Kit commands you now have

| Command | Used at | By |
|---|---|---|
| `/speckit.constitution` | Once per repo, at setup | Tech Lead |
| `/speckit.specify` | Stage 2 | Feature Manager |
| `/speckit.clarify` | Stage 2 | Feature Manager |
| `/speckit.plan` | Stage 4 | Tech Lead |
| `/speckit.tasks` | Stage 5 | Tech Lead |
| `/speckit.analyze` | Stage 5a | Tech Lead |
| `/speckit.checklist` | Stage 7 | QA |
| `/speckit.implement` | Stage 6 | Developer |

These are **per-repo**. The `@feature.*` agents you'll build in step 6 sit *above* them and call down
into them. That's the up/down rule from [01](01-why.md) made concrete.

---

## Step 4 — Write each repo's constitution

```
/speckit.constitution
```

This is the repo's law and it's worth an hour of the team's time. Be specific and be strict — vague
principles generate vague code. Real examples:

**API repo:**
> Money is stored and transmitted as integer cents, never a decimal or float (D-011).
> Business values — thresholds, caps, fees, windows — are read from configuration by name.
> Never inline a literal.
> Every endpoint has a contract test before it has an implementation.
> Domain names come from `08-glossary.md`. No synonyms.

**UI repo:**
> Components call the real API. Never a mock, never a stub, never fixture data outside of tests.
> All components meet WCAG AA.
> Use the design system's components. Do not hand-roll a button.
> Config values come from the config hook, never from a literal.

The constitution is what lets you push implementation *down* into the repo and trust the result.
Without it, generated code drifts toward whatever the model saw most on the internet.

---

## Step 5 — The multi-root workspace (this is the important one)

**Here's the problem.** Copilot sees your workspace. If your workspace is one repo, Copilot cannot see
the other repo — which means it cannot reconcile a contract, cannot project a spec into two repos, and
cannot run a test in one repo against a service in another. Every cross-repo thing this playbook does
would be impossible.

**The fix is a multi-root workspace.** Create `~/src/acme/acme.code-workspace`:

```jsonc
{
  "folders": [
    { "name": "docs", "path": "acme-docs" },
    { "name": "api",  "path": "acme-api"  },
    { "name": "ui",   "path": "acme-ui"   }
  ],
  "settings": {
    // Make the feature.* agents visible regardless of which folder is focused.
    "chat.agentFilesLocations": {
      "acme-docs/.github/agents": true
    }
  }
}
```

Open **that file**, not the individual folders (`File → Open Workspace from File…`). Now Copilot can
see all three repos at once, and `@feature.plan` can genuinely compare the API's contract against the
UI's.

> **Sanity-check this.** In a multi-root workspace, ask Copilot Chat something that requires two repos
> — *"list the endpoint paths in acme-api/src and every path acme-ui calls"*. If it can only see one,
> your agents will silently do half a job. Fix this before you build anything on top of it.

---

## Step 6 — The `feature.*` custom agents

This is the coordination layer. VS Code calls them **custom agents** (they were called *custom chat
modes* until recently — if you find `.chatmode.md` files in a blog post, that's the old name for the
same thing).

### Where they go

```
acme-docs/.github/agents/
├── feature.specify.agent.md
├── feature.spec-signoff.agent.md
├── feature.plan.agent.md
├── feature.tasks.agent.md
├── feature.analyze.agent.md
├── feature.implement.agent.md
├── feature.verify.agent.md
├── feature.signoff.agent.md
├── feature.uat.agent.md
├── feature.ship.agent.md
└── feature.adopt.agent.md
```

They live in the **docs repo** because the docs repo is the coordination layer — that's where
`features/NNN-slug/` lives, and these agents' whole job is to reach *down* into the code repos.

VS Code discovers `.agent.md` files in `.github/agents/` and also in `.claude/agents/`. The
`chat.agentFilesLocations` setting in your workspace file makes them visible from any folder.

**Copy them from [`starter-kit/docs-repo/.github/agents/`](../starter-kit/docs-repo/.github/agents/)
— they're written and ready.**

Each orchestrator dispatches **sub-agents** (`repo-planner`, `repo-implementer`, …) down into the
code repos. Those are project-specific, so you define them yourself — see the
[starter kit README](../starter-kit/README.md#how-the-agents-are-wired). Give each
`user-invocable: false` so they stay out of the dropdown.

### The file format

```markdown
---
name: feature.plan
description: Feature-level Plan orchestrator (Tech Lead). Turns a signed spec into
  per-repo technical artifacts, and reconciles contracts across the repo seam.
tools: ['codebase', 'search', 'editFiles', 'runCommands', 'agent']
agents: ['repo-planner']
handoffs:
  - agent: feature.tasks
    prompt: "Slice the tasks for this feature into per-story slices."
---

## Goal
You are the Tech Lead. …

## Operating Constraints
**Write scope (CRITICAL).** You may write ONLY: …
**Forbidden — STOP if you are about to:** …

## Execution Steps
### 1. Resolve the workspace
…
```

Frontmatter fields worth knowing:

| Field | What it does |
|---|---|
| `name` | The agent's identifier. Defaults to the filename. |
| `description` | Shown in the dropdown. Write it so a human knows when to pick it. |
| `tools` | Which tools it may use. **This is how you enforce write scope.** |
| `model` | Pin a model if you want. Usually omit. |
| `agents` | Sub-agents this agent may dispatch. Needs `agent` in `tools`. |
| `handoffs` | Suggested next agents, with a pre-filled prompt. **This is your pipeline chain.** |
| `user-invocable` | `false` hides it from the dropdown (for sub-agents only). |
| `disable-model-invocation` | `true` stops other agents using it as a sub-agent. |

### Two frontmatter fields that map onto the pipeline exactly

**`handoffs` is your pipeline.** Each stage suggests the next one with a pre-filled prompt, so the
process is discoverable from inside the tool rather than from this document. `feature.specify` hands
off to `feature.spec-signoff`, which hands off to `feature.plan`, and so on. A new team member can
follow the chain without knowing it exists.

**`agents` is your per-repo dispatch.** The up/down rule needs an orchestrator that can send work down
into a repo. Declaring a sub-agent and including `agent` in `tools` is how: `feature.plan` dispatches
a planner into each repo, each working inside that repo's own constitution, then reconciles what comes
back. That's the whole architecture in two frontmatter fields.

### Write scope is enforced by `tools`, and it matters

Look at what each agent is allowed to touch:

| Agent | Can write | Cannot write |
|---|---|---|
| `feature.specify` | `features/`, per-repo `spec.md` | `plan.md`, `tasks.md`, any `src/` |
| `feature.plan` | `plan.md`, `data-model.md`, `contracts/` | `spec.md`, `tasks.md`, any `src/` |
| `feature.tasks` | `tasks.md`, the manifest's slices | any `src/` |
| `feature.implement` | `src/`, tests, task checkboxes | the manifest's gate fields |
| `feature.verify` | the manifest's `done_gate` | **any `src/`** — it observes, never repairs |
| `feature.signoff` | the manifest's `signoff` | **any `src/`** |
| `feature.analyze` | one log line | **everything else** — it's an auditor |

These aren't arbitrary. `feature.verify` can't write code *because a verifier that can fix things
will fix things*, and then you have no gate. Encode the boundary in `tools`, and state it in the body
under **Forbidden**, so it holds even when a model is feeling helpful.

---

## Step 7 — Custom instructions

Two mechanisms, different jobs:

**`.github/copilot-instructions.md`** — applies to everything in that repo. Keep it short and point at
the real authority:

```markdown
# acme-api

Backend for Acme. Event-sourced, .NET.

## Non-negotiable
- Domain vocabulary comes from `../acme-docs/08-glossary.md`. Never invent a synonym.
- Money is integer cents (D-011). Never decimal.
- Business values come from config by name. Never a literal.
- The full rules are in `.specify/memory/constitution.md`. Read it before writing code.

## Structure
- `src/Api/` — endpoints
- `src/Domain/` — aggregates, events
- `.specify/specs/NNN-slug-api/` — the spec for feature NNN
```

**`*.instructions.md` with `applyTo`** — path-scoped rules:

```markdown
---
applyTo: "src/Domain/**/*.cs"
---
Aggregates never call out to infrastructure. All state changes are events.
Event names are past tense and match `08-glossary.md` exactly.
```

Rule of thumb: **`copilot-instructions.md` for facts about the repo, `.instructions.md` for rules about
a folder, and the constitution for the law.** Don't duplicate the constitution into
`copilot-instructions.md` — point at it. Two copies of a rule become two different rules.

---

## Step 8 — Seed the docs repo

Copy [`starter-kit/docs-repo/`](../starter-kit/docs-repo/) into `acme-docs/` and fill it in.
Start with:

1. **`08-glossary.md`** — do this first, and do it as a team. Every term you'll use. This is the
   single highest-value hour you will spend, and doing it late means renaming things later.
2. **`03-product-principles.md`** — 5 to 10 principles, numbered `P-01`…
3. **`02-personas-and-jobs.md`** — who these people are.
4. **`09-decisions.md`** — start empty. Add `D-001` the first time you decide something.
5. **`07-feature-catalog.md`** — your first feature row.
6. **`features/_template/manifest.yaml`** — copy it as-is.

---

## The finished layout

```
~/src/acme/
├── acme.code-workspace              ← open THIS
│
├── acme-docs/
│   ├── .github/agents/              ← the 11 feature.* agents
│   ├── 01-vision.md … 11-walkthroughs.md
│   └── features/
│       ├── _template/manifest.yaml
│       └── 015-watch-list/{feature.md, manifest.yaml}
│
├── acme-api/
│   ├── .specify/{memory,templates,specs,scripts}/
│   ├── .github/{copilot-instructions.md, prompts/}
│   └── src/
│
└── acme-ui/
    ├── .specify/{memory,templates,specs,scripts}/
    ├── .github/{copilot-instructions.md, prompts/}
    ├── e2e/
    └── src/
```

---

## Check it works

Run through this before you trust it with a real feature:

- [ ] Opening `acme.code-workspace` shows all three folders in the Explorer.
- [ ] Copilot Chat can answer a question that requires two repos at once.
- [ ] `/speckit.` autocompletes in Chat when a code repo file is focused.
- [ ] The agents dropdown lists your `feature.*` agents.
- [ ] `@feature.specify` on a real catalog row produces a `features/NNN-slug/` folder.
- [ ] The generated spec uses **your glossary's words**. If it invented a synonym, your glossary is
      thin or the agent isn't reading it — fix that now, not later.

That last one is the real test. Everything else is plumbing.

---

## Honest notes on the tooling

- **Spec Kit is young and moves.** Commands and flags have changed and will change again. Pin a
  version if you need reproducibility across a team, and check `--help` over any doc.
- **"Custom agents" were "custom chat modes" until recently.** Same feature, new name and extension
  (`.chatmode.md` → `.agent.md`). Blog posts are mostly still using the old name.
- **Multi-root agent discovery is the fragile bit.** It's the newest thing here and the thing most
  likely to behave differently in your VS Code version. If the dropdown is missing agents, the
  `chat.agentFilesLocations` setting is your lever. Verify it explicitly rather than assuming.
- **The agents are prompts, not programs.** They will occasionally do the wrong thing. That's exactly
  why every write scope is stated as an explicit **Forbidden** list in the body, and why every gate is
  a human. Don't remove either safeguard because it's been fine for a month.

---

Next: [06 — Rollout](06-rollout.md)
