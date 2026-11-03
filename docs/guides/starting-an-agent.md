# Guide: starting an agent

An add-on can start a new agent, Claude Code, Codex or another, on a task in a folder, just as
the person would from Houston's composer. It asks first, every time. Example:
[`agent-starter`](../../examples/agent-starter).

## 1. Ask for the providers

```json
"permissions": {
  "agents": ["claude", "codex"]
}
```

`agents` lists the providers whose agents the add-on may see, message and start: `claude`,
`codex`, `cursor`, `gemini`, `opencode`, `aider`, or `all`. The same permission lets it list and
read those agents. See [permissions](../permissions.md#agents).

## 2. Start one

```js
say("Waiting for your yes…");
try {
  await houston.call("agent.start", {
    provider: "claude",                    // in permissions.agents, or "all" is
    repository: "/Users/jane/code/site",   // an absolute path to a folder that exists
    task: "Update the README's install steps.",
  });
  say("Started. It's on Agents.");
} catch (e) {
  say(e.message);
}
```

The person sees "Let Agent Starter start a Claude Code agent?" with "In ~/code/site: Update the
README's install steps." beneath, and **Allow Once** or **Don't Allow**. After a yes, Houston
starts it as its composer's Start would, and the agent shows in the sidebar and on Agents like any
other.

| Provider | Starts from an add-on |
|---|---|
| `claude`, `codex`, `gemini`, `opencode`, `aider`, `local` | Yes, if the person has it set up and on in Houston |
| `cursor` | No: "Cursor starts only from its composer." |

`local` is a local model ("Local model" in Houston); to start one, the add-on needs `"agents": ["all"]`.

## 3. Where the folder comes from

Houston doesn't tell a page the home folder, and there's no folder picker. Good sources:

- an agent's `directory`, from `agents.list`: "start another agent where this one works";
- the person, in a text field, remembered with `storage` so they type it once:

```js
const box = document.getElementById("repository");
box.value = (await houston.call("storage.get", { key: "repository" })) || "";
box.onchange = () => houston.call("storage.set", { key: "repository", value: box.value.trim() });
```

Check it starts with `/` before calling; Houston refuses anything else.

## Errors

| Message | Fix |
|---|---|
| agent.start needs provider: claude, codex or another agent. | Pass `provider`. |
| agent.start needs task. | Pass a non-empty `task`. |
| agent.start needs repository: a folder on this Mac. | Absolute path, to a folder that exists. |
| Codex isn't on in Houston. | The person hasn't that provider set up or turned on. Say so, and offer another. |
| Cursor starts only from its composer. | Pick another provider. |
| … wasn't allowed that. | The provider isn't in `permissions.agents`. |
| You said no. | The person chose Don't Allow. |

## After it starts

`agent.start` resolves with `true`, not the new agent. To follow it:

- list `agent.started` in `events` and listen: the new agent arrives with its `id` and `name`
  (see [events](events.md));
- or call `agents.list` again and look for it.

## Messaging an agent

To give an agent that's already running more to do, use `agent.send`:

```js
await houston.call("agent.send", { provider: agent.provider, id: agent.id, text: "Please run the tests again." });
// resolves "Sent to fix-login."
```

It asks first ("Let … send a message to a Claude Code agent?", with the message beneath), and goes
in as if typed into the agent's chat. A resting Claude agent is woken to take it. It rejects with
"There's no such agent.", or with why it couldn't be sent, such as "Houston can't send … a
message: it runs in a terminal Houston doesn't own."

## Good manners

- Start agents only from a button the person pressed, with the task in view.
- Write the task as you would for a colleague; the person reads it in the question.
- Don't start several at once: each is its own question, and its own running agent.
