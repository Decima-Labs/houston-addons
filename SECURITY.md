# Security

## Reporting a problem

Please report security problems **privately**, through GitHub's
[private vulnerability reporting](https://github.com/Decima-Labs/houston-addons/security/advisories/new) for this repository. Don't open a
public issue, pull request or discussion for them.

Say what's affected, how to reproduce it, and what someone could do with it. We'll answer within
a few working days, keep you updated, and credit you when it's fixed if you'd like.

## What to report here

- An add-on listed in this registry that does something its README and permissions don't say: it
  reaches hosts it shouldn't, writes files it shouldn't, sends agents text the person didn't see,
  or hides what its code does.
- A way for an add-on to get past Houston's gate, signature check or page sandbox, as described in
  the [security model](docs/security-model.md).
- A mistake in these docs that would lead authors or users to something unsafe.
- A secret committed to this repository by mistake.

Problems in Houston itself can be reported here too; we'll get them to the right place.

## What happens to a bad add-on

A listed add-on found to be harmful is removed from the marketplace at once, and its author is
told. Removing a listing stops new installs; people who installed it already should uninstall it
in Claude Code (`/plugin`). Houston keeps its grant afterwards; see the
[FAQ](docs/faq.md) to remove it entirely.

## Supported versions

The registry is its `main` branch. Docs follow Houston's current release.
