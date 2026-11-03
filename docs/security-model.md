# Security model

An add-on is code from someone the person may not know, running inside the app that drives their
coding agents. Houston's design assumes the add-on might be hostile, and limits it at five layers.
This page says what each layer does, and what it doesn't.

## 1. The signature: who made it, and that it's unchanged

- Every add-on carries `houston-addon.sig`: an Ed25519 signature, by the manifest's `author.key`,
  over the SHA-256 of every file ([format](houston-addon-tool.md#the-signature-format)).
- Houston checks it whenever it reads the installed add-on, and refuses it if any file was
  changed, added or removed: "Its files aren't the ones its author signed: it was changed after
  signing."
- Houston serves a page or theme only from the signed bytes: each file is hashed again as it's
  served, and one that no longer matches isn't served.
- **The first key is pinned.** Houston keeps the grant under the add-on's id with the key it was
  granted for. A later version, or another add-on claiming the same id, signed by another key is
  refused: "It's signed by someone other than who signed it before." Someone who takes over a
  listing can't take over its users.

What the signature does **not** say: that the author is who they say, or that the code is good. It
says only that the files are the ones the holder of that key signed. Trust comes from review
(this registry's) and from the permissions.

## 2. The grant: off until the person says yes

- An installed add-on is **off**. Claude Code installing it does nothing in Houston.
- Turning it on shows who signed it and every permission it asks for, in words, and needs
  **Allow and Turn On**.
- The grant is kept in `~/Library/Application Support/Houston/addons.json`, readable by the
  person's account alone.
- If an update asks for more, the add-on stays off until the person says yes to the new asks.
- Turning it off keeps the grant but stops everything: every call is refused and no event arrives.

## 3. The gate: every call, every time

Every `houston.call` goes through `AddonGate.decide` ([permissions](permissions.md#the-gate-in-order)):

- refused while Houston is **locked**;
- refused while the add-on is **off**;
- refused outside the **intersection** of what was granted and what the installed version asks
  for, so an update can never use a grant it no longer asks for, and a grant can never cover more
  than the manifest says;
- and **asked of the person each time** for anything that changes something: messaging or starting
  an agent, writing a file, changing missions, memory or schedules. After their yes it decides
  again, in case Houston locked or the add-on was turned off meanwhile.

Events go through the same gate: a page hears an agent's events only if it may see that agent's
provider, and a mission's only if it may read missions.

## 4. The lock

Houston can be locked (by the person, or by Houston itself). While it is, add-ons are refused
everything: no calls, no events, no answers to questions they asked before. "It never works while
Houston is locked" is part of what the person reads as they turn it on.

## 5. The page: a sandbox with one door

Each page runs in its own web view:

- Served from `houston-addon://<id>/`, only the add-on's own signed files. It can't navigate
  anywhere else, open windows, or load anything from the web.
- With this content policy, which a page can't loosen:

  ```
  default-src 'none'; script-src houston-addon: 'unsafe-inline'; style-src houston-addon: 'unsafe-inline';
  img-src houston-addon: data:; font-src houston-addon:; connect-src 'none'; frame-src 'none';
  form-action 'none'; base-uri 'none'
  ```

  so no `fetch()`, XHR, WebSocket, frames, forms, or remote scripts, styles, images or fonts.
- With non-persistent web storage: nothing is kept between pages, and nothing is shared with
  Houston's own browser or any other add-on.
- Its only way out is `houston.call`, answered only for the add-on's own top frame. The `houston`
  object is frozen.
- Its `fetch` call goes through Houston: https only, granted hosts only, no cookies, no cache, no
  redirects.

## What an add-on can't do at all

- Run code outside its page. No shell, no native code, no access to the Mac but through the calls
  above.
- Read Houston's settings, credentials, tokens or other add-ons' storage.
- Read or type into an agent's terminal.
- Act while Houston is locked or the add-on is off.
- Change anything without the person's yes, each time.

## The Claude Code side

An add-on is also a Claude Code plugin, and **its plugin parts are not sandboxed by Houston.**
Skills, commands, agents, hooks and MCP servers it brings run in Claude Code, with Claude Code's
own permissions, whether or not the add-on is on in Houston. Houston's Add-ons card doesn't list
them yet, and its gate doesn't cover them: Claude Code's `/plugin` shows what a plugin brings.

That's why this registry's review reads the plugin parts as carefully as the pages, and why
add-ons here may not ship hooks or MCP servers that run code without a strong reason, stated in
the README. See the [review checklist](publishing.md#the-review-checklist).

## For authors

- Ask for the least. Every permission is shown to the person and checked by reviewers.
- Never ship a secret: your add-on's files are public, and are read by anyone who installs it.
- Don't render untrusted text as HTML. Agent names, mission titles, file contents and fetched
  bodies are data: use `textContent`, not `innerHTML`. Your page can't load scripts from the web,
  but a script injected into your page can use every permission you were granted.
- Keep your private key off any server and out of every repository.

## Reporting a vulnerability

In Houston's add-on host, or in an add-on listed here: see [SECURITY.md](../SECURITY.md). Please
don't open a public issue.
