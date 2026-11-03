# Bridge API reference

Every add-on page has one global, `houston`, there before the page's first script runs:

```js
houston.call(name, args)   // → Promise: Houston's answer, or a rejection with its reason
houston.on(event, handler) // hear one of Houston's events
houston.context            // where the page opened: { agent, mission, terminal }
```

It's frozen: a page can't replace or wrap it. The source is `AddonPageModel.bridge` in
`Sources/Houston/AddonHost.swift`, and `AddonRequest.read` in `Sources/HoustonCore/AddonRuntime.swift`.

## houston.call

```js
try {
  const agents = await houston.call("agents.list");
} catch (error) {
  console.log(error.message); // "Standup wasn't allowed that."
}
```

- `args` is an object of named values. **Values are strings or numbers**; numbers arrive as their
  text, `true`/`false` arrive as `"1"`/`"0"`, and arrays, objects and `null` are dropped. Where a
  call takes a yes/no (`paused`), pass the string `"true"` or `"false"`.
- The promise resolves with the answer (below) or rejects with an `Error` whose `message` is
  Houston's reason, written to show the person as it is.
- A call that changes something waits for the person's **Allow Once** or **Don't Allow**, which
  may take a while. Don't time it out; show that you're waiting.
- Only the page's own top frame can call. (Pages can't have frames anyway.)

### Calls at a glance

