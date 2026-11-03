# Manifest reference

`houston-addon.json` sits at the top of the add-on, beside `.claude-plugin/plugin.json`. It says
who signed the add-on, what it asks for, and what it adds to Houston. Houston reads it with
`AddonManifest.read` (`Sources/HoustonCore/AddonManifest.swift` and `AddonContributions.swift`).

A complete one:

```json
{
  "houston-addon": 1,
  "id": "com.example.standup",
  "name": "Standup",
  "version": "1.2.0",
  "description": "Writes your standup from what your agents did yesterday.",
  "author": {
    "name": "Example Co",
    "key": "4n0pRbW2lq…Q8="
  },
  "permissions": {
    "files": ["~/Notes", "repository"],
    "network": ["api.example.com"],
    "agents": ["claude", "codex"],
    "houston": ["missions.read", "knowledge.write", "notify"]
  },
  "contributes": {
    "screens": [
      { "id": "standup", "title": "Standup", "page": "pages/standup.html", "symbol": "sun.max" }
    ],
    "panels": [
      { "id": "agent-summary", "title": "Summarize for Standup", "page": "pages/summary.html", "place": "agent" }
    ],
    "sidebarSections": [
      { "id": "today", "title": "Today", "page": "pages/today.html", "symbol": "calendar" }
    ],
    "commands": [
      { "id": "open-standup", "title": "Open Standup", "keys": "cmd+option+shift+u", "opens": "standup" },
      { "id": "refresh", "title": "Refresh Standup" }
    ],
    "themes": [
      { "id": "morning", "title": "Morning", "file": "themes/morning.json" }
    ],
    "events": ["agent.stopped", "mission.moved", "command.ran"]
  }
}
```

The file can be at most 256 KB. Fields Houston doesn't know are ignored, so a misspelled field is
silently ignored too: run `houston-addon check` and read what it says it adds.

## Top level

| Field | Required | Rules |
|---|---|---|
| `houston-addon` | yes | The format, a whole number. Today `1`. A higher one is refused as "made for a newer Houston". |
| `id` | yes | Reverse-DNS: 2 to 8 words joined by `.`, each word 1–40 characters of `a-z`, `0-9` and `-`, not starting or ending with `-`; at most 100 characters in all. |
| `name` | yes | One line, at most 60 characters. What people see. |
| `version` | yes | `MAJOR.MINOR.PATCH`, digits only, with an optional `-prerelease` of letters, digits, `.` and `-`: `1.0.0`, `2.1.0-beta.1`. At most 40 characters. |
| `description` | no | At most 500 characters. |
| `author` | yes | Who signs it: see below. |
| `permissions` | no | What it asks to read and do. Left out, it asks for nothing. |
| `contributes` | no | What it adds to Houston. Left out, it adds nothing (it may still bring skills through its plugin). |

### id

The `id` is the add-on's identity, and Houston keeps your grant and the pinned signer under it.

- Use a domain you control, reversed: `com.example.standup`, `io.github.yourname.standup`.
- Never change it. A new id is a new add-on to Houston: the person has to turn it on again.
- It names the add-on's storage and its pages' address, `houston-addon://<id>/…`.

Valid: `com.example.standup`, `dev.jane.pr-review`, `io.github.jane.a1`.
Not valid: `standup` (one word), `com.Example.standup` (capitals), `com.example.-x` (leading `-`),
`com.example.my_addon` (`_`).

### author

```json
"author": { "name": "Example Co", "key": "4n0pRbW2lq…Q8=" }
```

| Field | Rules |
|---|---|
| `name` | One line, at most 100 characters. Shown as "signed by …". |
| `key` | The public key `houston-addon new-key` printed: base64 of a 32-byte Ed25519 key. |

The signature in `houston-addon.sig` has to be by this key.

## permissions

Each list is optional, and no list may name the same thing twice. Everything is explained, with
what the person sees, in the [permissions reference](permissions.md).

| Field | At most | Each is |
|---|---|---|
| `files` | 20 | A folder in your home, as `~/Notes` (not `~` alone, no trailing `/`, no `.` or `..`, at most 200 characters), or the word `repository` |
| `network` | 50 | A host, lowercase, as `api.example.com`, or every host under a domain, as `*.example.com` |
| `agents` | 10 | `all`, or a provider: `claude`, `codex`, `cursor`, `gemini`, `opencode`, `aider` |
| `houston` | 8 | `missions.read`, `missions.write`, `knowledge.read`, `knowledge.write`, `schedules.read`, `schedules.write`, `chat.read`, `notify` |

A value Houston doesn't know in `houston` makes the whole manifest unreadable, so check spelling.

## contributes

