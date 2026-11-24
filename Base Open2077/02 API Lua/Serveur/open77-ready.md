# Open77.ready (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-ready.md) · [Page web](https://open2077.net/docs/api/server/open77-ready)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## hold

```lua
Open77.ready.hold(playerId, [reason])
```

`NETWORK`: Uses the network backend.

Takes or refreshes this resource's hold on a player.

No permission. Returns the player's `session`, which later identifies the hold being released. Refreshing an existing hold restarts its deadline.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `reason` | `string` | optional | — |

Returns: `session, or nil`, `reason`

```lua
-- Hold a joining player until their character is loaded from the database
AddEventHandler("onPlayerConnected", function(playerId)
    local session = Open77.ready.hold(playerId, "loading character")
    loadCharacter(playerId, function() Open77.ready.release(playerId, session) end)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8125) as `LuaResourceRuntime.Bootstrap`.

## isReady

```lua
Open77.ready.isReady(playerId)
```

`NETWORK`: Uses the network backend.

Whether anything is still holding a player.

No permission. A player the host does not know is reported ready, so an unknown id never blocks a mode forever. The rule the gate exists for: do not teleport, spawn, kill or force a respawn on a player until this is true. Reading state, rosters and HUD payloads is unaffected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `boolean`

```lua
-- Never teleport a player the gate still holds
if not Open77.ready.isReady(playerId) then return end
Open77.players.teleport(playerId, spawnPoint)
```

Registered in `LuaResourceRuntime.cs` (line 8127) as `LuaResourceRuntime.Bootstrap`.

## participate

```lua
Open77.ready.participate([options])
```

`NETWORK`: Uses the network backend.

Declares that this resource holds joining players until it is finished with them.

No permission. Call it **once, at load**: every player who connects from then on arrives with one hold in this resource's name, which is what stops another resource teleporting or spawning them mid-setup. Options are `timeoutMs` and `reason`. Every hold carries a deadline, so a resource that never releases costs one degraded join and one `WRN` naming it, never a stuck server.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table { timeoutMs?, reason? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Declare at load that this resource holds every joining player
Open77.ready.participate({ timeoutMs = 15000, reason = "character load" })
```

Registered in `LuaResourceRuntime.cs` (line 8124) as `LuaResourceRuntime.Bootstrap`.

## release

```lua
Open77.ready.release(playerId, [session], [note])
```

`NETWORK`: Uses the network backend.

Clears this resource's hold on a player.

No permission. A `session` that no longer matches is dropped rather than releasing a newer hold -- that is what keeps a late release from opening the gate for the player who reconnected in the meantime. `onPlayerReady(playerId, detail)` is emitted into **every** running resource when the last hold clears.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `session` | `session token` | optional | — |
| `note` | `string` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Character loaded: release the hold taken at connect
Open77.ready.release(playerId, session, "character loaded")
```

Registered in `LuaResourceRuntime.cs` (line 8126) as `LuaResourceRuntime.Bootstrap`.

## status

```lua
Open77.ready.status(playerId)
```

`NETWORK`: Uses the network backend.

Who is holding a player, and for how long.

No permission. Returns `{ known, ready, session, ageMs, holds = { { resource, reason, ageMs, remainingMs } } }`. The `ready` console command prints the same thing, which is the quickest way to find the resource that is not releasing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `table`

```lua
-- /whoholds <id>: which resource is not releasing a player?
RegisterCommand("whoholds", function(source, args)
    local status = Open77.ready.status(tonumber(args[1]))
    for _, hold in ipairs(status.holds or {}) do
        Open77.chat.send(source, ("%s: %s (%d ms left)"):format(hold.resource, hold.reason, hold.remainingMs))
    end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8128) as `LuaResourceRuntime.Bootstrap`.
