# r (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/r.md) · [Page web](https://open2077.net/docs/api/server/r)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

16 functions.

## batch

```lua
r.batch(…)
```

`SERVER`: Available in server resources only.

Array of replies; a failed command is an error object in its slot.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Array of replies; a failed command is an error object in its slot.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2862).

## call

```lua
r.call(name, args..., cb?)
```

`SERVER`: Available in server resources only.

Any command, name first.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `not specified` | required | — |
| `args...` | `not specified` | required | — |
| `cb` | `not specified` | optional | — |

Returns: `Any command, name first.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2859).

## close

```lua
r.close(cb?)
```

`SERVER`: Available in server resources only.

Close after pending commands; state; current database; connection changes.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cb` | `not specified` | optional | — |

Returns: `Close after pending commands; state; current database; connection changes.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2866).

## configGet

```lua
r.configGet(pattern, cb?)
```

`SERVER`: Available in server resources only.

A table `{ field = value }`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `pattern` | `not specified` | required | — |
| `cb` | `not specified` | optional | — |

Returns: `A table `{ field = value }`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2861).

## database

```lua
r.database()
```

`SERVER`: Available in server resources only.

Close after pending commands; state; current database; connection changes.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Close after pending commands; state; current database; connection changes.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2866).

## evalScript

```lua
r.evalScript(source, keys?, args?, cb?)
```

`SERVER`: Available in server resources only.

`EVALSHA` with the script cached per handle.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `source` | `not specified` | required | — |
| `keys` | `not specified` | optional | — |
| `args` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: ``EVALSHA` with the script cached per handle.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2865).

## hgetall

```lua
r.hgetall(key, cb?)
```

`SERVER`: Available in server resources only.

A table `{ field = value }`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `not specified` | required | — |
| `cb` | `not specified` | optional | — |

Returns: `A table `{ field = value }`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2861).

## hscanAll

```lua
r.hscanAll(…)
```

`SERVER`: Available in server resources only.

Walk a cursor with `match`, `count`, `type`, `limit`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Walk a cursor with `match`, `count`, `type`, `limit`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2864).

## isOpen

```lua
r.isOpen()
```

`SERVER`: Available in server resources only.

Close after pending commands; state; current database; connection changes.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Close after pending commands; state; current database; connection changes.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2866).

## multi

```lua
r.multi(commands, { watch }?, cb?)
```

`SERVER`: Available in server resources only.

`MULTI`/`EXEC`; `.await` returns the replies or `nil, reason`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `commands` | `not specified` | required | — |
| `{ watch }` | `table` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: ``MULTI`/`EXEC`; `.await` returns the replies or `nil, reason`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2863).

## onState

```lua
r.onState(handler)
```

`SERVER`: Available in server resources only.

Close after pending commands; state; current database; connection changes.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handler` | `not specified` | required | — |

Returns: `Close after pending commands; state; current database; connection changes.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2866).

## pipeline

```lua
r.pipeline(commands, cb?)
```

`SERVER`: Available in server resources only.

Array of replies; a failed command is an error object in its slot.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `commands` | `not specified` | required | — |
| `cb` | `not specified` | optional | — |

Returns: `Array of replies; a failed command is an error object in its slot.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2862).

## scanAll

```lua
r.scanAll(…)
```

`SERVER`: Available in server resources only.

Walk a cursor with `match`, `count`, `type`, `limit`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Walk a cursor with `match`, `count`, `type`, `limit`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2864).

## set

```lua
r.set(key, value, options?, cb?)
```

`SERVER`: Available in server resources only.

`SET` with `ex`, `px`, `exat`, `pxat`, `nx`, `xx`, `keepttl`, `get`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `key` | `not specified` | required | — |
| `value` | `not specified` | required | — |
| `options` | `not specified` | optional | — |
| `cb` | `not specified` | optional | — |

Returns: ``SET` with `ex`, `px`, `exat`, `pxat`, `nx`, `xx`, `keepttl`, `get`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2860).

## sscanAll

```lua
r.sscanAll(…)
```

`SERVER`: Available in server resources only.

Walk a cursor with `match`, `count`, `type`, `limit`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Walk a cursor with `match`, `count`, `type`, `limit`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2864).

## zscanAll

```lua
r.zscanAll(…)
```

`SERVER`: Available in server resources only.

Walk a cursor with `match`, `count`, `type`, `limit`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Walk a cursor with `match`, `count`, `type`, `limit`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2864).