Every id in `contributes` is a word: 1–40 characters of `a-z`, `0-9` and `-`, not starting or ending
with `-`. Screens, panels and sidebar sections share one set of ids (they're all pages); commands
have their own, themes their own.

Every `page` is a path to an `.html` file in the add-on, as `pages/home.html`. Every theme `file` is
a `.json` file in the add-on. Paths use `/`, are relative to the add-on folder, and follow the
[path rules](houston-addon-tool.md#what-gets-signed). A page or theme the manifest names but the
folder doesn't have is refused.

Several entries may use the same page file.

### screens

A full page that opens over the window, from ⌘K or from the add-on's row in Settings. At most 10.
[Guide](guides/screen.md).

| Field | Required | Rules |
|---|---|---|
| `id` | yes | A word, unique among pages |
| `title` | yes | One line, at most 60 characters |
| `page` | yes | An `.html` file in the add-on |
| `symbol` | no | An SF Symbol's name, as `sun.max`: `a-z`, `0-9` and `.`, at most 60 characters, not starting or ending with `.`. Defaults to `puzzlepiece.extension`. |

### panels

A page beside an agent, a mission, a chat or a terminal tab. At most 20. [Guide](guides/panels.md).

| Field | Required | Rules |
|---|---|---|
| `id` | yes | A word, unique among pages |
| `title` | yes | One line, at most 60 characters. The menu item's words. |
| `page` | yes | An `.html` file in the add-on |
| `place` | yes | `agent`, `mission`, `chat` or `terminal` |

### sidebarSections

A page under a heading in the sidebar, below your agents. At most 5.
[Guide](guides/sidebar-section.md).

Same fields as `screens`: `id`, `title`, `page`, `symbol`.

### commands

An item in ⌘K and in the View menu, optionally with a keyboard shortcut. At most 50.
[Guide](guides/commands-and-shortcuts.md).

| Field | Required | Rules |
|---|---|---|
| `id` | yes | A word, unique among commands |
| `title` | yes | One line, at most 60 characters |
| `keys` | no | A shortcut, as `cmd+shift+u`: 2 to 5 parts joined by `+`. The last is one letter or digit; the others are `cmd`, `shift`, `option`, `ctrl`, each at most once, and `cmd` is required. |
| `opens` | no | The id of one of its screens, panels or sidebar sections. Left out, running the command tells the add-on's open pages, as the `command.ran` event. |

### themes

A Houston theme: colours only. At most 20. [Guide](guides/theme.md).

| Field | Required | Rules |
|---|---|---|
| `id` | yes | A word, unique among themes |
| `title` | yes | One line, at most 60 characters |
| `file` | yes | A `.json` file in the add-on, in Houston's theme format |

### events

The events its open pages may hear, each at most once:

`agent.started`, `agent.stopped`, `agent.asks`, `agent.message`, `mission.moved`, `schedule.ran`,
`command.ran`.

An event not listed here never reaches its pages, whatever they `houston.on`. An unknown name makes
the manifest unreadable. [Guide](guides/events.md).

### chat

Chat actions: an item in an agent chat that opens one of its pages. At most 10.

| Field | Required | Rules |
|---|---|---|
| `id` | yes | A word |
| `title` | yes | One line, at most 40 characters |
| `opens` | yes | The id of one of its pages |

Houston reads and validates these, and `check` lists "Chat actions", but today's Houston doesn't
show them anywhere yet. To put something in a chat now, use a [panel](guides/panels.md) with
`"place": "chat"`, which appears in the chat's ⋯ menu.

## Limits at a glance

| | Limit |
|---|---|
| Manifest file | 256 KB |
| `id` | 2–8 words, 100 characters |
| `name` / `title` | 60 characters (chat titles 40) |
| `description` | 500 characters |
| `author.name` | 100 characters |
| screens / panels / sidebar sections | 10 / 20 / 5 |
| commands / themes / chat | 50 / 20 / 10 |
| files / network / agents permissions | 20 / 50 / 10 |
| Add-on folder | 2,000 files, 50 MB |
| `houston-addon.sig` | 4 KB |

## The plugin manifest

`.claude-plugin/plugin.json` is Claude Code's, not Houston's. Houston needs nothing from it, but
marketplaces do. Keep these in step with `houston-addon.json`:

```json
{
  "name": "standup",
  "version": "1.2.0",
  "description": "Writes your standup from what your agents did yesterday.",
  "author": { "name": "Example Co" },
  "keywords": ["houston-addon"]
}
```

`name` is the plugin's name in Claude Code and the marketplace (`/plugin install standup@…`):
lowercase words joined by `-`. It doesn't have to match the add-on's `id`.
