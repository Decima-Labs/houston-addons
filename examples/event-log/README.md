# Event Log

Listens to every event an add-on can hear, in the sidebar and as a ⌘K screen.

**Extension points:** [`events`](../../docs/guides/events.md), `houston.on`, a sidebar section and a screen sharing one page

## Try it

1. In Claude Code, add this registry as a marketplace and install the example:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install event-log@houston-addons
   ```

2. In Houston, open **Settings › Agents › Add-ons**, turn it on and say yes to what it asks.

## Make it yours

Change the `id`, `name` and `author` in `houston-addon.json`, put your own key in `author.key`, then
sign it with `houston-addon sign`. See [Getting started](../../docs/getting-started.md).

A page hears events only while it is open: keep the sidebar section unfolded, or open the screen.
