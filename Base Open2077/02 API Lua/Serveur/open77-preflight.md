# Open77.preflight (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-preflight.md) · [Page web](https://open2077.net/docs/api/server/open77-preflight)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

10 functions.

## broadcast

```lua
Open77.preflight.broadcast(patch)
```

`SERVER`: Available in server resources only.

A data patch for every open screen; returns the count.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `patch` | `not specified` | required | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 818).

## find

```lua
Open77.preflight.find(userId)
```

`SERVER`: Available in server resources only.

The live checks, one by id, the live one of an account.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 817).

## get

```lua
Open77.preflight.get(id)
```

`SERVER`: Available in server resources only.

The live checks, one by id, the live one of an account.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `not specified` | required | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 817).

## passed

```lua
Open77.preflight.passed(userId)
```

`SERVER`: Available in server resources only.

`{ at, age, sessionId }` when the account was allowed within `preflight.passSeconds`, else nil. Other resources read `player.preflight` at the gate.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 819).

## playerEnteredScope

```lua
Open77.preflight.playerEnteredScope(…)
```

`SERVER`: Available in server resources only.

Connection preflight: playerEnteredScope.

The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 822).

## playerLeftScope

```lua
Open77.preflight.playerLeftScope(…)
```

`SERVER`: Available in server resources only.

Connection preflight: playerLeftScope.

The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 822).

## server

```lua
Open77.preflight.server()
```

`SERVER`: Available in server resources only.

Configuration, live counts and counters / `{ name, description, players, maxPlayers, whitelist }` now.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 820).

## sessions

```lua
Open77.preflight.sessions()
```

`SERVER`: Available in server resources only.

The live checks, one by id, the live one of an account.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 817).

## status

```lua
Open77.preflight.status()
```

`SERVER`: Available in server resources only.

Configuration, live counts and counters / `{ name, description, players, maxPlayers, whitelist }` now.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 820).

## welcome

```lua
Open77.preflight.welcome(view | nil)
```

`SERVER`: Available in server resources only.

The anonymous screen on the server's page in the launcher.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `view | nil` | `not specified` | required | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 816).
