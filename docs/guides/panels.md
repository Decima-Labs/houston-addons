# Guide: panels

A panel is a page of yours about one thing in Houston: an agent, a mission, a chat or a terminal
tab. The person opens it from that thing's menu, and your page learns which thing it was from
`houston.context`. Example: [`context-panels`](../../examples/context-panels), which has one panel
for each place.

## Where each place shows

| `place` | The menu item is in | `houston.context` |
|---|---|---|
| `agent` | The agent's own menu, wherever it shows: right-click on its pin in the sidebar or its row in the Agents list, and the ⋯ in its chat header | `{ agent }` |
| `chat` | The ⋯ menu in an agent's chat header | `{ agent }` |
| `mission` | Right-click on a mission's card on the board | `{ mission }` |
| `terminal` | Right-click on a terminal tab | `{ terminal }`, plus `agent` when the tab shows an agent |

Choosing the item opens your page in a sheet over the window, with **Done**, bringing the window
forward if it was hidden. The menu item's words are the panel's `title`.

## 1. The manifest

```json
"permissions": {
  "agents": ["all"],
  "houston": ["missions.read"]
},
"contributes": {
  "panels": [
    { "id": "agent-info", "title": "Agent Details", "page": "pages/context.html", "place": "agent" },
    { "id": "mission-info", "title": "Mission Details", "page": "pages/context.html", "place": "mission" }
  ]
}
```

- Several panels may share one page: it tells them apart by what's in `houston.context`.
- Up to 20 panels. See [`panels`](../manifest.md#panels).
- The context gives only an id. To learn more about the thing, the add-on needs the permission to
  read it: `agents` with that agent's provider, or `missions.read`.

## 2. Reading the context

```js
const context = houston.context; // frozen: { agent }, { mission }, { terminal } or {}

if (context.agent) {
  // The context has the id; agent.read needs the provider too, which agents.list gives.
  const agents = await houston.call("agents.list");
  const agent = agents.find((a) => a.id === context.agent);
  if (!agent) {
    show("Not an agent this add-on may see.");   // its provider isn't in permissions.agents
  } else {
    show(await houston.call("agent.read", { provider: agent.provider, id: agent.id }));
  }
} else if (context.mission) {
  const missions = await houston.call("houston.read", { scope: "missions" });
  show(missions.find((m) => m.id === context.mission));
} else {
  show("Open this from an agent or a mission.");
}
```

Always handle the empty context: a [command](commands-and-shortcuts.md) that `opens` a panel
opens it with `{}`.

## 3. Doing something with it

Once the page knows the agent, the usual next steps are:

- read its files: the agent's `directory` is its folder, see
  [files and agents](files-and-agents.md);
- send it a message with `agent.send`, which asks the person first;
- remember something about the mission with `houston.change` on `knowledge`, see
  [changing Houston](changing-houston.md).

A terminal's id names the tab only; there's no call to read a terminal's output.

## Good panels

- Make the title say what opens, about what: "Summarize for Standup", "Agent Details". It sits
  among Houston's own items in a busy menu.
- Show something useful straight away; the person opened it with one thing in mind.
- A panel is opened on purpose and closed after. For something always in view, use a
  [sidebar section](sidebar-section.md).
