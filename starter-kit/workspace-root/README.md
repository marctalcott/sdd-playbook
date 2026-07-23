# `<product>`

This folder is a plain container, not a git repo — it just holds three sibling repos together on
disk for convenience. Nothing placed directly in this folder is ever committed anywhere, so this
`README.md` is a local, uncommitted copy (harmless — it rarely changes). `AGENTS.md`, next to it,
is a **symlink** into `<product>-docs/AGENTS.md` rather than a copy, so there's nothing to keep in
sync for that one.

**Start in [`<product>-docs/`](../docs-repo/README.md).** It's the coordination hub — specs, the
glossary, decisions log, and feature backlog — and it holds the multi-root workspace file
(`<product>-docs/<product>.code-workspace`). Open that file in VS Code, not this folder directly,
so Copilot/agents can see all three repos at once.

- [`<product>-docs/`](../docs-repo/README.md) — coordination repo. **Read its `README.md` first.**
- `<product>-api/` — API repo
- `<product>-ui/` — UI repo

**Agents/assistants:** if you're opening this folder directly rather than the workspace file, read
`AGENTS.md` before making changes anywhere in this tree, then treat `<product>-docs/` as the
source of truth for specs, vocabulary, and decisions.
