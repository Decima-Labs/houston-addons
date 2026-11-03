# Getting started

This walks you from nothing to an add-on of your own, installed and running in Houston. It takes
about fifteen minutes the first time.

You'll make a sidebar section with a button that lists your agents. Along the way you'll see every
part of an add-on: the plugin, the manifest, a page, a permission, the signature and the grant.

## What you need

- A Mac with **Houston** and **Claude Code**.
- A clone of [Decima-Labs/houston](https://github.com/Decima-Labs/houston), for the
  `houston-addon` tool, and Swift (Xcode or the command-line tools). The tool builds on macOS only
  for now.
- A clone of this repository, for the examples:
  `git clone https://github.com/Decima-Labs/houston-addons ~/code/houston-addons`.

## 1. Make your key

Your key is your signature. Make it once and use it for every add-on you write.

```sh
cd ~/code/houston
swift run houston-addon new-key ~/.houston-addon-key
```

```
Your add-on's author.key: 4n0pR…Q8=
```

- `~/.houston-addon-key` holds the private key, readable by you alone (mode 0600). Back it up
  somewhere safe. **Never commit it.** If you lose it you can't ship updates under the same
  signer: Houston pins the first key it sees for an add-on id.
- The printed line is your **public** key. It goes in every manifest you sign, as `author.key`.
- `new-key` never writes over an existing file.

## 2. Copy hello

```sh
cp -R ~/code/houston-addons/addons/hello ~/code/agent-count
cd ~/code/agent-count
```

You now have:

```
agent-count/
├── .claude-plugin/plugin.json   the Claude Code plugin
├── houston-addon.json           the manifest
├── houston-addon.sig            hello's signature (yours replaces it)
├── pages/hello.html             the page
├── skills/hello/SKILL.md        a skill for your agents
└── README.md
```

## 3. Make the plugin yours

`.claude-plugin/plugin.json` is what Claude Code reads. Keep `"keywords": ["houston-addon"]`: it's
how Houston and marketplaces know this plugin is an add-on.

```json
{
  "name": "agent-count",
  "version": "1.0.0",
  "description": "How many agents are running, in the sidebar.",
  "author": { "name": "Your Name" },
  "keywords": ["houston-addon"]
}
```

Delete `skills/hello` (or keep it and see [Skills and tools for your agents](guides/agent-skills.md)).

## 4. Write the manifest

Replace `houston-addon.json` with this, putting your public key in `author.key`:

```json
{
  "houston-addon": 1,
  "id": "com.example.agent-count",
  "name": "Agent Count",
  "version": "1.0.0",
  "description": "How many agents are running, in the sidebar.",
  "author": {
    "name": "Your Name",
    "key": "4n0pR…Q8="
  },
  "permissions": {
    "agents": ["all"]
  },
  "contributes": {
    "sidebarSections": [
      { "id": "agent-count", "title": "Agent Count", "page": "pages/count.html", "symbol": "number" }
    ],
    "commands": [
      { "id": "show-agent-count", "title": "Show Agent Count", "opens": "agent-count" }
    ]
  }
}
```

- `id` is the add-on's identity: reverse-DNS, lowercase. Pick one you own and never change it.
- `permissions.agents: ["all"]` asks to see your agents. Without it, `agents.list` is refused.
- `contributes` says what it adds: a section in the sidebar, and a command in ⌘K that opens it.

Every field is in the [manifest reference](manifest.md).

## 5. Write the page

Rename `pages/hello.html` to `pages/count.html` and replace it with:

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<style>
  body { font: 13px -apple-system, system-ui, sans-serif; margin: 12px; color: #222; }
  @media (prefers-color-scheme: dark) { body { color: #ddd; } }
</style>
</head>
<body>
<p id="count">…</p>
<button id="refresh">Refresh</button>
<script>
  async function refresh() {
    try {
      const agents = await houston.call("agents.list");
      const busy = agents.filter((a) => ["starting", "working", "waiting"].includes(a.state));
      document.getElementById("count").textContent = `${busy.length} of ${agents.length} agents at work.`;
    } catch (e) {
      document.getElementById("count").textContent = String(e.message || e);
    }
  }
  document.getElementById("refresh").onclick = refresh;
  refresh();
</script>
</body>
</html>
```

`houston` is there before your script runs. `houston.call(name, args)` returns a promise; it
rejects with Houston's reason when a call isn't allowed. Every call is in the
[bridge API reference](bridge-api.md).

Pages can use inline scripts and styles and files from the add-on itself (`<script src="app.js">`,
images, fonts). They can't load anything from the web. See [Security model](security-model.md#5-the-page-a-sandbox-with-one-door).

## 6. Sign and check

```sh
cd ~/code/houston
HOUSTON_ADDON_KEY="$(cat ~/.houston-addon-key)" swift run houston-addon sign ~/code/agent-count
```

```
Agent Count 1.0.0, signed by Your Name.
It asks to:
  - See all your agents
It adds:
  - Sidebar sections
  - Commands
```

`sign` writes `houston-addon.sig` and then runs `check`, which verifies the add-on exactly as
Houston will. "It asks to" is what the person will read before they say yes.

**Sign again after every change**, to any file. Houston refuses an add-on whose files changed
after signing.

## 7. Install it

Claude Code installs add-ons from marketplaces, and Houston reads only the add-ons Claude Code has
copied into its plugin folder, `~/.claude/plugins`. Claude Code copies a plugin there when its
marketplace is a git repository. (A marketplace added from a folder on your disk is read in
place, not copied, so Houston doesn't see its add-ons. See
[Testing and debugging](testing-and-debugging.md#a-marketplace-of-your-own).)

So make a marketplace of your own: a git repository, private if you like, with
`.claude-plugin/marketplace.json` listing your add-on.

```sh
mkdir -p ~/code/my-houston-addons/.claude-plugin
mv ~/code/agent-count ~/code/my-houston-addons/
cat > ~/code/my-houston-addons/.claude-plugin/marketplace.json <<'JSON'
{
  "name": "my-houston-addons",
  "owner": { "name": "Your Name" },
  "plugins": [
    { "name": "agent-count", "source": "./agent-count", "keywords": ["houston-addon"] }
  ]
}
JSON
cd ~/code/my-houston-addons
git init && git add -A && git commit -m "Add Agent Count"
```

Push it to GitHub (or any git host your `git` can clone from), as `you/my-houston-addons`. Then
in Claude Code:

```
/plugin marketplace add you/my-houston-addons
/plugin install agent-count@my-houston-addons
```

The listing has no `version`, and neither should `plugin.json` while you develop: then Claude
Code uses the commit as the version, and every push is an update. With a `version`, Claude Code
keeps the copy it has until the string changes.

## 8. Turn it on

In Houston, open **Settings › Agents** and find **Add-ons**. Your add-on is listed, off, with its
version, its signer, and "May see all your agents". Turn it on: Houston asks

> Turn on Agent Count?
> Signed by Your Name. It asks to:
> • See all your agents
> It still asks you before anything that needs a yes, and never works while Houston is locked.

Say **Allow and Turn On**. **Agent Count** appears below your agents in the sidebar, and
**Show Agent Count** is in ⌘K and the View menu.

## 9. Change it

Edit the page, sign again, commit and push, then have Claude Code fetch the new copy:

```sh
claude plugin marketplace update my-houston-addons
claude plugin update agent-count@my-houston-addons
```

Then open **Settings › Agents** in Houston: that is when Houston reads its add-ons again and
finds the new copy. Close and reopen the page to see your change. You turn it on again only if
the new version asks for more, or its id or key changed.
The whole loop is in [Testing and debugging](testing-and-debugging.md#the-edit-sign-reload-loop).
When you publish, give `plugin.json` a `version` that matches `houston-addon.json`, and bump both
with every release.

## Next

- Add more: a [screen](guides/screen.md), [panels](guides/panels.md),
  [shortcuts](guides/commands-and-shortcuts.md), [events](guides/events.md), a [theme](guides/theme.md).
- Read the [permissions reference](permissions.md) before you ask for any.
- [Publish it](publishing.md) to this registry.
