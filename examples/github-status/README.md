# GitHub Status

Reaches one host on the web with `fetch` and sends a notification with `notify`.

**Extension points:** [`fetch` and `notify`](../../docs/guides/network-storage-and-notifications.md), a screen, a command, `storage`

## Try it

1. In Claude Code, add this registry as a marketplace and install the example:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install github-status@houston-addons
   ```

2. In Houston, open **Settings › Agents › Add-ons**, turn it on and say yes to what it asks.

## Make it yours

Change the `id`, `name` and `author` in `houston-addon.json`, put your own key in `author.key`, then
sign it with `houston-addon sign`. See [Getting started](../../docs/getting-started.md).
