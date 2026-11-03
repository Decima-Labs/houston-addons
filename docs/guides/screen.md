# Guide: a ⌘K screen

A screen is a full page of your own that opens over Houston's window. Example:
[`agents-screen`](../../examples/agents-screen), which lists the person's agents.

## What the person sees

- In **⌘K**, under **Go to**, a row with your `symbol` and `title`.
- On the add-on's row in **Settings › Agents**, a link **Open <title>**.
- Either opens your page in a sheet over the window, at least 640 × 440 points, with a **Done**
  button below it (Return closes it too).

The page is loaded fresh each time it opens and is gone when it closes. Keep anything that should
outlast it with [`storage`](network-storage-and-notifications.md#storage).

## 1. The manifest

```json
"permissions": {
  "agents": ["all"]
},
"contributes": {
  "screens": [
    { "id": "agents", "title": "Agents at a Glance", "page": "pages/agents.html", "symbol": "person.3" }
  ],
  "commands": [
    { "id": "open-agents", "title": "Open Agents at a Glance", "opens": "agents" }
  ]
}
```

- `id`: a word, unique among the add-on's screens, panels and sidebar sections.
- `symbol`: any [SF Symbol](https://developer.apple.com/sf-symbols/) name. Left out, a puzzle
  piece.
- The command is optional. It adds the screen to the View menu and lets you give it a shortcut;
  see [commands and shortcuts](commands-and-shortcuts.md).

Up to 10 screens per add-on. Several may share one `.html` file.

## 2. The page

A page is plain HTML, CSS and JavaScript, all inside the add-on folder. Everything has to be in
the folder: the [content policy](../security-model.md#5-the-page-a-sandbox-with-one-door) blocks
every outside URL, so no CDN scripts, web fonts or remote images.

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Agents at a Glance</title>
  <style>
    :root { color-scheme: light dark; }
    body { font: 13px -apple-system, system-ui, sans-serif; margin: 16px; }
    .error { color: #d33; }
  </style>
</head>
<body>
  <h1>Agents</h1>
  <button id="refresh">Refresh</button>
  <ul id="list"></ul>
  <p id="status"></p>
  <script>
    const $ = (id) => document.getElementById(id);
    function say(text, isError) {
      $("status").textContent = text;
      $("status").className = isError ? "error" : "";
    }
    async function call(name, args) {
      try {
        return await houston.call(name, args);
      } catch (e) {
        say(e.message || String(e), true);
        throw e;
      }
    }

    async function load() {
      const agents = await call("agents.list");
      $("list").replaceChildren(...agents.map((agent) => {
        const item = document.createElement("li");
        item.textContent = `${agent.name} (${agent.provider}, ${agent.state})`;
        return item;
      }));
      say(`${agents.length} agents.`);
    }
    $("refresh").onclick = load;
    load();
  </script>
</body>
</html>
```

Things to know:

- `houston` is there before your first script runs; no import, no waiting.
- Inline `<script>` and `<style>` are allowed, as are files beside the page:
  `<script src="app.js"></script>`, `<link rel="stylesheet" href="style.css">`,
  `<img src="icon.png">`. Relative URLs resolve inside the add-on.
- Images may also be `data:` URLs.
- Use `textContent`, not `innerHTML`, for anything that came from Houston or the network. Agent
  names, mission titles and fetched text are not yours.
- `color-scheme: light dark` lets the system draw form controls to match light or dark mode. The
  page doesn't get Houston's theme colours.
- `alert()`, `confirm()` and `prompt()` don't show. Put messages in the page.
- Links and navigations that leave the add-on are cancelled: nothing opens.

## 3. Opening a screen from elsewhere

Only the person opens screens: from ⌘K, from Settings, or from a command of yours that `opens` it.
A page can't open Houston's other places or a new window. It can link to another file in its own
folder (`<a href="settings.html">`), which loads in the same sheet, but changing what one page
shows is usually simpler.

## Screens vs. the other places

| You want | Use |
|---|---|
| Room to work, opened on purpose | A screen |
| Something always in view | A [sidebar section](sidebar-section.md) (180 points tall) |
| Something about one agent, mission, chat or terminal | A [panel](panels.md) |

A sidebar section is listed in ⌘K's **Go to** as well, and opens there as a screen-sized sheet.
