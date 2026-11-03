# Agent Starter

Starts an agent with `agent.start`. Houston asks you first, every time, and starts it the way its own composer would.

**Extension points:** [`agent.start`](../../docs/guides/starting-an-agent.md), a screen, a command, `storage`

## Try it

1. In Claude Code, add this registry as a marketplace and install the example:

   ```
   /plugin marketplace add Decima-Labs/houston-addons
   /plugin install agent-starter@houston-addons
   ```

2. In Houston, open **Settings › Agents › Add-ons**, turn it on and say yes to what it asks.

## Make it yours

Change the `id`, `name` and `author` in `houston-addon.json`, put your own key in `author.key`, then
sign it with `houston-addon sign`. See [Getting started](../../docs/getting-started.md).
