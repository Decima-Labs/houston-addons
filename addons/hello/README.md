# hello

A starting point for a Houston add-on. Copy this folder, then:

1. Make your own key, once: `swift run houston-addon new-key ~/.houston-addon-key`. Keep the file; it is your signature.
2. Put the public key it prints in `houston-addon.json` as `author.key`, with your own `id`, `name` and `author.name`.
3. Change the pages, commands and skills, and ask for only the permissions you need.
4. Sign it: `HOUSTON_ADDON_KEY="$(cat ~/.houston-addon-key)" swift run houston-addon sign <folder>`. Sign again after every change.
5. Check it as Houston will: `swift run houston-addon check <folder>`.

See `docs/architecture/addons.md` for everything an add-on can add and ask for.
