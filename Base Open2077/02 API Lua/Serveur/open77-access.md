# Open77.access (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-access.md) · [Page web](https://open2077.net/docs/api/server/open77-access)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

24 functions.

## allow

```lua
Open77.access.allow(userId, [label])
```

`NETWORK`: Uses the network backend.

Adds an identity to the built-in whitelist.

Requires `players.access`. The optional label is operator-facing text shown beside the entry in the console and the admin panel; it is not an identity.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string` | required | — |
| `label` | `string` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- /whitelist add <userId> <label>
RegisterCommand("whitelist-add", function(source, args)
    Open77.access.allow(args[1], table.concat(args, " ", 2))
end, true)
```

[Connection control guide](../../01%20Documentation/connection-control.md)

Registered in `LuaResourceRuntime.cs` (line 8140) as `LuaResourceRuntime.Bootstrap`.

## approve

```lua
Open77.access.approve(userId, label?, actor?)
```

`SERVER`: Available in server resources only.

Answer a request: `approve` whitelists the identity and removes the request; `deny` keeps it as `denied` with the note for the player. `false, "request_not_found"` / `"request_not_pending"`.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |
| `label` | `not specified` | optional | — |
| `actor` | `not specified` | optional | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 778).

## ban

```lua
Open77.access.ban(userId, [reason], [seconds], [name])
```

`NETWORK`: Uses the network backend.

Bans an identity on the built-in list and disconnects them if they are online.

Requires `players.access`. `seconds` nil or 0 makes the ban permanent. `name` is the display name recorded with the entry, so a later reader knows who was banned without resolving the identity again. Unlike `Open77.players.ban`, this writes the *server's own* list rather than submitting a device ban to the master.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string` | required | — |
| `reason` | `string` | optional | — |
| `seconds` | `number` | optional | — |
| `name` | `string` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A one-day local ban from an in-game command
RegisterCommand("tempban", function(source, args)
    local target = tonumber(args[1])
    local identity = target and Open77.players.identity(target)
    if identity then Open77.access.ban(identity.userId, table.concat(args, " ", 2), 86400, identity.name) end
end, true)
```

[Connection control guide](../../01%20Documentation/connection-control.md)

Registered in `LuaResourceRuntime.cs` (line 8142) as `LuaResourceRuntime.Bootstrap`.

## deny

```lua
Open77.access.deny(userId, note?, actor?)
```

`SERVER`: Available in server resources only.

Answer a request: `approve` whitelists the identity and removes the request; `deny` keeps it as `denied` with the note for the player. `false, "request_not_found"` / `"request_not_pending"`.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |
| `note` | `not specified` | optional | — |
| `actor` | `not specified` | optional | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 778).

## disallow

```lua
Open77.access.disallow(userId)
```

`NETWORK`: Uses the network backend.

Takes an identity off the built-in whitelist.

Requires `players.access`. It does not disconnect a player who is already in: combine it with `Open77.players.kick` when that is the intent.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Remove from the whitelist and kick if they are online
Open77.access.disallow(userId)
for _, playerId in ipairs(Open77.players.all()) do
    if Open77.players.identifier(playerId) == userId then Open77.players.kick(playerId, "Whitelist removed.") end
end
```

[Connection control guide](../../01%20Documentation/connection-control.md)

Registered in `LuaResourceRuntime.cs` (line 8141) as `LuaResourceRuntime.Bootstrap`.

## forgetRequest

```lua
Open77.access.forgetRequest(userId, actor?)
```

`SERVER`: Available in server resources only.

Remove a request, pending or denied. `true` when there was one.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |
| `actor` | `not specified` | optional | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 779).

## getBan

```lua
Open77.access.getBan(userId)
```

`SERVER`: Available in server resources only.

One record: the active ban (`{ userId, name, reason, bannedBy, bannedAt, bannedAtUnix, expiresAt?, expiresAtUnix?, permanent }`) or the whitelist entry (`{ userId, label, addedAt, addedBy }`), else `nil`.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 773).

## getEntry

```lua
Open77.access.getEntry(userId)
```

`SERVER`: Available in server resources only.

