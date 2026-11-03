# Board Tools

Changes Houston's own things with `houston.change`: a mission's status, a schedule's pause, a new memory. Houston asks you before each change.

**Extension points:** [`houston.change`](../../docs/guides/changing-houston.md), `houston.read`, a screen

## Try it

1. In Claude Code, add this registry as a marketplace and install the example:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install board-tools@houston-addons
   ```

2. In Houston, open **Settings › Agents › Add-ons**, turn it on and say yes to what it asks.

## Make it yours

Change the `id`, `name` and `author` in `houston-addon.json`, put your own key in `author.key`, then
sign it with `houston-addon sign`. See [Getting started](../../docs/getting-started.md).
