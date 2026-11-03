# Context Panels

One page used as four panels: in an agent's ⋯ menu, on a mission card, in a chat's ⋯ menu and on a terminal tab. Each shows `houston.context` and what it can read about it.

**Extension points:** [`panels`](../../docs/guides/panels.md) at all four places, `houston.context`, `agents.list`, `agent.read`, `houston.read`

## Try it

1. In Claude Code, add this registry as a marketplace and install the example:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install context-panels@houston-addons
   ```

2. In Houston, open **Settings › Agents › Add-ons**, turn it on and say yes to what it asks.

## Make it yours

Change the `id`, `name` and `author` in `houston-addon.json`, put your own key in `author.key`, then
sign it with `houston-addon sign`. See [Getting started](../../docs/getting-started.md).