| Call | Needs | Asks? | Resolves with |
|---|---|---|---|
| [`agents.list`](#agentslist) | `agents` | | `Agent[]` |
| [`agent.read`](#agentread) | `agents` with its provider | | `Agent` |
| [`agent.send`](#agentsend) | `agents` with its provider | yes | a string |
| [`agent.start`](#agentstart) | `agents` with its provider | yes | `true` |
| [`file.read`](#fileread) | `files` | | a string |
| [`file.write`](#filewrite) | `files` | yes | `true` |
| [`fetch`](#fetch) | `network` | | `{status, body}` |
| [`houston.read`](#houstonread) | `houston` `*.read` or `*.write` | | an array |
| [`houston.change`](#houstonchange) | `houston` `*.write` | yes | `true` |
| [`notify`](#notify) | `houston: notify` | | `true` |
| [`storage.get`](#storageget) | nothing | | a string or `null` |
| [`storage.set`](#storageset) | nothing | | `true` |

Every call can also reject with the [gate's reasons](#errors-every-call-can-have).

---

### agents.list

The agents of the providers it may see.

```js
const agents = await houston.call("agents.list");
// [{ id: "a1b2…", provider: "claude", name: "fix-login", state: "working", directory: "/Users/jane/code/site" }]
```

An `Agent`:

| Field | |
|---|---|
| `id` | Houston's id for it |
| `provider` | `claude`, `codex`, `cursor`, `gemini`, `opencode`, `aider` or `local` |
| `name` | Its name, as the sidebar shows it |
| `state` | `starting`, `working`, `waiting` (for the person), `idle`, `open`, `done`, `failed` or `stopped` |
| `directory` | The folder it works in, absolute |

### agent.read

One agent, by its provider and id.

```js
const agent = await houston.call("agent.read", { provider: "claude", id });
```

Rejects with "There's no such agent." when there isn't one with that provider and id.

### agent.send

Sends a message to an agent, as if typed into its chat. Asks first:
"Let Standup send a message to a Claude Code agent?", with the message beneath.

```js
await houston.call("agent.send", { provider: "claude", id, text: "Please run the tests again." });
```

Resolves with Houston's words on how it went. Rejects with "agent.send needs text.", "There's
no such agent.", or why it couldn't be sent.

### agent.start

Starts a new agent on a task, in a folder, as the composer's Start does. Asks first:
"Let Standup start a Codex agent?", with "In ~/code/site: the task" beneath.

```js
await houston.call("agent.start", {
  provider: "codex",
  repository: "/Users/jane/code/site",
  task: "Update the README's install steps.",
});
```

| Arg | |
|---|---|
| `provider` | `claude`, `codex`, `gemini`, `opencode`, `aider` or `local`. Also needs that provider in `permissions.agents` (or `all`). |
| `repository` | An absolute path to a folder that exists |
| `task` | What the agent should do |

Rejects with:

- "agent.start needs provider: claude, codex or another agent."
- "agent.start needs task."
- "agent.start needs repository: a folder on this Mac."
- "Codex isn't on in Houston." when the person hasn't that provider set up or turned on.
- "Cursor starts only from its composer." for a provider Houston can't start directly.

See the [guide](guides/starting-an-agent.md).

### file.read

Reads a text file inside a folder it was granted.

```js
const text = await houston.call("file.read", { path: "/Users/jane/Notes/today.md" });
const readme = await houston.call("file.read", { path: agent.directory + "/README.md", repository: agent.directory });
```

| Arg | |
|---|---|
| `path` | Absolute. No `.` or `..` parts. |
| `repository` | For `files: ["repository"]`: the agent folder the path is in |

Resolves with the file's text (UTF-8). Rejects with "Houston couldn't read that file." or "That
file is more than 2 MB."

### file.write

Writes a text file inside a folder it was granted, replacing it. Asks first:
"Let Standup change ~/Notes/today.md?", with the text beneath.

```js
await houston.call("file.write", { path: "/Users/jane/Notes/today.md", text: "# Today\n" });
```

Same `path` and `repository` as `file.read`, plus `text`. Rejects with "That's more than 2 MB." or
"Houston couldn't write it: …".

### fetch

An https request to a host it may reach.

```js
const { status, body } = await houston.call("fetch", { url: "https://api.example.com/v1/items" });
const created = await houston.call("fetch", {
  url: "https://api.example.com/v1/items",
  method: "POST",
  body: JSON.stringify({ name: "x" }),
  contentType: "application/json",
});
const items = JSON.parse(body);
```

| Arg | |
|---|---|
| `url` | An `https://` URL on a host in `permissions.network` |
| `method` | `GET` (the default), `POST`, `PUT`, `PATCH` or `DELETE` |
| `body` | The request body, as text |
| `contentType` | Its `Content-Type` |

- Resolves with `{ status, body }` for any HTTP answer, `404` and `500` included: check `status`.
  `body` is the response as text, at most 2 MB.
- No cookies are sent or kept, nothing is cached, and **redirects aren't followed** (a redirect
  resolves with its 3xx status).
- No other headers can be set, so APIs that need an `Authorization` header aren't reachable yet.
  Never put a secret in a page or a URL: anyone can read your add-on's files.
- Times out after 30 seconds.
- Rejects with "fetch needs an https url.", "fetch can't HEAD.", or "That didn't load: …" when the
  request itself fails.

A page's own `fetch()`, `XMLHttpRequest` and `WebSocket` are blocked by its content policy:
`houston.call("fetch")` is the only way out.

### houston.read

Houston's missions, memory or schedules.

```js
const missions = await houston.call("houston.read", { scope: "missions" });
```

| `scope` | Needs | Resolves with |
|---|---|---|
| `missions` | `missions.read` or `missions.write` | `[{ id, title, status, priority }]`: `status` is `backlog`, `next`, `in-flight`, `done` or `closed`; `priority` is `low`, `normal`, `high` or `urgent` |
| `knowledge` | `knowledge.read` or `knowledge.write` | The first 500 memories: `[{ id, kind, title }]` |
| `schedules` | `schedules.read` or `schedules.write` | `[{ id, name, paused }]`, `paused` a boolean |

Rejects with "houston.read needs scope: missions, knowledge or schedules."

### houston.change

Changes Houston's missions, memory or schedules. Asks first: "Let Standup change missions?"

```js
await houston.call("houston.change", { scope: "missions", id: "20261102-ab12", status: "done" });
await houston.call("houston.change", { scope: "knowledge", text: "The deploy key rotates on Mondays.", category: "gotcha" });
await houston.call("houston.change", { scope: "schedules", id: "nightly-backup", paused: "true" });
```

| `scope` | Args | Under the question | Rejects with |
|---|---|---|---|
| `missions` | `id`, `status` (as above) | Move 20261102-ab12 to done | "Changing missions needs id and status.", "There's no such mission." |
| `knowledge` | `text`, `category`: `session`, `decision` (the default), `gotcha` or `incident` | The text | "Changing memory needs text." |
| `schedules` | `id`, `paused`: `"true"` or `"false"` | Pause nightly-backup / Resume nightly-backup | "Changing schedules needs id and paused: true or false.", "There's no such schedule." |

A moved mission notes it was moved "by Standup"; a memory notes it came from the add-on. See the
[guide](guides/changing-houston.md).

### notify

A notification, as Houston's own.

```js
await houston.call("notify", { title: "Build broke", body: "main failed on CI." });
```

The title shows as "Standup: Build broke", cut to 80 characters; the body to 300.

### storage.get

```js
const value = await houston.call("storage.get", { key: "notes" }); // a string, or null
```

### storage.set

```js
await houston.call("storage.set", { key: "notes", value: JSON.stringify(notes) });
await houston.call("storage.set", { key: "notes" }); // no value: forgets it
```

Each add-on has its own keys and values, strings only, at most 1 MB in all, kept on the Mac in
`~/Library/Application Support/Houston/addon-storage/<id>.json`, readable by the person's account
alone. Uninstalling the add-on doesn't delete them. Rejects with "storage.set needs key." or "That's more than an add-on keeps:
1 MB in all." Store JSON as text if you need structure.

`localStorage`, `sessionStorage`, IndexedDB and cookies **don't persist**: every page starts
empty. Use `storage`.

---

## houston.on

```js
houston.on("agent.stopped", (data) => {
  console.log(`${data.name} stopped (${data.state})`);
});
```

A page hears an event only if:

1. the event is in its manifest's `contributes.events`;
2. it may read what the event is about (below);
3. the add-on is on and Houston isn't locked;
4. the page is open. (A closed page, or a folded sidebar section, hears nothing.)

| Event | Data | Needs |
|---|---|---|
| `agent.started` | `{ id, name, state }` | The agent's provider in `agents` |
| `agent.stopped` | `{ id, name, state }` | The agent's provider in `agents` |
| `agent.asks` | `{ id, name, state, question }` | The agent's provider in `agents` |
| `agent.message` | `{ id, name, state }` | The agent's provider in `agents` |
| `mission.moved` | `{ id, title, from, to }` | `missions.read` or `missions.write` |
| `schedule.ran` | `{ id, name }` | `schedules.read` or `schedules.write` |
| `command.ran` | `{ id }` | Nothing: its own commands only |

All data values are strings. What each means, and when it fires, is in the
[events guide](guides/events.md). A handler that throws is ignored; the others still run.

## houston.context

Where the page opened. Empty (`{}`) for screens and sidebar sections; for [panels](guides/panels.md):

| Panel `place` | `houston.context` |
|---|---|
| `agent` | `{ agent }` |
| `chat` | `{ agent }` |
| `mission` | `{ mission }` |
| `terminal` | `{ terminal }`, and `agent` when the tab shows one |

`agent` is the agent's `id` (find its provider in `agents.list`), `mission` the mission's id, and
`terminal` the terminal tab's id. Each is a string, and frozen.

## Errors every call can have

| Message | Means |
|---|---|
| Houston is locked. | The person locked Houston. Try again after they unlock. |
| Standup is off. | The add-on isn't turned on (or its grant is for another signer). |
| Standup wasn't allowed that. | Outside what it asked for and was granted. |
| You said no. | The person chose Don't Allow. |
| Standup is already waiting for your yes. | Another call from this page is waiting on the person. |
| Standup can't do that now. | Houston locked, or the add-on was turned off, while the person was deciding. |
| Houston has no call X. | Misspelled call name. |
| X needs Y. | A required argument is missing or empty. |
| The page closed. | The page closed before the answer. |
| This page can't reach Houston. | The call didn't come from the add-on's own page. |

## A helper

The examples share this:

```js
async function call(name, args) {
  try {
    return await houston.call(name, args);
  } catch (error) {
    say(error.message || String(error));
    throw error;
  }
}
function say(text) {
  document.getElementById("status").textContent = text;
}
```
