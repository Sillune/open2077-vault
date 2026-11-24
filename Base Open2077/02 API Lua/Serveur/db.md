# db (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/db.md) · [Page web](https://open2077.net/docs/api/server/db)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

11 functions.

## backup

```lua
db.backup(path, cb?)
```

`SERVER`: Available in server resources only.

Online copy into another file under `data/`; `true`.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `path` | `not specified` | required | — |
| `cb` | `not specified` | optional | — |

Returns: `Online copy into another file under `data/`; `true`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2827).

## batch

```lua
db.batch(sql, paramSets, cb?)
```

`SERVER`: Available in server resources only.

One statement per parameter table, in one transaction; total rows changed.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `not specified` | required | — |
| `paramSets` | `not specified` | required | — |
| `cb` | `not specified` | optional | — |

Returns: `One statement per parameter table, in one transaction; total rows changed.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2825).

## close

```lua
db.close(cb?)
```

`SERVER`: Available in server resources only.

Close after pending requests; open state.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cb` | `not specified` | optional | — |

Returns: `Close after pending requests; open state.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2828).

## execute

```lua
db.execute(sql, params?, cb?)
```

`SERVER`: Available in server resources only.

Runs every statement of a script; total rows changed.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `not specified` | required | — |
| `params` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: `Runs every statement of a script; total rows changed.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2824).

## insert

```lua
db.insert(sql, params?, cb?)
```

`SERVER`: Available in server resources only.

`last_insert_rowid()`.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `not specified` | required | — |
| `params` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: ``last_insert_rowid()`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2822).

## isOpen

```lua
db.isOpen()
```

`SERVER`: Available in server resources only.

Close after pending requests; open state.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

Returns: `Close after pending requests; open state.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2828).

## query

```lua
db.query(sql, params?, cb?)
```

`SERVER`: Available in server resources only.

Array of rows. `db.prepare` is an alias.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `not specified` | required | — |
| `params` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: `Array of rows. `db.prepare` is an alias.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2819).

## scalar

```lua
db.scalar(sql, params?, cb?)
```

`SERVER`: Available in server resources only.

First column of the first row or `nil`.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `not specified` | required | — |
| `params` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: `First column of the first row or `nil`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2821).

## single

```lua
db.single(sql, params?, cb?)
```

`SERVER`: Available in server resources only.

First row or `nil`.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `not specified` | required | — |
| `params` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: `First row or `nil`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2820).

## transaction

```lua
db.transaction(statements, cb?)
```

`SERVER`: Available in server resources only.

`true` on commit; `.await` returns `false, reason` on rollback.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `statements` | `not specified` | required | — |
| `cb` | `not specified` | optional | — |

Returns: ``true` on commit; `.await` returns `false, reason` on rollback.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2826).

## update

```lua
db.update(sql, params?, cb?)
```

`SERVER`: Available in server resources only.

Rows changed.

SQLite (also Open77.sqlite) opens SQLite files under the resource's own data/ directory.
Nothing is configured in server.jsonc: the resource chooses the file and its settings in
SQLite.open, the returned handle is the connection, and close ends it. Requires
sqlite.access. Requests use the same two forms as MySQL: a callback receiving
(result, err), or .await from a managed coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `sql` | `not specified` | required | — |
| `params` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: `Rows changed.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#sqlite) (line 2823).
