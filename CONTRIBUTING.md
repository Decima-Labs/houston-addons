# Contributing

Thanks for building for Houston. There are three ways to help here: publish an add-on, improve
the docs or examples, and report what's wrong.

## Publish an add-on

1. Build and check it: [getting started](docs/getting-started.md), then
   [testing and debugging](docs/testing-and-debugging.md).
2. Read [publishing](docs/publishing.md), and go through its
   [review checklist](docs/publishing.md#the-review-checklist) yourself first.
3. Open a pull request that:
   - adds your folder under `addons/<name>/`, signed, with a README that says what it does and why
     it asks for each permission, **or** lists your own repository as the source;
   - adds one entry to [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json);
   - fills in the pull request template.

One add-on per pull request. Updates to an add-on come from its author, signed with the same key.

## Improve the docs or examples

The docs describe what Houston really does. If something here doesn't match Houston, Houston
wins: say which Houston commit you looked at, and what it does instead.

- Keep the voice: plain words, short sentences, what the person sees first.
- Snippets must work as written. Example add-ons must pass `houston-addon check`.
- Examples are signed with the examples key, which only maintainers hold. Change an example and
  leave it unsigned, and say so in the pull request; a maintainer signs it before merging.
- Check that links and headings you link to exist.

## Report a problem

Use an [issue](https://github.com/Decima-Labs/houston-addons/issues/new/choose): a bug in the docs or an example, a request for a guide,
or a report about a listed add-on. **Security problems go privately**: see
[SECURITY.md](SECURITY.md).

## What doesn't belong here

- Changes to Houston itself: this repository documents Houston's add-on interface and lists
  add-ons; Houston is developed separately.
- Secrets of any kind. Anyone can read every file here.
- Large binaries, build output, or minified code without its source.
- Your private key. Only the public key goes in a manifest.

## Commits

Small, focused commits with plain messages saying what changed and why. By contributing, you agree
your contribution is under this repository's [MIT license](LICENSE), or for an add-on, the license
its folder states.

Everyone here follows the [code of conduct](CODE_OF_CONDUCT.md).
