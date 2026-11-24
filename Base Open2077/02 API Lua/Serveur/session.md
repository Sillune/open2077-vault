# session (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/session.md) · [Page web](https://open2077.net/docs/api/server/session)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

11 functions.

## alive

```lua
session.alive()
```

`SERVER`: Available in server resources only.

End the check now / whether it is still undecided.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 815).

## createdAt

```lua
session.createdAt(…)
```

`SERVER`: Available in server resources only.

Opaque id, Unix seconds, and `checking`, `interactive`, `allowed`, `refused` or `expired`.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player. fields

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 805).

## door

```lua
session.door(…)
```

`SERVER`: Available in server resources only.

What the launcher reports about itself (unverified); `{ name, description, players, maxPlayers, whitelist }` when the check opened; `{ code, message, details }` while the door list refuses the player.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player. tables

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 807).

## endpoint

```lua
session.endpoint(…)
```

`SERVER`: Available in server resources only.

The source IP address.

Requires `players.preflight` + `players.identity.sensitive`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player. string

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 808).

## hide

```lua
session.hide()
```

`SERVER`: Available in server resources only.

`view` is an RML string or `{ document = ... | file = "ui/x.rml", data = { ... } }`. Not well-formed XML with an `rml` root, or over 64 KiB: `false, reason`.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 810).

## id

```lua
session.id(…)
```

`SERVER`: Available in server resources only.

Opaque id, Unix seconds, and `checking`, `interactive`, `allowed`, `refused` or `expired`.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player. fields

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 805).

## launcher

```lua
session.launcher(…)
```

`SERVER`: Available in server resources only.

What the launcher reports about itself (unverified); `{ name, description, players, maxPlayers, whitelist }` when the check opened; `{ code, message, details }` while the door list refuses the player.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player. tables

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 807).

## phase

```lua
session.phase(…)
```

`SERVER`: Available in server resources only.

Opaque id, Unix seconds, and `checking`, `interactive`, `allowed`, `refused` or `expired`.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player. fields

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 805).

## player

```lua
session.player(…)
```

`SERVER`: Available in server resources only.

The verified identity; the store and Discord identifiers come with a verified connect ticket.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player. `{ userId, name, publicKey, fingerprint, ticket, license?, steam?, gog?, epic?, discord? }`

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 806).

## refuse

```lua
session.refuse(message, { code?, view? }?)
```

`SERVER`: Available in server resources only.

The verdict; final.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `message` | `not specified` | required | — |
| `{ code?, view? }` | `table` | optional | — |

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 813).

## server

```lua
session.server(…)
```

`SERVER`: Available in server resources only.

What the launcher reports about itself (unverified); `{ name, description, players, maxPlayers, whitelist }` when the check opened; `{ code, message, details }` while the door list refuses the player.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player. tables

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 807).
