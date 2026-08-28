# `<product>`

This folder is a plain container, not a git repo — it just holds three sibling repos together on
disk for convenience. Nothing placed directly in this folder is ever committed anywhere; the
canonical copies of `README.md`, `AGENTS.md` and `CLAUDE.md` live in
[`<product>-docs/`](../docs-repo/README.md), and these top-level files are a local copy kept in
sync by hand. If that sounds fragile, it's because it is slightly — see `AGENTS.md` in this same
folder for the tradeoff and the alternative.

**Start in [`<product>-docs/`](../docs-repo/README.md).** It's the coordination hub — specs, the
glossary, decisions log, and feature backlog — and it holds the multi-root workspace file
(`<product>-docs/<product>.code-workspace`). Open that file in VS Code, not this folder directly,
so Copilot/agents can see all three repos at once.

- [`<product>-docs/`](../docs-repo/README.md) — coordination repo. **Read its `README.md` first.**
- `<product>-api/` — API repo
- `<product>-ui/` — UI repo

**Running Claude Code?** Launch it from *this* folder — a session that can only see one repo
cannot reconcile a contract across the seam. The `/feature-*` stage commands live in
`<product>-docs/.claude/`, so they only resolve here once you've made the two symlinks:

```bash
mkdir -p .claude
ln -s ../<product>-docs/.claude/skills .claude/skills
ln -s ../<product>-docs/.claude/agents .claude/agents
```

This folder has no `.git`, so those are yours alone — every developer makes them once.

**Agents/assistants:** if you're opening this folder directly rather than the workspace file, read
`AGENTS.md` in this same folder before making changes anywhere in this tree, then treat
`<product>-docs/` as the source of truth for specs, vocabulary, and decisions.
