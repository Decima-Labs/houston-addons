# Testing and debugging

How to try an add-on while you write it, and how to find out what's wrong when it doesn't work.

- [A marketplace of your own](#a-marketplace-of-your-own)
- [The edit-sign-reload loop](#the-edit-sign-reload-loop)
- [Checking before you install](#checking-before-you-install)
- [Trying a page in a browser](#trying-a-page-in-a-browser)
- [Debugging in Houston](#debugging-in-houston)
- [Things that catch people out](#things-that-catch-people-out)

## A marketplace of your own

Houston doesn't install add-ons; Claude Code does, and Houston reads what it installed. To be
exact, Houston reads `~/.claude/plugins/installed_plugins.json` and takes, for each plugin, the
folder of its newest install, **if that folder is inside `~/.claude/plugins`** and has a
`houston-addon.json`.

Claude Code puts a plugin there when it copies it into its plugin cache, which it does for plugins
from a **git** marketplace: GitHub (`owner/repo`), or any git URL Claude Code can clone. It
doesn't copy a plugin from a marketplace you added from a folder on your disk
(`/plugin marketplace add ~/code/…`): it reads that one in place, so Houston doesn't see it.
`claude --plugin-dir` loads in place too, and isn't an install at all.

So, to try your add-on, keep it in a git repository that is a marketplace:

```
my-houston-addons/
├── .claude-plugin/marketplace.json
└── agent-count/                 your add-on, signed
    ├── .claude-plugin/plugin.json
    ├── houston-addon.json
    ├── houston-addon.sig
    └── pages/count.html
```

```json
{
  "name": "my-houston-addons",
  "owner": { "name": "Your Name" },
  "plugins": [
    { "name": "agent-count", "source": "./agent-count", "keywords": ["houston-addon"] }
  ]
}
```

Push it (a private repository is fine, as long as your `git` can clone it), then once:

```sh
claude plugin marketplace add you/my-houston-addons
claude plugin install agent-count@my-houston-addons
```

(Or `/plugin marketplace add …` and `/plugin install …` inside Claude Code.) Then in Houston open
**Settings › Agents › Add-ons** and turn it on.

**Leave `version` out** of `plugin.json` and the listing while you develop. Claude Code then uses
the commit as the version, so each push is an update. With a `version`, `claude plugin update`
keeps the copy it has until you change the string. Houston's own `version`, in
`houston-addon.json`, is separate and required; it's what people see.

## The edit-sign-reload loop

After every change:

```sh
# 1. Sign. This also checks the add-on exactly as Houston will.
cd ~/code/houston
HOUSTON_ADDON_KEY="$(cat ~/.houston-addon-key)" swift run houston-addon sign ~/code/my-houston-addons/agent-count

# 2. Commit and push.
cd ~/code/my-houston-addons && git commit -am "Count waiting agents too" && git push

# 3. Have Claude Code fetch the new copy.
claude plugin marketplace update my-houston-addons
claude plugin update agent-count@my-houston-addons
```

4. In Houston, **open Settings › Agents**. Houston reads its add-ons when that page opens (and
   when you change a switch there), not on its own: until you do, it keeps showing the copy it
   read before.
5. Close the page and open it again. A sidebar section: fold it and unfold it. A screen or panel:
   close it and open it again from ⌘K or the menu.

You don't turn the add-on on again unless the new version asks for more than you allowed (Houston
says "Its new version asks to: …" and keeps it off until you say yes), or its id or signing key
changed.

Forgot to sign? Houston shows "Its files aren't the ones its author signed: it was changed after
signing." in the add-on's row and doesn't run it.

## Checking before you install

`houston-addon check <folder>` runs every test Houston does on an add-on's folder (signature, key,
files, manifest) without installing it, and prints what the person will read:

```
Agent Count 1.0.0, signed by Your Name.
It asks to:
  - See all your agents
It adds:
  - Sidebar sections
  - Commands
```

Run it in CI over every add-on folder: it exits `1` with Houston's own reason when something's
wrong. See [the houston-addon tool](houston-addon-tool.md).

`claude plugin validate <folder>` checks the Claude Code side: `plugin.json`, and any skills,
commands, agents or hooks the plugin also has.

## Trying a page in a browser

A page is plain HTML and JavaScript, so you can work on its layout and logic in Safari or Chrome,
with their developer tools, by giving it a stand-in `houston`. Put this in a file beside the page,
`mock-houston.js`, and add `<script src="mock-houston.js"></script>` **before** the page's own
script while you work. Take both out before you sign: an add-on shouldn't ship its mock.

```js
// A stand-in for Houston's bridge, for a desktop browser only.
(() => {
  const heard = {};
  const answers = {
    "agents.list": () => [
      { id: "a1", provider: "claude", name: "fix-login", state: "working", directory: "/Users/you/code/site" },
      { id: "a2", provider: "codex", name: "docs", state: "idle", directory: "/Users/you/code/docs" },
    ],
    "agent.read": ({ id }) => answers["agents.list"]().find(a => a.id === id) ?? Promise.reject(new Error("There's no such agent.")),
    "storage.get": ({ key }) => localStorage.getItem("mock:" + key),
    "storage.set": ({ key, value }) => { value == null ? localStorage.removeItem("mock:" + key) : localStorage.setItem("mock:" + key, value); return true; },
    "houston.read": ({ scope }) => ({ missions: [], knowledge: [], schedules: [] })[scope],
    "notify": ({ title, body }) => { console.log("notify:", title, body); return true; },
  };
  const asks = ["agent.send", "agent.start", "file.write", "houston.change"];
  window.houston = Object.freeze({
    context: {},                       // a panel? e.g. { agent: "a1" }
    async call(name, args = {}) {
      console.log("houston.call", name, args);
      if (asks.includes(name) && !confirm(`Allow ${name}?\n${JSON.stringify(args)}`)) throw new Error("You said no.");
      const answer = answers[name];
      if (!answer) throw new Error(asks.includes(name) ? "(mock: did nothing)" : `Houston has no call ${name}.`);
      return answer(args);
    },
    on(kind, fn) { (heard[kind] ||= []).push(fn); },
  });
  // Fire an event from the console: hear("agent.stopped", { id: "a1", name: "fix-login", state: "done" })
  window.hear = (kind, data) => (heard[kind] || []).forEach(fn => { try { fn(data); } catch (e) { console.error(e); } });
})();
```

What a browser won't tell you: whether Houston's content policy allows your page (Houston serves
it with `default-src 'none'`, scripts and styles only from the add-on itself or inline, images
also from `data:`, and no network), and what the gate decides. Keep everything the page needs in
the add-on's folder, and test in Houston before you publish.

## Debugging in Houston

**There's no Web Inspector for add-on pages.** Houston doesn't make them inspectable, and
`console.log` goes nowhere you can read. So make the page tell you:

- Show every rejected call's `error.message` on the page. Houston's reasons are written for
  people ("Agent Count wasn't allowed that."), so you can show them as they are. The examples'
  [`call` helper](bridge-api.md#a-helper) does this.
- Catch what you didn't expect:

  ```js
  window.addEventListener("error", (e) => say(`${e.message} (line ${e.lineno})`));
  window.addEventListener("unhandledrejection", (e) => say(String(e.reason?.message ?? e.reason)));
  ```

- If the page is blank, it may have been refused before it loaded. Houston shows the reason in
  place of the page: "That add-on is off, or has no such page." or "It changed since you turned it
  on: turn it on again in Settings › Agents › Add-ons."
- A file that isn't in the add-on, isn't signed, or doesn't match its signature isn't served at
  all: check the name and sign again.
- A call that hangs is usually waiting for the person's **Allow Once**: look for Houston's sheet.
  Only one can wait at a time per page ("… is already waiting for your yes.").
- An event that never arrives: is it in `contributes.events`, may the add-on read what it's about,
  and is the page open? A folded sidebar section hears nothing. See the
  [events guide](guides/events.md).

Settings › Agents › Add-ons shows each add-on's state: off, on, refused (and why), or waiting for
a yes to an update. `houston-addon check` on the installed folder (its path is under
`~/.claude/plugins/cache/`) says exactly what Houston thinks of it.

## Things that catch people out

| | |
|---|---|
| The add-on isn't listed in Settings | Installed from a local-folder marketplace, or with `--plugin-dir`: Houston only reads add-ons Claude Code copied into `~/.claude/plugins`. Use a git marketplace. |
| Still the old version after an update | Open Settings › Agents, which is when Houston reads add-ons again; then reopen the page. If the plugin has a `version`, Claude Code didn't update it: change the string. |
| A new add-on's section or command doesn't appear | Turn it on in Settings › Agents › Add-ons first. Houston shows nothing from an add-on that's off. |
| `localStorage` is empty every time | Pages keep nothing between openings. Use [`storage.get` / `storage.set`](bridge-api.md#storageget). |
| `fetch()` fails in the page | Pages can't reach the web themselves. Use `houston.call("fetch", …)` and list the host in `permissions.network`. |
| `paused: true` doesn't work | Values arrive as strings: `true` becomes `"1"`. Pass `"true"` or `"false"`. |
| A shortcut does nothing | Houston or another add-on already has those keys, and the first one wins. Press ⌘/ to see Houston's. See [commands and shortcuts](guides/commands-and-shortcuts.md). |
| "It's signed by someone other than who signed it before." | You signed with a new key. See the [FAQ](faq.md#its-signed-by-someone-other-than-who-signed-it-before). |

More messages, and what to do about each, in the [FAQ](faq.md).
