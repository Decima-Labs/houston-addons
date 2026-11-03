# Guide: a theme

A theme add-on gives Houston new colours. It's the simplest add-on there is: no page, no script,
no permissions. Example: [`theme-dusk`](../../examples/theme-dusk).

## What the person sees

On the add-on's row in **Settings › Agents**, once it's on, a link **Use Dusk Theme**. Clicking it
copies the theme into their own themes, as "Dusk (Dusk Theme)", and Houston draws with it at once.
From then on it's listed in **Settings › Appearance** with their other themes.

Because it's a copy:

- it stays after the add-on is turned off or uninstalled (they delete it in Settings › Appearance,
  as any theme of their own);
- a new version of your theme reaches them only when they click **Use … Theme** again, which
  replaces the copy rather than adding a second one;
- they can edit their copy in Houston's theme editor without touching your file.

## 1. Write the theme file

`themes/dusk.json`, in the same format Houston's theme editor exports (Settings › Appearance ›
Export…):

```json
{
  "houstonTheme": 1,
  "name": "Dusk",
  "base": "midnight",
  "accent": "#FF9E64",
  "background": "#1A1B26",
  "surface": "#202233",
  "text": "#E6E6F0",
  "attention": "#E0AF68",
  "green": "#9ECE6A",
  "red": "#F7768E",
  "cyan": "#7DCFFF",
  "pink": "#BB9AF7",
  "yellow": "#E0AF68",
  "syntax": {
    "keyword": "#BB9AF7",
    "string": "#9ECE6A",
    "comment": "#8C93BA",
    "number": "#FF9E64",
    "type": "#7DCFFF",
    "function": "#7AA2F7"
  }
}
```

| Field | Required | |
|---|---|---|
| `houstonTheme` | yes | `1`. Anything else is "from a newer Houston". |
| `name` | yes | Not empty. Houston shows the manifest's `title` instead, with the add-on's name. |
| `base` | yes | The built-in theme every colour you leave out comes from: `houston`, `graphite`, `midnight`, `paper`, `linen` or `mist`. |
| `accent` | | Buttons, selection, links. |
| `background` | | The window. The terminal, sunken fields and lines follow it. |
| `surface` | | Panels and the sidebar, and what's raised on them. |
| `text` | | The brightest words; the quieter tiers are stepped from it toward the background. |
| `attention` | | **Waiting**: an agent waiting on you, a check, a busy machine. |
| `green` | | **Passing**: working, passing, done. |
| `red` | | **Failing**: failures and error banners. |
| `cyan` | | **In progress**: starting and running. |
| `pink` | | Claude's agents. |
| `yellow` | | Codex's agents. |
| `selection`, `cursor` | | The terminal's selection and cursor. Left out, blended from the words and the accent. |
| `ansi` | | The terminal's 16 colours, normal then bright: exactly 16, or leave it out. |
| `syntax` | | The Code screen's colour per token: `keyword`, `string`, `comment`, `number`, `type`, `function`, `attribute`. |
| `colourSpace` | | `"displayP3"` for Display P3 colours. Left out, sRGB. |
| `backgroundTo`, `accentTo` | | Gradients: where the background ends at the bottom, and the accent's fill at its trailing edge. |

Every colour is `"#RRGGBB"`. One bad colour, an unknown `base`, or `ansi` with other than 16
makes the whole file unreadable.

The easiest way to make one is in Houston: build it in **Settings › Appearance**, choose
**Export…**, and put the file in your add-on. Houston shows a warning in the editor where a colour
reads short of its contrast bar (words and accent 4.5:1 on the background, terminal colours 3:1),
and lifts it to read when drawing. Check the warnings before you ship.

## 2. Name it in the manifest

```json
{
  "houston-addon": 1,
  "id": "io.github.you.theme-dusk",
  "name": "Dusk Theme",
  "version": "1.0.0",
  "description": "A dark theme with warm accents.",
  "author": { "name": "You", "key": "<your public key>" },
  "contributes": {
    "themes": [
      { "id": "dusk", "title": "Dusk", "file": "themes/dusk.json" }
    ]
  }
}
```

One add-on can carry up to 20 themes, each its own entry and file; each gets its own **Use**
link. See [`themes`](../manifest.md#themes).

## 3. Sign, check, install

```sh
houston-addon sign theme-dusk ~/.houston-addon/me.key
houston-addon check theme-dusk
```

Then install it as in the [getting started guide](../getting-started.md), turn it on in Settings ›
Agents (a theme asks for nothing, so the question lists nothing) and click **Use Dusk Theme**.

## When it goes wrong

The message shows on the add-on's row when you click **Use**:

| Message | Fix |
|---|---|
| Dusk isn't a Houston theme: That file isn't a theme Houston reads… | A colour isn't `#RRGGBB`, `base` isn't one of the six, a required field is missing, or `ansi` isn't 16 long. |
| Dusk isn't a Houston theme: That theme is from a newer Houston… | `houstonTheme` isn't `1`. |
| Dusk Theme's Dusk isn't among its signed files. | The file changed after signing, or the add-on's files no longer match. Sign again. |

Themes are colours only: no fonts, sizes or layout. Houston's theme editor is the reference for
what each colour touches.
