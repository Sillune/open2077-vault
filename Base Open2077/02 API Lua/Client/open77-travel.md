# Open77.travel (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-travel.md) · [Page web](https://open2077.net/docs/api/client/open77-travel)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

11 functions.

## beginSettle

```lua
Open77.travel.beginSettle(x, y, z, [heading], [timeoutMs])
```

`GAME`: Requires a live game instance.

Raw half of teleportAndSettle: issues the teleport and returns a settle ticket.

Requires `player.travel`. Prefer `Open77.travel.teleportAndSettle`, which wraps this and `settleStatus` in a promise. `timeoutMs` is clamped to 1000..30000, default 12000.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `heading` | `number` | optional | — |
| `timeoutMs` | `integer` | optional | — |

Returns: `ticket, or nil`, `reason`

```lua
-- The raw form: teleport, then poll the ticket yourself
local ticket = Open77.travel.beginSettle(-1442.2, 127.4, 18.0, 90.0)
CreateThread(function()
    while true do
        local status = Open77.travel.settleStatus(ticket)
        if not status or status.state ~= "pending" then print("settle:", status and status.state); break end
        Wait(100)
    end
end)
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25432) as `LuaTravelBeginSettle`.

## cancelSettle

```lua
Open77.travel.cancelSettle(ticket)
```

`GAME`: Requires a live game instance.

Stops the live settle session.

Requires `player.travel`. The body stays wherever it is; only the watch stops, and its promise rejects with `settle_cancelled`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `ticket` | `integer` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The player pressed ESC during the fade: stop watching the move
Open77.travel.cancelSettle(ticket)
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25434) as `LuaTravelCancelSettle`.

## getNoclipSpeed

```lua
Open77.travel.getNoclipSpeed()
```

`GAME`: Requires a live game instance.

Returns the current native noclip speed for the local player.

Requires `player.travel`. Reads the speed the noclip controller uses right now, whether or not noclip is active; pair it with `isNoclip` to tell the two apart.

Returns: `number, the noclip speed in metres per second`, `false, reason (`permission_denied:player.travel`, `travel_unavailable_on_this_host`)`

```lua
-- Mirror the noclip speed on the HUD while an admin flies
if Open77.travel.isNoclip() then
    Open77.hud.notify(("Noclip %.0f m/s"):format(Open77.travel.getNoclipSpeed()))
end
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25429) as `LuaTravelNoclipSpeed`.

## isMapPick

```lua
Open77.travel.isMapPick()
```

`GAME`: Requires a live game instance.

Whether the world-map double-click travel gesture is armed.

Returns whether this client has the world-map double-click travel gesture armed. It requires `player.travel` and reports false when the permission or native bridge is unavailable. This is only local input state: it neither teleports the player nor bypasses the ACL check performed by the server command that consumes a picked point.

Returns: `boolean`

```lua
-- Show the "double-click the map to travel" hint only while the gesture is armed
if Open77.travel.isMapPick() then
    Open77.hud.notify("Double-click the map to travel there")
end
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25436) as `LuaTravelIsMapPick`.

## isNoclip

```lua
Open77.travel.isNoclip()
```

`GAME`: Requires a live game instance.

Whether local-player noclip is currently active.

Returns false without `player.travel` or when the backend is unavailable.

Returns: `boolean`

```lua
-- Hide the speedometer while flying around in noclip
SetTick(function()
    speedometer.visible = not Open77.travel.isNoclip()
end)
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25428) as `LuaTravelIsNoclip`.

## setMapPick

```lua
Open77.travel.setMapPick(enabled)
```

`GAME`: Requires a live game instance.

Arms or disarms the world-map double-click travel gesture.

Requires `player.travel`. This only lets the local map publish a picked world point; it moves nobody and does not bypass the ACL-checked server command that consumes the point.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- An admin resource arms map-pick travel when the server says the player may
RegisterNetEvent("admin:mapTravel", function(allowed)
    Open77.travel.setMapPick(allowed)
end)
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25435) as `LuaTravelSetMapPick`.

## setNoclip

```lua
Open77.travel.setNoclip(enabled)
```

`GAME`: Requires a live game instance.

Enables or disables native noclip for the local player.

