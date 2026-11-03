# Guide: commands and shortcuts

A command is an item in ⌘K and in Houston's **View** menu, optionally with a keyboard shortcut. It
does one of two things:

- **opens** one of your pages (a screen, a panel or a sidebar section) over the window; or
- **tells your open pages it ran**, as the `command.ran` event, so a page that's already showing
  can act on it.

Example: [`scratchpad`](../../examples/scratchpad), a sidebar notepad with one command of each
kind.

## 1. A command that opens a page

```json
"contributes": {
  "sidebarSections": [
    { "id": "scratchpad", "title": "Scratchpad", "page": "pages/scratchpad.html", "symbol": "note.text" }
  ],
  "commands": [
    { "id": "open-scratchpad", "title": "Open Scratchpad", "keys": "cmd+option+shift+p", "opens": "scratchpad" }
  ]
}
```

`opens` names one of the add-on's own pages by its `id`. Running the command opens that page in a
sheet over the window, bringing the window forward if it was hidden: the same sheet ⌘K's **Go to**
opens. A sidebar section opened this way is a second, separate copy of the page; the one in the
sidebar stays as it was.

A command that opens a panel opens it without a context (`houston.context` is `{}`), since it
wasn't opened beside anything. A panel's page should cope with that.

## 2. A command your page hears

Leave `opens` out, and list `command.ran` in `events`:

```json
"commands": [
  { "id": "stamp-time", "title": "Stamp the Time in Scratchpad", "keys": "cmd+option+shift+t" }
],
"events": ["command.ran"]
```

Then, in the page:

```js
houston.on("command.ran", (data) => {
  if (data.id !== "stamp-time") return;   // data.id is the command's id
  const box = document.getElementById("text");
  box.value += new Date().toLocaleString() + " ";
  box.focus();
});
```

- Only your own add-on's pages hear your commands. No permission is needed.
- **Only open pages hear it.** If none of your pages is open (the sidebar section folded, no screen
  showing), running the command does nothing at all. Pair this kind of command with a sidebar
  section, or tell people in its title where it acts ("… in Scratchpad").
- Every open page of yours hears it: if the same page is open in the sidebar and in a sheet, both
  handlers run.
- Without `command.ran` in `events`, the command still appears, but no page hears it.

## 3. Shortcuts

`keys` is the shortcut, written as modifiers and one key joined by `+`:

| Valid | Not valid | Why |
|---|---|---|
| `cmd+shift+u` | `shift+u` | `cmd` is required |
| `cmd+option+shift+p` | `cmd+shift+F5` | the key must be one letter or digit |
| `cmd+ctrl+1` | `cmd+cmd+k` | each modifier at most once |
| | `command+k` | modifiers are `cmd`, `shift`, `option`, `ctrl` |

Write the letter in lowercase. A `keys` that isn't valid makes the manifest unreadable, so
`houston-addon check` catches it.

### Who gets a key

Houston decides when it builds the menu, every time:

1. **Houston's own keys win.** A command asking for a key Houston uses anywhere (in any screen, the
   editor, the mission board, a text field) gets no shortcut. It's still in ⌘K and the View menu.
2. **Then the first add-on wins.** Add-ons ask in the order Settings › Agents lists them,
   alphabetical by plugin and marketplace (`name@marketplace`); a later one asking for a key
   already given gets none. Within one add-on, its commands ask in the manifest's order.
3. Nothing tells the person or you that a key was skipped. The View menu shows the keys each
   command actually got, so look there after installing.

To see what Houston uses, open **Help › Keyboard Shortcuts** (⌘/). Short combinations are nearly
all taken; three modifiers, as `cmd+option+shift+<letter>`, are the safest choice.

Shortcuts work while Houston is the frontmost app, as menu keys do. They don't work in other apps,
and the View menu (with them) is disabled while Houston is hands-off.

## Naming

- `title` is what the person reads in ⌘K and the menu: a verb first, the add-on's thing named,
  at most 60 characters. "Open Scratchpad", "Check GitHub Status".
- `id` is yours: a word of `a-z`, `0-9` and `-`. It's what `command.ran` gives back, so don't
  rename it in an update.
- Up to 50 commands per add-on. See [`commands`](../manifest.md#commands).
