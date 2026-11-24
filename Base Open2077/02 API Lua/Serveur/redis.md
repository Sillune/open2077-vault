# Redis (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/redis.md) · [Page web](https://open2077.net/docs/api/server/redis)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

8 functions.

## connect

```lua
Redis.connect(options, callback)
```

`SERVER`: Available in server resources only.

A client handle. Options: `url`, `host`, `port`, `username`, `password`, `database`, `tls`, `tlsServerName`, `tlsVerify`, `protocol` (2 or 3), `name`, `connectTimeout`, `commandTimeout`, `maxReplyBytes`, `reconnect`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `not specified` | required | — |
| `callback` | `not specified` | required | — |

Returns: `A client handle. Options: `url`, `host`, `port`, `username`, `password`, `database`, `tls`, `tlsServerName`, `tlsVerify`, `protocol` (2 or 3), `name`, `connectTimeout`, `commandTimeout`, `maxReplyBytes`, `reconnect`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2857).

## isError

```lua
Redis.isError(…)
```

`SERVER`: Available in server resources only.

Null sentinel inside tables, reply helpers, open connections.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Null sentinel inside tables, reply helpers, open connections.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2868).

## isNull

```lua
Redis.isNull(…)
```

`SERVER`: Available in server resources only.

Null sentinel inside tables, reply helpers, open connections.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Null sentinel inside tables, reply helpers, open connections.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2868).

## null

```lua
Redis.null(…)
```

`SERVER`: Available in server resources only.

Null sentinel inside tables, reply helpers, open connections.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Null sentinel inside tables, reply helpers, open connections.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2868).

## openCount

```lua
Redis.openCount()
```

`SERVER`: Available in server resources only.

Null sentinel inside tables, reply helpers, open connections.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Null sentinel inside tables, reply helpers, open connections.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2868).

## parseInfo

```lua
Redis.parseInfo(…)
```

`SERVER`: Available in server resources only.

Null sentinel inside tables, reply helpers, open connections.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Null sentinel inside tables, reply helpers, open connections.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2868).

## subscriber

```lua
Redis.subscriber(options, callback)
```

`SERVER`: Available in server resources only.

A pub/sub handle: `subscribe`, `psubscribe`, `ssubscribe` with a trailing handler, their `un` forms, `on`, `onState`, `close`.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `not specified` | required | — |
| `callback` | `not specified` | required | — |

Returns: `A pub/sub handle: `subscribe`, `psubscribe`, `ssubscribe` with a trailing handler, their `un` forms, `on`, `onState`, `close`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2867).

## toMap

```lua
Redis.toMap(…)
```

`SERVER`: Available in server resources only.

Null sentinel inside tables, reply helpers, open connections.

Redis (also Open77.redis) is a Redis client the resource drives itself. Nothing is configured
in server.jsonc: Redis.connect takes a redis:// / rediss:// URL or a table of options,
the returned handle is the connection, and close ends it. Requires redis.access. Commands use
the same two forms as MySQL: a callback receiving (result, err), or .await from a managed
coroutine.

Returns: `Null sentinel inside tables, reply helpers, open connections.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2868).
