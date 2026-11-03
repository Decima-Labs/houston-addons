# Guide: files and agents

Reading the person's files, writing them with a yes, and seeing and messaging their agents. These
are the permissions people think hardest about, so ask for as little as you can. There's no
standalone example for files; the snippets below are complete, and
[`agents-screen`](../../examples/agents-screen) and
[`context-panels`](../../examples/context-panels) show the agent calls.

## Agents

### Ask

```json
"permissions": { "agents": ["claude", "codex"] }
```

Each provider listed lets the add-on list, read, message (asked) and start (asked) that provider's
agents. `all` is every provider, including ones Houston adds later. Prefer naming the ones you
need. See [permissions](../permissions.md#agents).

### List and read

```js
const agents = await houston.call("agents.list");
// Only the providers it may see:
// [{ id, provider: "claude", name: "fix-login", state: "working", directory: "/Users/jane/code/site" }]

const one = await houston.call("agent.read", { provider: "claude", id: agents[0].id });
```

`agent.read` needs both `provider` and `id`; a panel's `houston.context.agent` gives only the id,
so find the provider in `agents.list` first. `state` is `starting`, `working`, `waiting`, `idle`,
`open`, `done`, `failed` or `stopped`.

There's no call for an agent's transcript or its terminal output. For what an agent is doing,
use its `state` and the [events](events.md).

### Message

```js
await houston.call("agent.send", { provider: one.provider, id: one.id, text: "Please run the tests again." });
```

Asks first, every time, with the message beneath the question. See
[starting an agent](starting-an-agent.md#messaging-an-agent) for what it resolves with and how it
can fail.

## Files

### Ask

```json
"permissions": { "files": ["~/Notes"] }
```

| You want | Ask for |
|---|---|
| A folder of the person's, such as their notes | `"~/Notes"`: that folder and everything in it |
| The folder an agent works in | `"repository"`, together with `agents` |

Folders are written from the home folder: `~/Notes`, `~/code/site`, never `~` alone or an absolute
`/…` path. Up to 20. See [permissions](../permissions.md#files).

### Read

```js
const text = await houston.call("file.read", { path: "/Users/jane/Notes/today.md" });
```

- `path` is **absolute**, and may not have `.` or `..` parts.
- Links are followed before Houston checks the path is inside the folder, so a link in `~/Notes`
  that points out of it is refused.
- Text only, read as UTF-8, at most 2 MB. Binary files come back garbled; there's no binary read.
- Rejects with "Houston couldn't read that file." for a file that isn't there, as well as for one
  that can't be read.
- There's no directory listing. Read files whose names you know, or let the person type them.

### Write

```js
await houston.call("file.write", { path: "/Users/jane/Notes/today.md", text: "# Today\n" });
```

- Asks first: "Let Notes Helper change ~/Notes/today.md?", with the new text beneath.
- Replaces the whole file, atomically, or creates it if its folder exists. Folders aren't created.
- At most 2 MB of text. Rejects with "That's more than 2 MB." or "Houston couldn't write it: …".
- To append, read, add, and write the whole thing back.

### The home folder

Houston never tells a page where the home folder is, so you can't build `/Users/<name>/Notes`
yourself. Two ways round it:

1. **Ask once.** A text field where the person pastes the folder's path, kept with
   [`storage`](network-storage-and-notifications.md#storage).
2. **Start from an agent.** Each agent's `directory` is absolute; for `files: ["repository"]`,
   that's the folder to use (below).

### An agent's repository

```json
"permissions": {
  "agents": ["all"],
  "files": ["repository"]
}
```

```js
const [agent] = await houston.call("agents.list");
const readme = await houston.call("file.read", {
  path: agent.directory + "/README.md",
  repository: agent.directory,
});
```

Pass the agent's `directory` as `repository`, and a path inside it. A path outside the
`repository` given is refused, and `/` is never a repository. This is how a panel beside an agent
reads that agent's code:

```js
const agents = await houston.call("agents.list");
const agent = agents.find((a) => a.id === houston.context.agent);
const pkg = JSON.parse(await houston.call("file.read", {
  path: agent.directory + "/package.json",
  repository: agent.directory,
}));
```

## Asking well

- Name the folder you need, not a parent of it: `~/Notes/Standups`, not `~/Notes`, never
  `~/Documents`.
- If you only read, never call `file.write`; reviewers check that what a page does matches what
  it asks for.
- Show the person what you're about to write before the question does.
