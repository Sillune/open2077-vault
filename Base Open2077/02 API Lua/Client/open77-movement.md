# Open77.movement (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-movement.md) · [Page web](https://open2077.net/docs/api/client/open77-movement)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

5 functions.

## getWalkMode

```lua
Open77.movement.getWalkMode()
```

`GAME`: Requires a live game instance.

Reads the walk-mode lease this resource holds on the local player.

Requires `players.controls`. `enabled` is the mode the engine currently applies for this resource, `requested` what the resource last asked for through `setWalkMode`; the two differ while another resource's lease wins or the puppet is not yet attached. `speed` and `requestedSpeed` follow the same split, in metres per second.

Returns: `table with `enabled`, `requested`, `speed`, `requestedSpeed``, `nil, reason (`permission_denied:players.controls`, `unexpected_arguments`, `game_unavailable_on_this_host`)`

```lua
-- Show whether the slow walk this resource asked for is actually applied
local mode = Open77.movement.getWalkMode()
if mode and mode.requested and not mode.enabled then
    Open77.hud.notify("Walk mode pending: another resource holds the lease")
end
```

Registered in `ResourceHost.cpp` (line 24906) as `LuaGetWalkMode`.

## lock

```lua
Open77.movement.lock(kind, durationMs)
```

`GAME`: Requires a live game instance.

Applies a bounded local movement lock -- freeze, cripple or heavy cripple -- to the local player.

Client-only; requires `player.movement.lock`, a server-driven projection primitive a player resource should not hold by default. `kind` is `freeze` (no movement), `cripple` (slow) or `cripple_heavy`; `durationMs` is 1..3600000 and the adapter clamps it to its own cap. One slot, owned by the calling resource; a later call replaces this resource's own lock and never another's. The adapter releases on the deadline, on death, vehicle entry, workspot, session change and resource stop -- `lockState` exposes the watchdog that removes a lock whose owner never released it. Returns the request id; the server's deadline stays the authority. For a server-authoritative freeze use `Open77.players.setFrozen` on the server instead.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `kind` | `string (freeze | cripple | cripple_heavy)` | required | — |
| `durationMs` | `integer` | required | — |

Returns: `request id, or nil`, `reason: permission_denied:player.movement.lock, resource_not_running, movement_backend_unavailable, invalid_kind, invalid_duration`

```lua
-- A "cripple" status from the server: the player walks slowly for five seconds
RegisterNetEvent("hack:crippled", function()
    Open77.movement.lock("cripple", 5000)
end)
```

Registered in `ResourceHost.cpp` (line 24907) as `LuaMovementLock`.

## lockState

```lua
Open77.movement.lockState()
```

`GAME`: Requires a live game instance.

Reads the current client movement-lock slot and its lifecycle counters.

Client-only; requires player.movement.lock. Returns phase, kind, owner, reason, request, grantedMs, remainingMs, elapsedMs, locks, refusals, releases, watchdogRemovals, releaseOverdue and locked. phase is the adapter's own state, not proof of a rendered input restriction, and the server's deadline remains the authority. releaseOverdue and watchdogRemovals expose the bounded watchdog that removes a lock whose owner never released it.

Returns: `state table, or nil, reason`

```lua
local state, reason = Open77.movement.lockState()
if state then print(state.phase, state.kind, state.remainingMs) else print(reason) end
```

Registered in `ResourceHost.cpp` (line 24909) as `LuaMovementLockState`.

## setWalkMode

```lua
Open77.movement.setWalkMode(enabled, [speed])
```

`GAME`: Requires a live game instance.

Asks the local player to walk at a bounded speed, as a lease this resource owns.

Requires `players.controls`. Every resource holds its own request and the slowest live request wins; `false` releases only this resource's request and never touches another's. `speed` is metres per second between 0.5 and 2.5, default 1.5. Read the applied state back with `getWalkMode`. Walk mode slows walking and jogging but lets the player sprint out of it: to hold a player to the pace, also block `Sprint` with `Open77.input.setActionBlocked` (that call needs the `input.actions` permission), which also stops this lease from lifting on sprint intent.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |
| `speed` | `number` | optional | — |

Returns: `true on success, otherwise false`, `reason (`permission_denied:players.controls`, `resource_not_running`, `expected_boolean`, `invalid_walk_speed`, `game_unavailable_on_this_host`)`

```lua
-- Slow walk while carrying a body; release it when the body is put down
RegisterNetEvent("carry:start", function()
    Open77.movement.setWalkMode(true, 1.0)
end)
RegisterNetEvent("carry:stop", function()
    Open77.movement.setWalkMode(false)
end)
```

Registered in `ResourceHost.cpp` (line 24905) as `LuaSetWalkMode`.

## unlock

```lua
Open77.movement.unlock()
```

`GAME`: Requires a live game instance.

Releases this resource's owned client movement lock.

Client-only; requires player.movement.lock. Releases only a lock the calling resource owns and is harmless when it owns none. It cannot lift another resource's lock. true acknowledges the release request; the native adapter also releases on its own deadline and on death, vehicle entry, workspot, session change and resource stop. Releasing does not undo damage, cancel an authoritative hack or refund a status; use the server hacking APIs for gameplay state.

Returns: `true, or nil, reason`

```lua
-- The cripple status ended early: release this resource's lock
RegisterNetEvent("hack:cured", function()
    Open77.movement.unlock()
end)
```

Registered in `ResourceHost.cpp` (line 24908) as `LuaMovementUnlock`.
