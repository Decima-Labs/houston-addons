# Guide: events

An open page can hear what happens in Houston: agents starting, asking, finishing a turn and
stopping, missions moving, schedules running, and its own commands running. Example:
[`event-log`](../../examples/event-log), a sidebar section that logs every event it may hear.

## 1. Ask for the events

List each event in `contributes.events`, and ask for the permission to read what it's about:

```json
"permissions": {
  "agents": ["all"],
  "houston": ["missions.read", "schedules.read"]
},
"contributes": {
  "sidebarSections": [
    { "id": "event-log", "title": "Event Log", "page": "pages/log.html", "symbol": "list.bullet.rectangle" }
  ],
  "events": ["agent.started", "agent.stopped", "agent.asks", "agent.message", "mission.moved", "schedule.ran"]
}
```

| Event | Fires when | Data | Needs |
|---|---|---|---|
| `agent.started` | An agent comes to be running (starting, working, waiting, idle or open) from not running, including a new agent | `{ id, name, state }` | The agent's provider in `agents` |
| `agent.stopped` | A running agent becomes done, failed or stopped, or disappears | `{ id, name, state }` | The agent's provider in `agents` |
| `agent.asks` | An agent starts waiting for the person | `{ id, name, state, question }` (`question` may be missing) | The agent's provider in `agents` |
| `agent.message` | An agent goes from working to idle: it finished a turn | `{ id, name, state }` | The agent's provider in `agents` |
| `mission.moved` | A mission's status changes | `{ id, title, from, to }` | `missions.read` or `missions.write` |
| `schedule.ran` | A schedule runs | `{ id, name }` | `schedules.read` or `schedules.write` |
| `command.ran` | One of the add-on's own commands without `opens` runs | `{ id }`, the command's id | Nothing |

All values are strings. `state` is one of the agent states in
[`agents.list`](../bridge-api.md#agentslist); `from` and `to` are mission statuses (`backlog`,
`next`, `in-flight`, `done`, `closed`).

An unknown event name makes the manifest unreadable. An event the manifest doesn't list never
reaches the page, whatever it `houston.on`s.

## 2. Listen

```js
houston.on("agent.asks", (data) => {
  add(`${data.name} asks: ${data.question || "something"}`);
});

houston.on("mission.moved", (data) => {
  add(`“${data.title}” moved from ${data.from} to ${data.to}`);
});

function add(text) {
  const item = document.createElement("li");
  item.textContent = `${new Date().toLocaleTimeString()} ${text}`;
  document.getElementById("log").prepend(item);
}
```

- Call `houston.on` as often as you like; every handler for an event runs, in the order added.
- A handler that throws is caught; the other handlers still run.
- There's no `off`. Keep a flag in your handler if you need to stop acting.

## 3. When a page hears what

A page hears an event only while **all** of these hold:

1. it's in the manifest's `events`;
2. the add-on may read what it's about: an agent event for a Codex agent needs `codex` (or `all`)
   in `agents`; a mission event needs `missions.read` or `missions.write`;
3. the add-on is on and Houston isn't locked;
4. the page is open: a showing screen or panel, or an unfolded sidebar section.

So:

- **Nothing is replayed.** What happened before the page opened, or while it was folded or locked,
  is never told. When the page opens, read the current state with `agents.list` or `houston.read`,
  then listen for changes.
- An add-on that may see only Claude agents never hears about the others.
- A new mission isn't `mission.moved`; only a change of status on one that was there is.

## 4. How Houston notices

Houston compares what its agents, missions and schedules look like each time they change, and
tells listening pages the differences. In practice:

- An agent that goes from working to waiting gives `agent.asks`; from working to idle,
  `agent.message`; from idle to done, `agent.stopped`.
- Quick changes in between two looks can be merged: a state that lasted a moment may never be
  told. Don't count on seeing every step.
- The first page to start listening sets the baseline; it hears changes from then on.

## Doing something when an event comes

Common next steps, each covered elsewhere:

- send a [notification](network-storage-and-notifications.md#notify) when an agent asks;
- note it in [storage](network-storage-and-notifications.md#storage) to show a history across
  openings;
- read the agent with `agent.read` for its folder, or its mission with `houston.read`.

Anything that changes something, such as `agent.send` or `houston.change`, asks the person each
time. Don't do it from every event; let the person press a button.
