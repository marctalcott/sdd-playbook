# Starter kit

Copy-paste implementation of the [playbook](../README.md). Everything here is generic — replace
`<product>` with your own name.

**Read [02 — The pipeline](../docs/02-pipeline.md) first.** These files won't make sense as a
process until you know what the ten stages are.

---

## What goes where

| Folder | Copy into | Contains |
|---|---|---|
| **`docs-repo/`** | `<product>-docs/` | The 11 agents, the manifest template, the three docs that do the daily work |
| **`code-repo/`** | **each** code repo | The Copilot instructions template |
| **`example.code-workspace`** | `<workspace>/<product>.code-workspace` | The multi-root workspace — **the file that lets Copilot see across repos** |

```bash
# from <workspace>/, with acme-docs, acme-api, acme-ui as siblings
cp -r sdd-playbook/starter-kit/docs-repo/.       acme-docs/
cp -r sdd-playbook/starter-kit/code-repo/.       acme-api/
cp -r sdd-playbook/starter-kit/code-repo/.       acme-ui/
cp    sdd-playbook/starter-kit/example.code-workspace  acme.code-workspace
```

Then find-and-replace `<product>` → `acme` and open `acme.code-workspace`.

**Spec Kit is NOT in this kit** — install it per code repo with
`specify init . --integration copilot`. See [05 — Setup](../docs/05-copilot-setup.md).

---

## The 11 agents

In `docs-repo/.github/agents/`. They live in the **docs repo** because that's the coordination
layer — their whole job is to reach *down* into the code repos.

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
| **`08-glossary.md`** | **Fill this in FIRST, as a team, before your first spec.** Most spec-driven development failures are vocabulary failures. |
| `07-feature-catalog.md` | Your backlog. Has the completeness bar `@feature.specify` enforces, and a worked example. |
| `09-decisions.md` | Append-only. Two worked examples — delete them once you have real ones. |

The other docs from [03 — Structure](../docs/03-structure.md) (`01-vision`, `02-personas-and-jobs`,
`03-product-principles`, and so on) aren't templated here — they're too project-specific to fake
usefully. Create them as plain markdown; the numbering is the only convention that matters.

---

## How the agents are wired

Two frontmatter fields carry the whole architecture:

**`handoffs`** — the pipeline chain. Each stage suggests the next with a pre-filled prompt, so a new
team member can follow the process from inside VS Code without reading this repo.

**`agents`** — per-repo dispatch. An orchestrator sends work *down* into a repo (`repo-planner`,
`repo-implementer`, …) where it runs against that repo's own constitution, then reconciles what
comes back. That's the up/down rule from [01](../docs/01-why.md) in two lines of YAML.

You need to **define the sub-agents** (`repo-planner`, `repo-scaffolder`, `repo-implementer`,
`repo-tasker`, `repo-auditor`, `repo-detector`) for your own stack, since what they do is
project-specific. Give each `user-invocable: false` so they stay out of the dropdown. Start simple:
a sub-agent that reads the repo's constitution and runs the matching `/speckit.*` command is enough
to begin with.

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

If you drop something, drop it deliberately — [07 — Pitfalls](../docs/07-pitfalls.md) says what each
rule was protecting you from.