One record: the active ban (`{ userId, name, reason, bannedBy, bannedAt, bannedAtUnix, expiresAt?, expiresAtUnix?, permanent }`) or the whitelist entry (`{ userId, label, addedAt, addedBy }`), else `nil`.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 773).

## getRequest

```lua
Open77.access.getRequest(userId)
```

`SERVER`: Available in server resources only.

Whitelist requests, pending first: `{ userId, name, message, status = "pending" | "denied", requestedAt, requestedAtUnix, source, note, decidedBy?, decidedAt? }`.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 776).

## isAllowed

```lua
Open77.access.isAllowed(userId)
```

`NETWORK`: Uses the network backend.

Whether an identity is on the built-in whitelist.

Requires `players.access`. Reads the built-in list only; it does not consider a resource's own gate logic in `onPlayerConnecting`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string` | required | — |

Returns: `boolean, or false`, `reason`

```lua
-- Show a "VIP" tag in chat for whitelisted players
if Open77.access.isAllowed(userId) then name = "[VIP] " .. name end
```

[Connection control guide](../../01%20Documentation/connection-control.md)

Registered in `LuaResourceRuntime.cs` (line 8137) as `LuaResourceRuntime.Bootstrap`.

## isBanned

```lua
Open77.access.isBanned(userId)
```

`NETWORK`: Uses the network backend.

Whether an identity is on the built-in ban list.

Requires `players.access`. An expired temporary ban reads as not banned.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string` | required | — |

Returns: `boolean, or false`, `reason`

```lua
-- Refuse a friend invite to a banned account
if Open77.access.isBanned(friendUserId) then
    return Open77.chat.send(source, "That account is banned here.")
end
```

[Connection control guide](../../01%20Documentation/connection-control.md)

Registered in `LuaResourceRuntime.cs` (line 8138) as `LuaResourceRuntime.Bootstrap`.

## onAccessChanged

```lua
Open77.access.onAccessChanged(userId, change, actor)
```

`SERVER`: Available in server resources only.

The door list changed, whoever changed it: Warden (`warden:<user>`), the console (`console`), a resource (`resource:<name>`) or an expiry (empty actor). `change` is `ban`, `unban`, `ban_expired`, `allow`, `disallow`, `whitelist_on`, `whitelist_off`, `request`, `request_approved`, `request_denied` or `request_removed`; `userId` is empty for the whitelist switch. Delivered on the tick after the change.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |
| `change` | `not specified` | required | — |
| `actor` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 780).

## onPreflight

```lua
Open77.access.onPreflight(session)
```

`SERVER`: Available in server resources only.

A launcher opened a check and the door list admits the player. Decide with `session:allow` / `session:refuse`, or `session:hold()` to wait for the player. A handler error, or no decision within `preflight.decisionSeconds`, allows the player. Raised again on the same `session` when a refused player is admitted while the check is open.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `session` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 800).

## onPreflightEnd

```lua
Open77.access.onPreflightEnd(session, reason)
```

`SERVER`: Available in server resources only.

`allowed`, `refused`, `cancelled`, `expired`, `replaced` or `resource_stopped`.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `session` | `not specified` | required | — |
| `reason` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 803).

## onPreflightEvent

```lua
Open77.access.onPreflightEvent(session, name, data, inputs)
```

`SERVER`: Available in server resources only.

A button or Enter in the screen. `data = { args = { ... } }` or nil; `inputs` maps input ids to their values. Untrusted.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `session` | `not specified` | required | — |
| `name` | `not specified` | required | — |
| `data` | `not specified` | required | — |
| `inputs` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 802).

## onPreflightJoined

```lua
Open77.access.onPreflightJoined(session, playerId)
```

`SERVER`: Available in server resources only.

The player who passed this check finished the in-game handshake. Once per check.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `session` | `not specified` | required | — |
| `playerId` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 804).

## onPreflightRefused

```lua
Open77.access.onPreflightRefused(session, code, message, details)
```

`SERVER`: Available in server resources only.

The door list refuses this player (`banned` or `not_whitelisted`). Present it with `session:refuse(message, { view })` or `session:show` + `session:hold` (an application form); `session:allow()` answers `false, "door_refused"`. Without a handler the plain refusal is immediate.

Requires `players.preflight`. The launcher's server check, before the game starts: one resource (preflight.resource in
server.jsonc, open77_preflight by default) decides it. The server guide is
Connection preflight, the markup Building preflight screens
and the full Lua reference Preflight in Lua. Every function returns
false, "not_preflight_resource" to any other resource, and a script error never refuses a player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `session` | `not specified` | required | — |
| `code` | `not specified` | required | — |
| `message` | `not specified` | required | — |
| `details` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-preflight) (line 801).

## refusal

```lua
Open77.access.refusal(userId)
```

`SERVER`: Available in server resources only.

What the door would answer this identity now: `nil`, or `{ code = "banned" | "not_whitelisted", message }` with the handshake's sentence.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 774).

## refusals

```lua
Open77.access.refusals()
```

`SERVER`: Available in server resources only.

Recent refused connections, newest first (Warden's Refused, 50 at most): `{ { userId, name, code, message, at, atUnix } }`. Covers the handshake and the launcher's server check.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 775).

## request

```lua
Open77.access.request(userId, name?, message?, actor?)
```

`SERVER`: Available in server resources only.

File or renew a whitelist request for an identity that is not listed (message cut at 300 characters). `false, "already_allowed"` / `"banned"` / `"requests_full"` (500 pending).

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `not specified` | required | — |
| `name` | `not specified` | optional | — |
| `message` | `not specified` | optional | — |
| `actor` | `not specified` | optional | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 777).

## requests

```lua
Open77.access.requests("pending"?)
```

`SERVER`: Available in server resources only.

Whitelist requests, pending first: `{ userId, name, message, status = "pending" | "denied", requestedAt, requestedAtUnix, source, note, decidedBy?, decidedAt? }`.

Requires `players.access`. Who gets in, decided in Lua. The guide, with a whitelist and a ban list to copy, is
Connection control.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `"pending"` | `not specified` | optional | — |

[Connection control guide](../../01%20Documentation/connection-control.md)

Documented in [server-api.md](../../01%20Documentation/server-api.md#connection-control) (line 776).

## setWhitelist

```lua
Open77.access.setWhitelist(enabled)
```

`NETWORK`: Uses the network backend.

Turns the built-in whitelist on or off.

Requires `players.access`. While it is on, only identities returned by `status().whitelist` are admitted. The switch is persisted by the host, so it survives a resource reload and a restart.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Lock the server to the whitelist during the event
RegisterCommand("private", function(source, args)
    Open77.access.setWhitelist(args[1] == "on")
end, true)
```

