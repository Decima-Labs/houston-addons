# Publishing

How to get an add-on into this registry, what a reviewer checks, and how to ship updates.

- [Two ways to share](#two-ways-to-share)
- [Before you open the pull request](#before-you-open-the-pull-request)
- [The pull request](#the-pull-request)
- [The review checklist](#the-review-checklist)
- [Updates](#updates)
- [Your key](#your-key)

## Two ways to share

**In this registry.** People who add `Decima-Labs/houston-addons` in Claude Code can install it by
name, and a person has reviewed it against the [checklist](#the-review-checklist). Houston names
this registry when no add-on is installed.

**From your own marketplace.** Any git repository with a `.claude-plugin/marketplace.json` is a
marketplace ([how](testing-and-debugging.md#a-marketplace-of-your-own)). People add it with
`/plugin marketplace add you/your-repo`. Nobody reviews it but you, and Houston holds it to the
same signature, grant and gate as any other.

You can do both: list your own repository here, and keep releasing from it.

## Before you open the pull request

- [ ] `houston-addon check` passes on the folder, and its output says exactly what you mean
      people to read.
- [ ] `claude plugin validate` passes on the folder.
- [ ] `id` is reverse-DNS under a domain or account you control: `com.yourname.standup`, or
      `io.github.yourname.standup`. Once published, it never changes.
- [ ] `version` is the same in `houston-addon.json` and `.claude-plugin/plugin.json`.
- [ ] `plugin.json` has `"keywords": ["houston-addon"]`, a `description`, an `author`, and a
      `license` if it isn't MIT.
- [ ] It asks for the least it needs (see [choosing well](permissions.md#choosing-well)).
- [ ] It has a `README.md`: what it does, what it asks for and why, and what it adds.
- [ ] It works, installed from a git marketplace, with Houston locked and unlocked, and with each
      ask answered **Don't Allow**.
- [ ] There's no secret in it: no token, password or key, not even "just for testing". Every file
      of an add-on is readable by anyone who installs it.

## The pull request

Either put the add-on in this repository:

1. Fork `Decima-Labs/houston-addons` and add your signed folder as `addons/<name>/`, where
   `<name>` is the `name` in your `plugin.json`.
2. Add it to [`.claude-plugin/marketplace.json`](../.claude-plugin/marketplace.json):

   ```json
   {
     "name": "standup",
     "source": "./addons/standup",
     "description": "Your agents' day in one screen, and a standup note in your Notes folder.",
     "version": "1.0.0",
     "author": { "name": "Jane Appleseed" },
     "keywords": ["houston-addon"]
   }
   ```

Or point at your own repository, which must have the signed add-on at its root:

```json
{
  "name": "standup",
  "source": { "source": "github", "repo": "jane/houston-standup" },
  "description": "Your agents' day in one screen, and a standup note in your Notes folder.",
  "version": "1.0.0",
  "author": { "name": "Jane Appleseed" },
  "keywords": ["houston-addon"]
}
```

Houston reads a `source` that is a folder in this repository (`"./addons/…"`), a GitHub
repository (`{ "source": "github", "repo": "owner/name" }`) or an `https` git URL
(`{ "source": "url", "url": "https://…" }`). Claude Code accepts more; keep to these.

3. Open the pull request with the template filled in. Say in one line what it's for, paste the
   output of `houston-addon check`, and say why it asks for each thing it asks for.

The listing `name` must be lowercase words joined by hyphens, unique in this registry, and the same
as your `plugin.json` `name`. Keep `"category": "example"` for the examples in this repository.

## The review checklist

A reviewer goes through this for every new add-on and every update. It's a second line: Houston's
gate holds whether or not a reviewer missed something. Review checks what the gate can't: whether
the add-on asks for what it needs and does what it says.

**Identity**

- [ ] The signature checks: `houston-addon check` passes on the folder as merged.
- [ ] For an update: `author.key` is the same as the version before. A new key is a new author,
      and needs the author to say why in the pull request (see [your key](#your-key)).
- [ ] For an update: `id` is the same, and `version` is higher, in both manifests.
- [ ] `id` and `name` don't pass as someone else's (another add-on, Houston, Decima Labs).

**What it asks for**

- [ ] Every permission is used by its pages, and its README says why.
- [ ] `files` names the narrowest folders that work. `repository` only if it works with the
      agents' own folders.
- [ ] `network` names exact hosts, not `*.` wildcards, unless the service really spreads across
      subdomains. Each host is one its README names.
- [ ] `agents` names providers, rather than `all`, unless it really is about every agent.
- [ ] `houston` asks `*.read` where it only reads.

**What it does**

- [ ] Every page's script is in the add-on and readable: no minified or obfuscated code without
      its source beside it, and no code built from strings (`eval`, `new Function`).
- [ ] Every `fetch` sends only what the README says it sends, to the hosts it says.
- [ ] `agent.send` and `agent.start` send only text the person can see in Houston's question
      before they say yes, and that matches what the add-on says it does.
- [ ] `file.write` writes only where and what the README says.
- [ ] It shows the person Houston's reasons when a call fails, rather than failing silently.
- [ ] Shortcuts don't take keys people expect from Houston or macOS.

**The Claude Code side**

- [ ] Skills, commands, agents, hooks and MCP servers in the plugin are reviewed as code: Houston
      doesn't sandbox, gate or list them. A hook or an MCP server runs with the person's own
      rights. They do what the README says, and nothing reaches the network or the disk that the
      README doesn't name.

**The listing**

- [ ] The marketplace entry's `name`, `version`, `description` and `author` match `plugin.json`,
      and `keywords` has `houston-addon`.
- [ ] No secrets, no large binaries, nothing generated that isn't needed.

## Updates

1. Change what you need, and raise `version` in **both** `houston-addon.json` and `plugin.json`
   (and the listing here). Claude Code doesn't update a plugin whose version string hasn't changed.
2. Sign with the **same key**, and run `houston-addon check`.
3. Open a pull request (or push to your own repository, if the listing points there and you've
   left `version` out of the listing). Say what changed, and whether it asks for anything new.

People get it with `/plugin marketplace update houston-addons` and `/plugin update <name>@houston-addons`,
or when Claude Code updates on its own. Then:

- If it asks for nothing new, it stays on. Houston uses the new files the next time it reads its
  add-ons.
- If it asks for more, it stays **off** until the person says yes: "Its new version asks to: …".
  Until then, nothing of it shows. Ask for more only when you must.
- A grant is never wider than what the add-on asks for now: drop a permission, and it's gone from
  the grant.

## Your key

The first key Houston sees for an add-on `id` is pinned on that Mac. Every later version must be
signed by the same key, or Houston refuses it:

> It's signed by someone other than who signed it before. Remove its grant first, if you trust its
> new author.

So:

- **Back up your key file**, somewhere safe and private. Lose it, and you can't update your
  add-on for anyone who has it.
- **Never commit it**, paste it, or put it in CI logs. Anyone with it can sign updates as you.
- If it leaks or you lose it, say so in an issue here. You'll publish under a **new `id`**, signed
  with a new key, and people move to it by choice. Don't ship a new key under the old id: everyone
  who had it would see the error above, and can only clear it by hand
  ([FAQ](faq.md#its-signed-by-someone-other-than-who-signed-it-before)).
