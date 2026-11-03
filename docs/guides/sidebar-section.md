# Guide: a sidebar section

A sidebar section is a small page of yours that sits in Houston's sidebar, below the agents, under
a heading. It's the place for something the person wants always in view: a note, a status, a
count, a log. Examples: [`hello`](../../addons/hello), the smallest add-on there is, and
[`scratchpad`](../../examples/scratchpad), a notepad that keeps what's typed.

## What the person sees

- A heading with your `symbol` and `title`, and a chevron. Clicking the heading folds the section
  away or opens it; Houston remembers which are folded across launches.
- Below it, your page: the sidebar's width, **180 points tall** (scaled with Houston's zoom), with
  rounded corners. It scrolls if your content is taller.
- The section is also listed in ⌘K's **Go to**; from there it opens in a full-sized sheet, as a
  [screen](screen.md) does.

Sections show in the order add-ons are listed, after Houston's own sidebar.

## 1. The manifest

```json
"contributes": {
  "sidebarSections": [
    { "id": "hello", "title": "Hello", "page": "pages/hello.html", "symbol": "hand.wave" }
  ]
}
```

Same fields as a screen: `id`, `title`, `page`, `symbol`. Up to 5 per add-on. See
[`sidebarSections`](../manifest.md#sidebarsections).

## 2. A page that fits

The sidebar is narrow and short. Design for about 220 × 180 points, in both light and dark mode:

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <style>
    :root { color-scheme: light dark; }
    html, body { margin: 0; height: 100%; }
    body { font: 12px -apple-system, system-ui, sans-serif; background: transparent; }
    textarea {
      box-sizing: border-box; width: 100%; height: calc(100% - 18px);
      border: 0; resize: none; background: transparent; color: inherit; font: inherit;
    }
    #status { font-size: 10px; opacity: 0.6; height: 18px; }
  </style>
</head>
<body>
  <textarea id="text" placeholder="Notes"></textarea>
  <div id="status"></div>
  <script>
    const $ = (id) => document.getElementById(id);
    let saving;
    $("text").oninput = () => {
      clearTimeout(saving);
      saving = setTimeout(async () => {
        await houston.call("storage.set", { key: "text", value: $("text").value });
        $("status").textContent = "Saved.";
      }, 400);
    };
    (async () => {
      $("text").value = (await houston.call("storage.get", { key: "text" })) || "";
    })();
  </script>
</body>
</html>
```

- Houston draws no background behind a page, so a transparent `body`, as above, shows the
  sidebar through it. Set `color` from `color-scheme` (as `inherit` does) so words read in both
  modes.
- Keep it to one idea. Put anything larger in a screen and add a [command](commands-and-shortcuts.md)
  that opens it.
- Don't rely on hover alone: the sidebar is often glanced at, not pointed at.
- Save as the person types, as above. The page is closed when the section is folded.

## 3. Its life

- The page loads when the section first shows, and again each time it's unfolded. A folded
  section's page is gone: it hears no events and runs no timers.
- Turning the add-on off removes the section at once; turning it on adds it back.
- While Houston is locked, calls and events are refused ("Houston is locked."). Show the message
  and try again later rather than going blank.

## Keeping it fresh

To update what a section shows:

- listen for [events](events.md), which reach it while it's unfolded; or
- poll with `setInterval`, gently (a minute, not a second), since every section page runs for as
  long as Houston's window is open.

The [`event-log`](../../examples/event-log) example is a sidebar section that does the first.
