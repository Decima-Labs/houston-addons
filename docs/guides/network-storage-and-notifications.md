# Guide: network, storage and notifications

Three small capabilities most add-ons want: reaching a web API, remembering things between
openings, and telling the person something. Example:
[`github-status`](../../examples/github-status), which checks githubstatus.com, remembers what it
saw, and notifies when it changes.

```json
"permissions": {
  "network": ["www.githubstatus.com"],
  "houston": ["notify"]
}
```

`storage` needs no permission.

## fetch

A page can't reach the network itself: its content policy sets `connect-src 'none'`, so the
browser's `fetch()`, `XMLHttpRequest`, `WebSocket` and `EventSource` all fail. Houston makes the
request instead, to the hosts the manifest names:

```js
const answer = await houston.call("fetch", { url: "https://www.githubstatus.com/api/v2/status.json" });
if (answer.status !== 200) throw new Error(`githubstatus.com answered ${answer.status}.`);
const status = JSON.parse(answer.body).status;
```

Sending data:

```js
await houston.call("fetch", {
  url: "https://api.example.com/v1/items",
  method: "POST",                         // GET (default), POST, PUT, PATCH, DELETE
  body: JSON.stringify({ name: "x" }),
  contentType: "application/json",
});
```

What to know:

- **Hosts.** `network` lists exact hosts (`api.example.com`) or every host under a domain
  (`*.example.com`, which doesn't include `example.com` itself). Lowercase names, not IP
  addresses. See [permissions](../permissions.md#network).
- **https only.** Anything else is "fetch needs an https url.".
- **Every status resolves.** `404` and `500` come back as `{ status, body }`; check `status`.
  Only a request that fails outright ("That didn't load: …") rejects.
- **Redirects aren't followed.** A `301` or `302` resolves with that status. Use the final URL,
  and list its host.
- **No headers but `Content-Type`.** No `Authorization`, no API keys in headers, no cookies.
  APIs that need a token aren't reachable from an add-on today.
- **No secrets.** Anyone can read an add-on's files, so a key in a page or a URL is public.
- **Text bodies**, at most 2 MB; 30 seconds before it times out.

## storage

Each add-on has its own small store of strings, kept on the Mac:

```js
await houston.call("storage.set", { key: "last", value: "All Systems Operational" });
const last = await houston.call("storage.get", { key: "last" });   // the string, or null
await houston.call("storage.set", { key: "last" });                // no value: forgets it
```

- Values are strings. Keep structure as JSON:

  ```js
  const prefs = JSON.parse((await houston.call("storage.get", { key: "prefs" })) || "{}");
  prefs.onlyRunning = true;
  await houston.call("storage.set", { key: "prefs", value: JSON.stringify(prefs) });
  ```

- At most 1 MB in all, per add-on; beyond that, "That's more than an add-on keeps: 1 MB in all.".
- Shared by all the add-on's pages: a screen and a sidebar section see the same keys. There's no
  event when another page changes a key, so read again when it matters.
- Kept in `~/Library/Application Support/Houston/addon-storage/<id>.json`, readable by the
  person's account only, under the add-on's `id`. It survives updates, and stays after the add-on
  is uninstalled.
- Works while the add-on is on and Houston is unlocked, like every call.
- `localStorage`, `sessionStorage`, IndexedDB and cookies **don't** keep anything: every page
  starts empty. Use `storage`.
- Not for secrets: it's a plain file.

## notify

```js
await houston.call("notify", { title: "GitHub", body: "Partial outage on Actions." });
```

- Shows as one of Houston's own notifications, titled with the add-on's name first:
  "GitHub Status: GitHub". The title is cut to 80 characters, the body to 300.
- Needs `houston: ["notify"]`. Doesn't ask: the permission is the yes.

Notify only on a change the person would want to hear about, and not more than they would want
it. The example remembers what it last said and notifies only when the status changes from
normal.

## Putting them together

```js
async function check() {
  const answer = await houston.call("fetch", { url: "https://www.githubstatus.com/api/v2/status.json" });
  if (answer.status !== 200) return say(`githubstatus.com answered ${answer.status}.`);
  const now = JSON.parse(answer.body).status;
  const before = await houston.call("storage.get", { key: "last" });
  if (now.indicator !== "none" && before !== now.description) {
    await houston.call("notify", { title: "GitHub", body: now.description });
  }
  await houston.call("storage.set", { key: "last", value: now.description });
}
```

A page only runs while it's open, so this checks when the screen opens or the person presses
**Check**. An add-on can't run in the background. To check on a schedule while the window is
open, put the page in a [sidebar section](sidebar-section.md) and use a gentle `setInterval`.
