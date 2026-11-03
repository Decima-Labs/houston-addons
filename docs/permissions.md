# Permissions reference

An add-on asks for what it needs in `permissions`. The person reads each one before they turn the
add-on on, and Houston's gate (`AddonGate` in `Sources/HoustonCore/AddonGate.swift`) checks every
call against what they granted.

```json
"permissions": {
  "files": ["~/Notes", "repository"],
  "network": ["api.example.com", "*.example.org"],
  "agents": ["claude", "codex"],
  "houston": ["missions.read", "knowledge.write", "notify"]
}
```

Ask for as little as you can. Every line is one more thing a stranger has to trust you with, and
reviewers ask why for each.

## What the person sees

When they turn the add-on on in **Settings › Agents › Add-ons**:

> **Turn on Standup?**
> Signed by Example Co. It asks to:
> • Read files in ~/Notes
> • Read the repository of an agent it may see
> • Reach api.example.com
> • Reach *.example.org
> • See your Claude Code agents
> • See your Codex agents
> • Read your missions
> • Change your memory, as you say yes
> • Send you notifications
> It still asks you before anything that needs a yes, and never works while Houston is locked.
>
> [Allow and Turn On] [Cancel]

The add-on's row then says "May read files in ~/Notes; …" for as long as it's installed.

## files

Lets its pages read files with `file.read`, and change them with `file.write` (each change asked).

| Value | The person reads | Allows |
|---|---|---|
| `~/Some/Folder` | Read files in ~/Some/Folder | Any file inside that folder of the person's home, at any depth |
| `repository` | Read the repository of an agent it may see | Files inside the folder the page passes as `repository` |

- A folder is written from the home folder: `~/Notes`, `~/code/site`. Not `~` alone, not `/…`,
  no trailing `/`, no `.` or `..` parts, at most 200 characters.
- Paths in calls are **absolute** (`/Users/jane/Notes/today.md`); a path with `.` or `..` parts is
  refused. Links are followed before the check, so a link inside `~/Notes` that points out of it
  is refused.
- Houston doesn't tell a page where the home folder is. Get absolute folders from `agents.list`
  (each agent's `directory`) or ask the person in your page.
- `repository` is meant for an agent's working folder: get it from the agent's `directory` and
  pass it as the `repository` argument. Ask for it together with `agents`. Houston refuses `/` as
  a repository.
- Reading is allowed outright. Writing always asks (see below).
- Files are at most 2 MB, read and written as UTF-8 text.

## network

Lets its pages call `fetch` on these hosts, over https only.

| Value | The person reads | Allows |
|---|---|---|
| `api.example.com` | Reach api.example.com | Exactly that host |
| `*.example.com` | Reach *.example.com | Any host **under** example.com (`api.example.com`, `a.b.example.com`), not `example.com` itself |

To reach both `example.com` and its subdomains, list both. Hosts are lowercase names with at
least two labels; IP addresses aren't hosts here. Redirects are never followed, so list the host
that answers, not one that redirects to it.

## agents

Lets its pages see agents, and, with the person's yes, message and start them.

| Value | The person reads |
|---|---|
| `all` | See all your agents |
| `claude` | See your Claude Code agents |
| `codex` | See your Codex agents |
| `cursor` | See your Cursor agents |
| `gemini` | See your Gemini CLI agents |
| `opencode` | See your OpenCode agents |
| `aider` | See your Aider agents |

`all` includes every provider, local models too. It allows:

- `agents.list`: the agents of the providers it may see (with any value at all).
- `agent.read`: one of them.
- `agent.send`: a message to one, **asked each time**.
- `agent.start`: a new agent of a provider it may see, **asked each time**.
- The agent events (`agent.started`, `agent.stopped`, `agent.asks`, `agent.message`), for agents of
  providers it may see, if it lists them in `contributes.events`.

## houston

Houston's own things.

| Value | The person reads | Allows |
|---|---|---|
| `missions.read` | Read your missions | `houston.read` with `scope: "missions"`; the `mission.moved` event |
| `missions.write` | Change your missions, as you say yes | `houston.change` with `scope: "missions"` (asked), and reading them |
| `knowledge.read` | Read your memory | `houston.read` with `scope: "knowledge"` |
| `knowledge.write` | Change your memory, as you say yes | `houston.change` with `scope: "knowledge"` (asked), and reading it |
| `schedules.read` | Read your schedules | `houston.read` with `scope: "schedules"`; the `schedule.ran` event |
| `schedules.write` | Change your schedules, as you say yes | `houston.change` with `scope: "schedules"` (asked), and reading them |
| `chat.read` | Read the chats of the agents it may see | Nothing yet: Houston grants it but has no call that reads chats today. Don't ask for it until one ships. |
| `notify` | Send you notifications | `notify` |

A `.write` includes its `.read`: ask for one, not both.

## No permission needed

- `storage.get` and `storage.set`: the add-on's own small store, which nothing else reads.
- `houston.context`: where a panel opened.
- `command.ran`: its own commands, if listed in `contributes.events`.
- Showing its pages, themes, commands and shortcuts.

## Asked each time

Whatever its permissions, these never happen without the person's yes, each time:

| Call | The person is asked | Under the question |
|---|---|---|
| `agent.send` | Let Standup send a message to a Claude Code agent? | The message |
| `agent.start` | Let Standup start a Codex agent? | In ~/code/site: the task |
| `file.write` | Let Standup change ~/Notes/today.md? | The text |
| `houston.change` missions | Let Standup change missions? | Move 20261102-ab12 to done |
| `houston.change` knowledge | Let Standup change memory? | The text |
| `houston.change` schedules | Let Standup change schedules? | Pause nightly-backup |

The buttons are **Allow Once** and **Don't Allow**. One question at a time per page: a second call
that asks, while one is waiting, is refused with "Standup is already waiting for your yes."

## The gate, in order

For every call, `AddonGate.decide`:

1. **Locked?** While Houston is locked, every call is refused: "Houston is locked." Events
   don't arrive either.
2. **On?** If the add-on isn't turned on, or its grant was for another author's key: "Standup is
   off."
3. **Granted?** It checks the call against what was granted **and** what the installed version
   still asks for: never more than both. Outside them: "Standup wasn't allowed that."
4. **Changes something?** Then it asks the person (above). After their yes it decides again, so
   if Houston locked or the add-on was turned off meanwhile, it doesn't go ahead: "Standup can't
   do that now."
5. Otherwise it goes ahead.

## Updates that ask for more

When a new version asks for something the person didn't grant, Houston keeps the add-on off and
its row says:

> Its new version asks to: Reach api.example.com. It stays off until you say yes.

Turning it on again asks for everything it asks for now. Updates that ask for less need nothing:
the gate never grants more than the version installed asks.

## Choosing well

- **Read before write.** If you only show missions, ask for `missions.read`, not `missions.write`.
- **Name providers.** If your add-on is about Claude Code agents, ask for `claude`, not `all`.
- **Name hosts.** `api.github.com`, not `*.github.com`.
- **Narrow folders.** `~/Notes/Standups`, not `~/Notes`, and never your whole home.
- Explain each permission in your README. Reviewers check that every one is used.
