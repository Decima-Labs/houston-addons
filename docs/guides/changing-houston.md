# Guide: changing Houston

With the person's yes each time, an add-on can move a mission, pause or resume a schedule, and
write a memory. All three go through one call, `houston.change`. Example:
[`board-tools`](../../examples/board-tools), a screen that does all three.

## 1. Ask for the scopes

```json
"permissions": {
  "houston": ["missions.write", "schedules.write", "knowledge.write"]
}
```

| Scope | Read with | Change with |
|---|---|---|
| Missions | `missions.read` or `missions.write` | `missions.write` |
| Schedules | `schedules.read` or `schedules.write` | `schedules.write` |
| Memory | `knowledge.read` or `knowledge.write` | `knowledge.write` |

A `*.write` permission includes reading, so don't list both. Ask only for what you change: the
person reads each line in the question before turning the add-on on ("Change your missions, as you say yes").
See [permissions](../permissions.md#houston).

## 2. Read first

```js
const missions = await houston.call("houston.read", { scope: "missions" });
// [{ id: "20261102-ab12", title: "Fix login", status: "in-flight", priority: "high" }]

const schedules = await houston.call("houston.read", { scope: "schedules" });
// [{ id: "nightly-backup", name: "Nightly backup", paused: false }]

const memories = await houston.call("houston.read", { scope: "knowledge" });
// [{ id: "…", kind: "gotcha", title: "The deploy key rotates on Mondays" }], at most 500
```

Mission `status` is `backlog`, `next`, `in-flight`, `done` or `closed`; `priority` is `low`,
`normal`, `high` or `urgent`.

## 3. Change, with a yes

Every `houston.change` shows the person a question in Houston's window, with **Allow Once** and
**Don't Allow**. The promise waits for the answer:

```js
async function change(args) {
  say("Waiting for your yes…");
  try {
    await houston.call("houston.change", args);
    say("Done.");
  } catch (e) {
    say(e.message);        // "You said no.", "There's no such mission.", …
  }
}

// Move a mission. Question: "Let Board Tools change missions?" / "Move 20261102-ab12 to done"
change({ scope: "missions", id: "20261102-ab12", status: "done" });

// Pause or resume a schedule. paused is the string "true" or "false".
change({ scope: "schedules", id: "nightly-backup", paused: "true" });

// Write a memory. category: decision (the default), gotcha, incident or session.
change({ scope: "knowledge", text: "The deploy key rotates on Mondays.", category: "gotcha" });
```

| Scope | Arguments | The person reads beneath the question |
|---|---|---|
| `missions` | `id`, `status` | Move *id* to *status* |
| `schedules` | `id`, `paused` | Pause *id* / Resume *id* |
| `knowledge` | `text`, `category` | The text |

What Houston records:

- A moved mission notes it was moved by your add-on, by name.
- A memory notes it came from your add-on.

## Things to get right

- **One question at a time.** While one call from a page waits for the person, another that asks
  is refused with "… is already waiting for your yes." Disable your buttons while waiting.
- **No batch changes.** Moving ten missions is ten questions. Design for one deliberate change at
  a time, made from a button the person pressed, never from a timer or an event.
- **Read again after.** Refresh your list after a change, and after a no, so it shows what's true.
- **"Allow Once" means once.** There's no "always allow"; that's by design.
- **Locks and turning off.** If Houston locks, or the add-on is turned off, while the question is
  showing, the call rejects with "… can't do that now." even after a yes.
- **Booleans are strings.** `paused: true` arrives as `"1"` and is refused; pass `"true"`.

Missions can't be created, renamed or deleted, schedules can't be created or run, and memories
can't be edited or deleted through the bridge today.
