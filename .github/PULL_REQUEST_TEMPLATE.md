## What this changes

<!-- A new add-on, an update to one, or docs/examples. One add-on per pull request. -->

## For a new add-on or an update

- **Add-on:** name, version
- **What it does, for the person:**
- **Why it asks for each permission:**
- **Hosts it fetches from, and what it sends:**
- **What it writes, and where:**
- **Claude Code parts (skills, commands, agents, hooks, MCP servers), and why:**

- [ ] `houston-addon check <folder>` passes on the folder as it's committed.
- [ ] I went through the [review checklist](https://github.com/Decima-Labs/houston-addons/blob/main/docs/publishing.md#the-review-checklist).
- [ ] The marketplace entry matches `plugin.json`, with the `houston-addon` keyword.
- [ ] For an update: same `id`, same `author.key`, higher `version` in both manifests.
- [ ] No secrets, no private key, no build output, no minified code without its source.

## For docs or examples

- [ ] Checked against Houston (commit or version):
- [ ] Snippets work as written; links and headings exist.
- [ ] Changed examples are left for a maintainer to sign, or still pass `houston-addon check`.