[Connection control guide](../../01%20Documentation/connection-control.md)

Registered in `LuaResourceRuntime.cs` (line 8139) as `LuaResourceRuntime.Bootstrap`.

## status

```lua
Open77.access.status()
```

`NETWORK`: Uses the network backend.

The server's built-in whitelist and ban list.

Requires `players.access`. Returns `{ whitelistEnabled, whitelist = { { userId, label, addedAt, addedBy } }, bans = { { userId, name, reason, bannedAt, expiresAt?, bannedBy } } }` -- the same door list the console and Warden read and write, so a gamemode and an operator never disagree about who is allowed in.

Returns: `table, or false`, `reason`

```lua
-- /bans: list the current bans
RegisterCommand("bans", function(source)
    local status = Open77.access.status()
    for _, ban in ipairs(status and status.bans or {}) do
        Open77.chat.send(source, ("%s: %s"):format(ban.name, ban.reason))
    end
end, true)
```

[Connection control guide](../../01%20Documentation/connection-control.md)

Registered in `LuaResourceRuntime.cs` (line 8132) as `LuaResourceRuntime.Bootstrap`.

## unban

```lua
Open77.access.unban(userId)
```

`NETWORK`: Uses the network backend.

Lifts a ban on the built-in list.

Requires `players.access`. Lifting a ban does not re-add the identity to the whitelist, which is a separate list.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `userId` | `string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- /unban <userId>
RegisterCommand("unban", function(source, args)
    if args[1] then Open77.access.unban(args[1]) end
end, true)
```

[Connection control guide](../../01%20Documentation/connection-control.md)

Registered in `LuaResourceRuntime.cs` (line 8145) as `LuaResourceRuntime.Bootstrap`.
