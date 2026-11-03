# Guide: skills and other Claude Code parts

Every Houston add-on is also a [Claude Code plugin](https://docs.claude.com/en/docs/claude-code/plugins),
so besides pages for the person, it can bring things for their Claude Code agents: skills, slash
commands, subagents, hooks and MCP servers. Example: [`hello`](../../addons/hello), which brings a
`hello` skill next to its sidebar section.

## How they differ from the rest of an add-on

| | Pages, commands, themes… | Skills, hooks, MCP servers… |
|---|---|---|
| Declared in | `houston-addon.json` | The plugin's folders, as Claude Code expects |
| Used by | Houston, for the person | Claude Code, for the agent |
| Runs | In Houston's sandbox, through the [gate](../security-model.md) | In Claude Code, with Claude Code's own permissions |
| On when | The person turns the add-on on in Houston | The plugin is enabled in Claude Code |

Houston doesn't load, list or sandbox the Claude Code parts. Only Claude Code does, and only for
Claude Code agents: Codex and the other providers don't read Claude Code plugins. See
[the Claude Code side](../security-model.md#the-claude-code-side).

They are covered by the add-on's signature, like every file in the folder, so a changed skill
fails `houston-addon check` until you sign again.

## A skill

```
my-addon/
├── .claude-plugin/plugin.json
├── houston-addon.json
├── houston-addon.sig
└── skills/
    └── release-notes/
        └── SKILL.md
```

```markdown
---
name: release-notes
description: Use when someone asks for release notes for the commits since the last tag.
---

1. Find the last tag with `git describe --tags --abbrev=0`.
2. List the commits since it with `git log <tag>..HEAD --oneline`.
3. Group them as Added, Changed and Fixed, in plain words, one line each.
```

The `description` is what Claude reads to decide when to use the skill: say when, not what. The
body is what it follows once it does. Keep it short and concrete. Files beside `SKILL.md` (a
template, a script) can be pointed to from it.

## The other parts

Each sits at the plugin's top level, as Claude Code's plugin docs describe:

| Folder or file | Brings | Review expects |
|---|---|---|
| `skills/<name>/SKILL.md` | A skill Claude uses when it fits | Plain instructions |
| `commands/<name>.md` | A `/name` slash command in Claude Code | Plain instructions |
| `agents/<name>.md` | A subagent Claude can hand work to | Plain instructions |
| `hooks/hooks.json` | Commands Claude Code runs on its own events | A strong reason, in the README |
| `.mcp.json` | MCP servers Claude Code starts | A strong reason, in the README |

Hooks and MCP servers run programs on the person's Mac, outside anything Houston can see or stop.
This registry accepts them only with a stated reason and code a reviewer can read; see the
[review checklist](../publishing.md#the-review-checklist).

A Claude Code slash command and a Houston [command](commands-and-shortcuts.md) are different
things: the first is typed in an agent's chat, the second is in ⌘K and the View menu.

## Trying them

Install the add-on from a git-hosted marketplace as in [testing and
debugging](../testing-and-debugging.md), then in Claude Code:

- `/plugin` shows the plugin and what it brings.
- `/help` lists its slash commands.
- Ask something the skill's `description` matches, and see that Claude uses it.

## Add-ons with only Claude Code parts

An add-on doesn't need pages. A manifest with no `contributes` and no `permissions` is a valid
add-on. Houston lists it in **Settings › Agents › Add-ons**, where turning it on or off changes
nothing: its parts live in Claude Code. A plain Claude Code plugin, without `houston-addon.json`, works too, but Houston
doesn't show it and this registry doesn't list it.
