# Scratchpad

A sidebar section with a text box, kept with the add-on's own storage, and two commands with keyboard shortcuts.

**Extension points:** [`sidebarSections`](../../docs/guides/sidebar-section.md), [`commands` with `keys`](../../docs/guides/commands-and-shortcuts.md), the `command.ran` event, `storage`

## Try it

1. In Claude Code, add this registry as a marketplace and install the example:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install scratchpad@houston-addons
   ```

2. In Houston, open **Settings › Agents › Add-ons**, turn it on and say yes to what it asks.

## Make it yours

Change the `id`, `name` and `author` in `houston-addon.json`, put your own key in `author.key`, then
sign it with `houston-addon sign`. See [Getting started](../../docs/getting-started.md).

- **Open Scratchpad** (⌥⇧⌘P) opens the section over the window.
- **Stamp the Time in Scratchpad** (⌥⇧⌘T) has no `opens`, so Houston tells the add-on's open pages it ran. The
  sidebar section has to be open (not folded) to hear it.

Both are in ⌘K and the View menu. A shortcut Houston or an earlier add-on already uses isn't given; the
command still works from ⌘K and the menu.
