# FAQ and troubleshooting

- [Questions](#questions)
- [It's signed by someone other than who signed it before](#its-signed-by-someone-other-than-who-signed-it-before)
- [Every message, and what to do](#every-message-and-what-to-do)

## Questions

**Do I need Houston's source to write an add-on?**
Only for the `houston-addon` tool, which signs and checks. It's a Swift package in the
[Houston repository](https://github.com/Decima-Labs/houston) and builds on macOS only for now.
Everything else is HTML, JavaScript and JSON.

**Can an add-on run code outside its page?**
Not through Houston. An add-on's pages run in a web view whose only way out is `houston.call`,
through the gate. But an add-on is also a Claude Code plugin, and the plugin's hooks and MCP
servers run as you, outside Houston. Houston doesn't run, gate or list them. Read
[the Claude Code side](security-model.md#the-claude-code-side).

**Where do I find my add-on in Houston?**
**Settings › Agents › Add-ons** lists every add-on Claude Code installed, with its version, signer
and what it asks for. A screen has an **Open** button there, and is in ⌘K. A sidebar section is
below your agents. Commands are in ⌘K and the View menu, and their shortcuts work while Houston is in front. Panels are in
the menu of what they belong to: an agent's ⋯ menu, a chat's menu, a mission card's right-click
menu, or a terminal tab's right-click menu.

**Why isn't my add-on listed?**
Houston lists add-ons Claude Code copied into `~/.claude/plugins`. A plugin from a marketplace you
added from a folder on disk, or loaded with `--plugin-dir`, isn't copied there. Use a git
marketplace: [how](testing-and-debugging.md#a-marketplace-of-your-own).

**I updated it, and Houston still runs the old one.**
Open **Settings › Agents**: Houston reads its add-ons again then. And make sure Claude Code did
update it: with a `version` in `plugin.json`, it waits for that string to change.

**Can a page use `localStorage`, cookies or `fetch()`?**
They don't persist or don't work. Use [`storage`](bridge-api.md#storageget) and
[`houston.call("fetch")`](bridge-api.md#fetch).

**Can my add-on call an API that needs a token?**
Not yet. `fetch` sets no headers but `Content-Type`, so there's no `Authorization`, and a token in
a page or a URL is readable by everyone who installs it. Use public endpoints.

**Can it read the agent's chat?**
Not yet. The `chat.read` permission is in the manifest format, and Houston shows it, but no call
uses it. Don't ask for it.

**Can it add a button to the chat?**
Not yet. `contributes.chat` is in the manifest format and checked, but Houston doesn't show chat
actions. Use a [chat panel](guides/panels.md) for now.

**Can my page know my home folder?**
No. Pages see absolute paths only from what Houston gives them, such as an agent's `directory`. A
`files` permission says `~/Notes`, but `file.read` takes `/Users/you/Notes/…`: let the person
type or pick the path, or derive it from an agent's folder.

**Which agents can it start?**
`claude`, `codex`, `gemini`, `opencode`, `aider` and `local`, if the person has that agent set up
and on in Houston. Cursor starts only from its composer. See the
[guide](guides/starting-an-agent.md).

**How do I make a theme?**
Easiest: in Houston, **Settings › Appearance**, make one in the theme editor and use **Export…**.
Put the file in your add-on and list it in `contributes.themes`. See the [theme guide](guides/theme.md).

**What happens when Houston is locked?**
Nothing of any add-on works: every call is refused with "Houston is locked.", and no events are
heard. Pages stay open, so show the reason and let the person try again.

**Is uninstalling enough to remove everything?**
Claude Code removes its files. Houston keeps the grant in `addons.json` and the add-on's
`storage` in `addon-storage/<id>.json`, both in `~/Library/Application Support/Houston/`, so a
reinstall picks up where it left off. Delete them by hand to forget it entirely.

**What's the license of an add-on in this registry?**
MIT, like this repository, unless its own folder says otherwise.

## It's signed by someone other than who signed it before

Houston pins the key of the first signer it sees for each add-on `id`. When a later version is
signed by another key, Houston won't turn it on: the row says

> It's signed by someone other than who signed it before. Remove its grant first, if you trust its
> new author.

and turning it on says "Couldn't keep that: … is signed by someone other than who signed it
before. Remove it first, if you trust its new author."

This is what it looks like when someone else publishes an add-on with the same id. It's also what
happens when an author changes keys. **First find out which**: ask the author, or read the add-on's
repository. If you don't trust the new key, uninstall the add-on.

If you do trust it: Houston has no button to remove a grant yet, so you remove it by hand.

1. Quit Houston.
2. Open `~/Library/Application Support/Houston/addons.json` in a text editor. It looks like:

   ```json
   {
     "builtInOff" : [],
     "grants" : {
       "com.example.standup" : { "enabled" : true, "granted" : "…", "key" : "…", "permissions" : { … } }
     }
   }
   ```

3. Delete the add-on's entry under `grants` (the whole `"com.example.standup" : { … }`, and the
   comma before or after it, so it stays valid JSON). Leave everything else.
4. Save, open Houston, and turn the add-on on in Settings › Agents › Add-ons. You'll be asked about
   everything again, as for a new add-on.

Authors: don't do this to your people. Keep your key ([publishing](publishing.md#your-key)).

## Every message, and what to do

### In Settings › Agents › Add-ons

An add-on Houston won't run says **Not run**, with one of these:

| Message | What to do |
|---|---|
| It has no houston-addon.json, so it isn't a Houston add-on. | It's a plain Claude Code plugin. If it's yours, add the manifest. |
| It isn't signed by its author, so Houston won't run it. | Sign it: `houston-addon sign`. |
| It's signed by a key other than its author's. | `author.key` in the manifest isn't the key it was signed with. Fix the key and sign again. |
| Its files aren't the ones its author signed: it was changed after signing. | A file changed after signing. Sign again, or reinstall it. |
| It names X, which it doesn't have. | A page or theme file in the manifest isn't in the folder. |
| Its file "X" has a name Houston doesn't take. | Rename it: no part may start with `-`, be `.` or `..`, or have a control character, and the path is at most 300 characters. See [what gets signed](houston-addon-tool.md#what-gets-signed). |
| It's larger than Houston takes in an add-on. | At most 2,000 files and 50 MB in all. |
| Its X is a link or something other than a file, which an add-on can't have. | Replace the link with the file. |
| Its houston-addon.json is larger than Houston reads. | At most 256 KB. |
| Its houston-addon.json isn't one Houston can read. | Not valid JSON, a missing field, or an unknown permission or event. Run `houston-addon check`. |
| It isn't a Houston add-on. | `"houston-addon"` must be `1`. |
| It's made for a newer Houston: update Houston first. | Update Houston. |
| Its id, "x", isn't one: reverse-DNS, as dev.example.standup. | See [`id`](manifest.md#id). |
| Its name has to be one line, at most 60 characters. | |
| Its version, "x", isn't one: as 1.2.0. | Three numbers. |
| Its description is longer than 500 characters. | |
| Its author's name has to be one line, at most 100 characters. | |
| Its author's key isn't an Ed25519 public key. | Paste what `houston-addon new-key` printed. |
| It asks for more than Houston grants. / It asks for something twice. | See the [limits](manifest.md#limits-at-a-glance). |
| It asks for files at "x", which isn't a folder Houston grants: as ~/Notes, or repository. | |
| It asks to reach "x", which isn't a host: as api.example.com or *.example.com. | A host only: no `https://`, path or port. |
| It asks for agents "x": Houston knows all, aider, claude, codex, cursor, gemini, opencode. | |
| It adds more than Houston takes from one add-on. | See the [limits](manifest.md#limits-at-a-glance). |
| "x" isn't an id: lowercase words joined by hyphens. | Ids of pages, commands and themes. |
| X's title has to be one line, at most 60 characters. | |
| X's page has to be an .html file in the add-on. | |
| X's symbol isn't an SF Symbol's name. | As `checklist` or `bolt.fill`. |
| X's keys aren't ones: as cmd+shift+u. | See [shortcuts](guides/commands-and-shortcuts.md). |
| X opens something that isn't an id. / Something of it opens "x", which it doesn't have. | A command's `opens` must be the id of one of its screens, panels or sidebar sections. |
| A theme's id or title isn't one. / X's file has to be a .json file in the add-on. | |
| Two of its pages, commands or themes share an id. | |
| It hears an event twice. | |
| It's signed by someone other than who signed it before. Remove its grant first, if you trust its new author. | [Above](#its-signed-by-someone-other-than-who-signed-it-before). |
| Its new version asks to: … It stays off until you say yes. | Turn it on again to say yes to the new asks. |

### In place of a page

| Message | What to do |
|---|---|
| That add-on is off, or has no such page. | Turn it on; or the page's id changed in an update. |
| It changed since you turned it on: turn it on again in Settings › Agents › Add-ons. | Its id or key changed. |

### From houston.call

The gate's own reasons are in the [bridge reference](bridge-api.md#errors-every-call-can-have),
and each call's in its section there. The ones people meet most:

| Message | What to do |
|---|---|
| Houston is locked. | Unlock Houston. |
| X is off. | Turn it on in Settings › Agents › Add-ons. |
| X wasn't allowed that. | It's outside what the add-on asks for and you allowed. Its author can ask for it in a new version. |
| You said no. | You chose Don't Allow. |
| X is already waiting for your yes. | Answer the question Houston is showing. |

### When you use a theme

| Message | What to do |
|---|---|
| X isn't a Houston theme: That file isn't a theme Houston reads: a Houston theme, an iTerm2 .itermcolors or a Warp .yaml. | The theme file has an error. See the [theme guide](guides/theme.md). |
| X isn't a Houston theme: That theme is from a newer Houston. Update Houston to use it. | `houstonTheme` must be `1`. |

### From houston-addon

| Message | What to do |
|---|---|
| HOUSTON_ADDON_KEY isn't set: … | Run it as the message says, with your key file. |
| That isn't a key houston-addon new-key made. | `HOUSTON_ADDON_KEY` must be the contents of the key file. |
| Its author.key isn't this key's public key, X. | Put `X` in `author.key`, or sign with the right key. |
| X is there already: houston-addon never writes over a key. | Choose another file. Don't delete a key you've published with. |
