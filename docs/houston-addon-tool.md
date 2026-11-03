# The houston-addon tool

`houston-addon` makes your key, signs your add-on, and checks it exactly the way Houston will. It
lives in the Houston repository (`Sources/HoustonAddon`) and runs with `swift run`:

```sh
git clone https://github.com/Decima-Labs/houston ~/code/houston
cd ~/code/houston
swift run houston-addon <command> …
```

It builds on macOS. (Houston's Linux build leaves it out for now.) The first `swift run` builds
it, which takes a minute; later runs are quick. To skip `swift run` each time:

```sh
swift build -c release --product houston-addon
cp .build/release/houston-addon /usr/local/bin/   # or anywhere on your PATH
```

| Command | Does |
|---|---|
| `houston-addon new-key <file>` | Makes a key pair: the private key into `<file>`, the public key printed |
| `houston-addon sign <folder>` | Writes `<folder>/houston-addon.sig` with the key in `HOUSTON_ADDON_KEY`, then checks |
| `houston-addon check <folder>` | Says whether Houston will take the add-on, and what it asks for and adds |

It exits 0 on success and 1 with the reason on standard error otherwise, so it fits in scripts.

## new-key

```sh
houston-addon new-key ~/.houston-addon-key
```

```
Your add-on's author.key: 4n0pR…Q8=
```

- Writes the **private** key (an Ed25519 key, base64 of its 32 bytes) to the file, readable only by
  you (0600).
- Prints the **public** key. That goes in your manifest as `author.key`.
- Refuses to write over a file that exists: "`<file>` is there already: houston-addon never writes
  over a key."

One key can sign any number of add-ons. Keep it like any other secret: in your password manager or
a file outside any repository. Anyone with it can sign updates to your add-ons that Houston will
take as yours.

**If you lose it**, you can't update your add-ons under the same signer. Houston pins the first key
it sees for each add-on id, and refuses a later version signed by another key until the person
removes the old grant. You can still publish under a new id.

**If it leaks**, make a new key, publish under the new key (people have to remove the old grant to
take it, see [FAQ](faq.md#its-signed-by-someone-other-than-who-signed-it-before)), and tell your
users through the registry.

## sign

```sh
HOUSTON_ADDON_KEY="$(cat ~/.houston-addon-key)" houston-addon sign ~/code/my-addon
```

1. Reads the private key from the `HOUSTON_ADDON_KEY` environment variable. The tool never prints
   it. Passing it this way keeps it off the command line and out of your shell history.
2. Reads and validates `houston-addon.json`. Any [manifest](manifest.md) problem stops it here.
3. Refuses if `author.key` isn't this key's public key: "Its author.key isn't this key's public
   key, `<the right one>`." Copy the key it names into the manifest.
4. Hashes every file and writes `houston-addon.sig`.
5. Runs `check` and prints what it says.

Sign **after every change** to any file in the folder, the README included. Houston serves a page
only while its bytes are the ones you signed.

## check

```sh
houston-addon check ~/code/my-addon
```

```
My Add-on 1.0.0, signed by You.
It asks to:
  - Read files in ~/Notes
  - Reach api.example.com
  - Send you notifications
It adds:
  - Screens
  - Commands
```

`check` does what Houston does when it reads an installed add-on (`AddonSignature.verify`):

- the manifest is valid ([manifest reference](manifest.md));
- `houston-addon.sig` is there, at most 4 KB, and signed by the manifest's `author.key`;
- every file is the one that was signed, and no file was added or removed since;
- every page and theme file the manifest names is in the folder.

"It asks to" lists the permissions as the person reads them. Read it as they will: would you say
yes to that, from someone you don't know?

Run `check` on any add-on before you install it. It's how a reviewer checks a pull request.

## What gets signed

Everything in the folder, with these exceptions, at the top of the folder only:

- `.git`
- `.DS_Store`
- `houston-addon.sig` itself

and `.DS_Store` anywhere. Everything else is signed: `.claude-plugin/plugin.json`, the README,
skills, images, all of it.

The folder may not have:

- **links** (symbolic links), anywhere. Replace one with a copy of the file.
- anything other than files and folders (sockets, pipes).
- a path with an empty part, `.` or `..`, a part starting with `-`, a control character, or more
  than 300 characters.
- more than 2,000 files, or more than 50 MB in all.

Keep build output, `node_modules` and editor files out of the folder you sign. If you build your
pages from sources, build into the add-on folder and keep the sources elsewhere.

## The signature format

You don't need this to write an add-on, only to write your own signer or verifier.

The message signed is the text:

```
houston-addon-signature 1
<sha256 hex of file 1> <path 1>
<sha256 hex of file 2> <path 2>
…
```

one line per file, each ending in `\n`, sorted by path, paths relative to the add-on folder with
`/` between parts. It is signed with Ed25519 (`Curve25519.Signing` in CryptoKit).

`houston-addon.sig` is JSON:

```json
{
  "key" : "<author.key, base64>",
  "signature" : "<the Ed25519 signature, base64>"
}
```

Its `key` has to equal the manifest's `author.key`.

The source is `Sources/HoustonCore/AddonSignature.swift` and `AddonAuthoring.swift` in Houston.

## In a script or CI

```sh
#!/bin/sh
set -e
for addon in addons/*/; do
  houston-addon check "$addon"
done
```

Never put your private key in CI for a public repository. Sign on your own machine, commit the
signed folder, and let CI only `check`.
