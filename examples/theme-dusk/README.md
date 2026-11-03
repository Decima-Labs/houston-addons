# Dusk Theme

A theme and nothing else: no pages, no permissions. It's the smallest kind of add-on.

**Extension points:** [`themes`](../../docs/guides/theme.md)

## Try it

1. In Claude Code, add this registry as a marketplace and install the example:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install theme-dusk@houston-addons
   ```

2. In Houston, open **Settings › Agents › Add-ons**, turn it on and say yes to what it asks.

## Make it yours

Change the `id`, `name` and `author` in `houston-addon.json`, put your own key in `author.key`, then
sign it with `houston-addon sign`. See [Getting started](../../docs/getting-started.md).

Once it's on, click **Use Dusk Theme** under it in Settings › Agents › Add-ons. Houston keeps it as one of your own
themes, so it shows in **Settings › Appearance**.
