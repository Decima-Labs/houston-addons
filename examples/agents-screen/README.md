# Agents at a Glance

A full screen you open from ⌘K (or from its row in Settings › Agents › Add-ons) that lists your agents and shows one's details.

**Extension points:** [`screens`](../../docs/guides/screen.md), a command that opens it, `agents.list`, `agent.read`, `storage`

## Try it

1. In Claude Code, add this registry as a marketplace and install the example:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install agents-screen@houston-addons
   ```

2. In Houston, open **Settings › Agents › Add-ons**, turn it on and say yes to what it asks.

## Make it yours

Change the `id`, `name` and `author` in `houston-addon.json`, put your own key in `author.key`, then
sign it with `houston-addon sign`. See [Getting started](../../docs/getting-started.md).
