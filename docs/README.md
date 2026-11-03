# Documentation

Start with [Getting started](getting-started.md). Everything here matches Houston's source: where
a detail matters, the doc names the file in [Decima-Labs/houston](https://github.com/Decima-Labs/houston)
it comes from.

## Learn

- [Getting started](getting-started.md): your first add-on, from key to installed.
- [The houston-addon tool](houston-addon-tool.md): `new-key`, `sign`, `check`, and what goes in the folder.
- [Testing and debugging](testing-and-debugging.md): a marketplace of your own, the edit-sign-reload loop, debugging a page.
- [Publishing](publishing.md): the pull request, the review checklist, signing and updates.

## Reference

- [Manifest](manifest.md): every field of `houston-addon.json`.
- [Permissions](permissions.md): each permission, what the person sees, what it allows.
- [Bridge API](bridge-api.md): `houston.call`, `houston.on`, `houston.context`.
- [Security model](security-model.md): the signature, the grant, the gate, the lock and the page.
- [FAQ and troubleshooting](faq.md): every message Houston shows an author, and the fix.

## Guides

One per thing an add-on can add or do. Each has a complete, working example.

| Guide | Adds | Example |
|---|---|---|
| [Theme](guides/theme.md) | Colours, in Settings › Appearance | [`theme-dusk`](../examples/theme-dusk) |
| [⌘K screen](guides/screen.md) | A full page you open from ⌘K | [`agents-screen`](../examples/agents-screen) |
| [Commands and shortcuts](guides/commands-and-shortcuts.md) | Items in ⌘K and the View menu, with keys | [`scratchpad`](../examples/scratchpad) |
| [Panels](guides/panels.md) | A page beside an agent, a mission, a chat or a terminal | [`context-panels`](../examples/context-panels) |
| [Sidebar section](guides/sidebar-section.md) | A page below your agents | [`hello`](../addons/hello), [`scratchpad`](../examples/scratchpad) |
| [Events](guides/events.md) | Hearing agents start, ask and stop, missions move, schedules run | [`event-log`](../examples/event-log) |
| [Changing Houston](guides/changing-houston.md) | Moving missions, pausing schedules, writing memory | [`board-tools`](../examples/board-tools) |
| [Starting an agent](guides/starting-an-agent.md) | Starting Claude Code, Codex and others on a task | [`agent-starter`](../examples/agent-starter) |
| [Files and agents](guides/files-and-agents.md) | Reading and writing files, messaging an agent | [`agents-screen`](../examples/agents-screen), [`context-panels`](../examples/context-panels) |
| [Network, storage and notifications](guides/network-storage-and-notifications.md) | `fetch`, `storage`, `notify` | [`github-status`](../examples/github-status) |
| [Skills and tools for your agents](guides/agent-skills.md) | Skills, commands, MCP servers through the Claude Code plugin | [`hello`](../addons/hello) |
