# SQLite.open (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/sqlite-open.md) · [Page web](https://open2077.net/docs/api/server/sqlite-open)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

1 function.

## await

```lua
SQLite.open.await(path, options?)
```

`SERVER`: Available in server resources only.

A database handle. Options: `readOnly`, `create`, `journal`, `synchronous`, `foreignKeys`, `busyTimeout`, `maxRows`.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `not specified` | required | — |
| `options` | `not specified` | optional | — |

Returns: `A database handle. Options: `readOnly`, `create`, `journal`, `synchronous`, `foreignKeys`, `busyTimeout`, `maxRows`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2818).
