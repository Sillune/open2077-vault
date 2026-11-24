# Redis.connect (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/redis-connect.md) · [Page web](https://open2077.net/docs/api/server/redis-connect)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

1 function.

## await

```lua
Redis.connect.await(options)
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

Returns: `A client handle. Options: `url`, `host`, `port`, `username`, `password`, `database`, `tls`, `tlsServerName`, `tlsVerify`, `protocol` (2 or 3), `name`, `connectTimeout`, `commandTimeout`, `maxReplyBytes`, `reconnect`.`

Documented in [server-api.md](../../01%20Documentation/server-api.md#redis) (line 2857).
