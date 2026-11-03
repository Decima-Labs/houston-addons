# Houston add-ons

The registry of add-ons for [Houston](https://github.com/Decima-Labs/houston), and everything you
need to write your own.

An add-on adds to Houston without changing it: a theme, a screen you open from ⌘K, a command with a
keyboard shortcut, a panel beside an agent, a mission, a chat or a terminal, a section in the sidebar.
Its pages can read what you let them read and, with your yes each time, act: message or start an
agent, write a file, move a mission, pause a schedule.

This repository is a [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces).
Houston knows it as `Decima-Labs/houston-addons`.

- [What an add-on is](#what-an-add-on-is)
- [Install one](#install-one)
- [Write one in five minutes](#write-one-in-five-minutes)
- [How install, review and permissions work](#how-install-review-and-permissions-work)
- [Documentation](#documentation)
- [Examples](#examples)
- [Publish yours](#publish-yours)

## What an add-on is

A Houston add-on is a Claude Code plugin with two more files beside its `.claude-plugin/plugin.json`:

```
my-addon/
├── .claude-plugin/plugin.json   the Claude Code plugin: name, version, keywords ["houston-addon"]
├── houston-addon.json           the manifest: who signed it, what it asks for, what it adds
├── houston-addon.sig            the signature over every file in the folder
├── pages/hello.html             pages Houston shows, if it adds any
├── themes/dark.json             themes, if it adds any
└── skills/ commands/ agents/ …  anything else a Claude Code plugin has, for your agents
```

- **Claude Code installs it.** Houston doesn't have its own installer: it reads what Claude Code
  installed, so the same plugin can also give your agents skills, commands and rules.
- **Houston runs it.** Houston shows its pages in a locked-down web view and answers their
  `houston.call(...)` through a gate that checks every call against what you allowed.
- **The author signs it.** Every file is signed with the author's Ed25519 key. Change one byte after
  signing and Houston won't run it.

## Install one

1. In Claude Code, add the marketplace and install an add-on:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install hello@houston-addons
   ```

2. In Houston, open **Settings › Agents**. Under **Add-ons**, find it and turn it on. Houston shows
   who signed it and everything it asks for; it stays off until you say **Allow and Turn On**.

3. Use it: open ⌘K for its screens and commands, look below your agents in the sidebar for its
   sections, or open the ⋯ menu of an agent or a chat for its panels.

Turn it off in the same place. Remove it with Claude Code's `/plugin uninstall`.

## Write one in five minutes

You need a Mac with Houston, Claude Code, and a clone of the Houston repository for its
`houston-addon` tool (it builds on macOS).

```sh
# 1. Make your signing key, once. Keep the file safe and out of git: it is your signature.
cd ~/code/houston
swift run houston-addon new-key ~/.houston-addon-key
#   → Your add-on's author.key: AbC…=

# 2. Start from the hello example.
cp -R ~/code/houston-addons/addons/hello ~/code/my-addon
```

3. Edit `~/code/my-addon/houston-addon.json`: set your own `id` (reverse-DNS, as
   `com.example.my-addon`), `name`, and `author` with the `key` step 1 printed. Edit
   `.claude-plugin/plugin.json` to match (`name`, `description`, `author`).

4. Sign it and check it the way Houston will:

   ```sh
   HOUSTON_ADDON_KEY="$(cat ~/.houston-addon-key)" swift run houston-addon sign ~/code/my-addon
   #   → My Add-on 1.0.0, signed by You.
   #     It asks to: nothing
   #     It adds:
   #       - Sidebar sections
   #       - Commands
   ```

5. Install it from a marketplace of your own: a git repository with
   `.claude-plugin/marketplace.json` listing it, pushed where Claude Code can clone it (see
   [Testing and debugging](docs/testing-and-debugging.md#a-marketplace-of-your-own)). Turn it on in
   Settings › Agents › Add-ons, and you'll see **Hello** below your agents.

Then change the page, add what you need from the [guides](docs/README.md#guides), and sign again
after every change. The full walkthrough is in [Getting started](docs/getting-started.md).

## How install, review and permissions work

**Signed.** `houston-addon sign` hashes every file in the folder (SHA-256) and signs the list with
your key. Houston checks the signature, that the key is the manifest's `author.key`, and that
nothing changed, every time it reads the add-on and every time it serves one of its files. The
first key Houston sees for an add-on id is pinned: an update signed by someone else is refused
until you remove the old grant yourself.

**Granted.** An add-on asks for permissions in its manifest: folders it may read, hosts it may
reach, agents it may see, and Houston's own things (missions, memory, schedules, notifications). You
see each in plain words before you turn it on, and Houston keeps only what you said yes to. A new
version that asks for more stays off until you say yes again. A grant is never wider than what the
add-on asks for now.

**Gated.** Every call a page makes goes through Houston's gate. Nothing works while Houston is
locked or the add-on is off. Reads are allowed only within the grant. Anything that acts (sending
to or starting an agent, writing a file, changing a mission, a memory or a schedule) asks you
first, every time, and is checked again after your yes.

**Contained.** Pages are served only from the add-on's signed files, under a content policy that
blocks every request to the web, in a web view that keeps no cookies or storage, can't navigate
away and can't open windows. The only way out is `houston.call`, and the gate.

**Reviewed.** Add-ons listed in this registry are reviewed by a person against the
[review checklist](docs/publishing.md#the-review-checklist) before they're merged. Review is a
second line: the gate is the first, and it holds for add-ons from any marketplace.

More in the [security model](docs/security-model.md) and the [permissions reference](docs/permissions.md).

## Documentation

| | |
|---|---|
| [Getting started](docs/getting-started.md) | Your first add-on, from key to installed, step by step |
| [The houston-addon tool](docs/houston-addon-tool.md) | `new-key`, `sign`, `check`, and what goes in the folder |
| [Manifest reference](docs/manifest.md) | Every field of `houston-addon.json`, with its limits |
| [Permissions reference](docs/permissions.md) | Each permission, what you see, what it allows, and the gate |
| [Bridge API reference](docs/bridge-api.md) | `houston.call`, `houston.on`, `houston.context`: every call and event |
| [Security model](docs/security-model.md) | What protects you, and what an add-on can't get around |
| [Testing and debugging](docs/testing-and-debugging.md) | A marketplace of your own, the edit-sign-reload loop, debugging a page |
| [Publishing](docs/publishing.md) | Getting into this registry: the PR, the review checklist, updates |
| [FAQ and troubleshooting](docs/faq.md) | Every error Houston shows, and what to do about it |

**Guides**, one per thing an add-on can add or do:
[Theme](docs/guides/theme.md) ·
[⌘K screen](docs/guides/screen.md) ·
[Commands and shortcuts](docs/guides/commands-and-shortcuts.md) ·
[Panels](docs/guides/panels.md) ·
[Sidebar section](docs/guides/sidebar-section.md) ·
[Events](docs/guides/events.md) ·
[Changing Houston](docs/guides/changing-houston.md) ·
[Starting an agent](docs/guides/starting-an-agent.md) ·
[Files and agents](docs/guides/files-and-agents.md) ·
[Network, storage and notifications](docs/guides/network-storage-and-notifications.md) ·
[Skills and tools for your agents](docs/guides/agent-skills.md)

## Examples

Small, signed, working add-ons in [`examples/`](examples/), one per extension point. Each has a
README saying what it shows.

| Example | Shows |
|---|---|
| [`addons/hello`](addons/hello) | The starting point: a sidebar section, a command, a skill |
| [`theme-dusk`](examples/theme-dusk) | A theme, and nothing else |
| [`agents-screen`](examples/agents-screen) | A ⌘K screen, `agents.list`, `agent.read`, storage |
| [`scratchpad`](examples/scratchpad) | A sidebar section, commands with shortcuts, `command.ran` |
| [`context-panels`](examples/context-panels) | Panels on an agent, a mission, a chat and a terminal |
| [`event-log`](examples/event-log) | Every event, live |
| [`board-tools`](examples/board-tools) | `houston.change`: missions, schedules, memory |
| [`agent-starter`](examples/agent-starter) | `agent.start` |
| [`github-status`](examples/github-status) | `fetch` and `notify` |

The examples are listed in this marketplace with the `example` category and keyword, so you can
install any of them by name: `/plugin install event-log@houston-addons`.

## Publish yours

Open a pull request that adds your add-on under `addons/<name>/` (or points at your own repository)
and lists it in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json). See
[Publishing](docs/publishing.md) and [CONTRIBUTING](CONTRIBUTING.md).

You don't need this registry to share an add-on: any git-hosted Claude Code marketplace works.
Houston knows an add-on by its signed `houston-addon.json`, wherever it came from, and holds it to
the same signature, grant and gate. The `houston-addon` keyword is how a marketplace says a listing
is one.

## License

[MIT](LICENSE). Each add-on in `addons/` is under this license unless its own folder says otherwise.

This license covers this repository only. Houston itself has no license and isn't open source;
the docs here describe its add-on interface so you can build for it.