Requires `player.travel`. Multiplayer deployments should expose this only behind an ACL-checked server command.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Toggle noclip, only after the server approved the admin command
RegisterNetEvent("admin:noclip", function(enabled)
    Open77.travel.setNoclip(enabled)
    Open77.hud.notify(enabled and "Noclip on" or "Noclip off")
end)
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25427) as `LuaTravelSetNoclip`.

## setNoclipSpeed

```lua
Open77.travel.setNoclipSpeed(speed)
```

`GAME`: Requires a live game instance.

Sets the local noclip movement speed.

Requires `player.travel`; the value must be finite and is validated by the native backend.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `speed` | `number` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Faster noclip while SHIFT is held
SetTick(function()
    if Open77.travel.isNoclip() then
        Open77.travel.setNoclipSpeed(Open77.input.isDown("SHIFT") and 40 or 10)
    end
end)
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25430) as `LuaTravelSetNoclipSpeed`.

## settleStatus

```lua
Open77.travel.settleStatus(ticket)
```

`GAME`: Requires a live game instance.

Reads the state of a settle ticket.

Requires `player.travel`. Returns `{ state, reason, x, y, z, attempts, pins, elapsedMs }` where `state` is `pending`, `settled`, `near`, `failed` or `idle`; `nil, unknown_settle_ticket` for a ticket that is neither the live session nor the last one to finish.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `ticket` | `integer` | required | — |

Returns: `table, or nil`, `reason`

```lua
-- Is the body really standing at the destination yet?
local status = Open77.travel.settleStatus(ticket)
if status and status.state == "settled" then Open77.hud.notify("Arrived") end
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25433) as `LuaTravelSettleStatus`.

## teleport

```lua
Open77.travel.teleport(x, y, z, [heading])
```

`GAME`: Requires a live game instance.

Teleports the local player directly to a world transform.

Requires `player.travel`. One queued engine request: it returns before the body has moved and knows nothing about streaming, so a destination whose floor is not resident yet drops the player through it and the engine's fall-under-world failsafe silently returns them to the save's spawn. Use `Open77.travel.teleportAndSettle` for a move that has to be believed, or `Open77.players.teleport` when the server decides. The server respawn transaction is NOT a general move: it refuses unless the player is already dead.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `z` | `number` | required | — |
| `heading` | `number` | optional | `0` |

Returns: `true on success, otherwise false`, `reason`

```lua
-- A quick local move to the garage (no confirmation that the floor was loaded)
RegisterCommand("garage", function()
    Open77.travel.teleport(-1442.2, 127.4, 18.0, 90.0)
end, false)
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 25431) as `LuaTravelTeleport`.

## teleportAndSettle

```lua
Open77.travel.teleportAndSettle(position, [heading], [options])
```

`GAME`: Requires a live game instance.

Teleports the local player and resolves when the body is really standing there.

Requires `player.travel`. Returns a Promise, or `nil, reason` when the teleport itself was refused. Resolves with `{ state, x, y, z, attempts, pins, elapsedMs }`, where `state` is `settled` (grounded on the point for three consecutive frames) or `near` (on the point but never reported grounded inside the 1.5 s window -- a mark on a prop, or a very slow stream-in). Rejects with `settle_timeout`, `settle_superseded`, `settle_cancelled`, or the reason the teleport refused. There is no streaming query anywhere in the engine surface, so this is the empirical answer: while the body sinks through a floor that has not streamed in, the teleport is re-issued every 250 ms, which resets the fall before the engine's own fall-under-world failsafe can drag the player to the save's spawn. One local player means one session, so a second call supersedes the first.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y, z }` | required | — |
| `heading` | `number` | optional | — |
| `options` | `table { timeoutMs? }` | optional | — |

Returns: `Promise, or nil`, `reason`

```lua
-- Move to the garage and only then show the menu
CreateThread(function()
    local pending, reason = Open77.travel.teleportAndSettle({ x = -1442.2, y = 127.4, z = 18.0 }, 90.0)
    if not pending then return Open77.hud.notify("Teleport refused: " .. reason) end
    local landed, err = pending:await()
    if landed then openGarageMenu() else Open77.hud.notify("Could not settle: " .. err) end
end)
```

[Travel & settling guide](../../01%20Documentation/travel.md)

Registered in `ResourceHost.cpp` (line 23214) as `LuaPrelude`.
