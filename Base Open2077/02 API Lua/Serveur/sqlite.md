# SQLite (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/sqlite.md) · [Page web](https://open2077.net/docs/api/server/sqlite)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## exists

```lua
SQLite.exists(path)
```

`SERVER`: Available in server resources only.

File checks, deletion with companion files, library version, open handles.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `not specified` | required | — |

Returns: `File checks, deletion with companion files, library version, open handles.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2829).

## open

```lua
SQLite.open(path, options?, callback)
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
| `callback` | `not specified` | required | — |

Returns: `A database handle. Options: `readOnly`, `create`, `journal`, `synchronous`, `foreignKeys`, `busyTimeout`, `maxRows`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2818).

## openCount

```lua
SQLite.openCount()
```

`SERVER`: Available in server resources only.

File checks, deletion with companion files, library version, open handles.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

Returns: `File checks, deletion with companion files, library version, open handles.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2829).

## remove

```lua
SQLite.remove(path)
```

`SERVER`: Available in server resources only.

File checks, deletion with companion files, library version, open handles.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `not specified` | required | — |

Returns: `File checks, deletion with companion files, library version, open handles.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2829).

## version

```lua
SQLite.version()
```

`SERVER`: Available in server resources only.

File checks, deletion with companion files, library version, open handles.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

Returns: `File checks, deletion with companion files, library version, open handles.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2829).
