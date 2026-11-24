# Open77.prevention (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-prevention.md) · [Page web](https://open2077.net/docs/api/client/open77-prevention)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

4 functions.

## setEnabled

```lua
Open77.prevention.setEnabled(enabled)
```

`GAME`: Requires a live game instance.

Turns NCPD dispatch on or off for this client.

`TogglePreventionSystem`, the nearest equivalent of `SetDispatchServiceActive`. This is the session's own switch and is **not** the three block flags `VehicleSpawnPolicy` owns and drives from world population suppression; this call deliberately never touches those, so the two cannot fight. Reaches the engine through the REDscript command queue, never a native call: all 276 methods on `PreventionSystem` are scripted, and invoking one from C++ without a script frame terminates the process. A `true` return therefore means *queued*, not applied; `Open77.prevention.state()` reports whether the script loop later acknowledged it. Requires `world.prevention`. A server that sets policy through `Open77.players.setWanted` reaches this same surface through the `open77_prevention` resource; a client resource calling it directly is overriding that.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true when queued, otherwise false`, `reason: permission_denied:world.prevention, invalid_prevention_argument, script_bridge_unavailable, script_queue_full`

```lua
-- The server turned dispatch off for this instance
RegisterNetEvent("world:dispatch", function(enabled)
    Open77.prevention.setEnabled(enabled)
end)
```

[Wanted level & NCPD dispatch](../../01%20Documentation/server-api.md#wanted-level-and-ncpd-dispatch)

Registered in `ResourceHost.cpp` (line 25242) as `LuaPreventionSetEnabled`.

## setMaxWanted

```lua
Open77.prevention.setMaxWanted(level)
```

`GAME`: Requires a live game instance.

Sets the ceiling the wanted level resets to, 0 to 5.

`SetMinMaxResetHeatLevels(0, level, false)`. The floor stays at 0 deliberately: a nonzero minimum would strand a session at a heat it could never clear. Reaches the engine through the REDscript command queue, never a native call: all 276 methods on `PreventionSystem` are scripted, and invoking one from C++ without a script frame terminates the process. A `true` return therefore means *queued*, not applied; `Open77.prevention.state()` reports whether the script loop later acknowledged it. Requires `world.prevention`. A server that sets policy through `Open77.players.setWanted` reaches this same surface through the `open77_prevention` resource; a client resource calling it directly is overriding that.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `level` | `integer (0..5)` | required | — |

Returns: `true when queued, otherwise false`, `reason: permission_denied:world.prevention, invalid_wanted_level, script_bridge_unavailable, script_queue_full`

```lua
-- The VIP zone caps heat at two stars on this client
Open77.prevention.setMaxWanted(2)
```

[Wanted level & NCPD dispatch](../../01%20Documentation/server-api.md#wanted-level-and-ncpd-dispatch)

Registered in `ResourceHost.cpp` (line 25241) as `LuaPreventionSetMaxWanted`.

## setWanted

```lua
Open77.prevention.setWanted(level)
```

`GAME`: Requires a live game instance.

Sets this client's NCPD wanted level, 0 to 5.

The wanted level is **world state on this client**, not player state: nothing in the engine's wanted surface takes a target, the heat stage is a scalar on a singleton `PreventionSystem`, and every consumer reads one global `UI_WantedBar`. The NCPD units it summons are this client's own - not Open77 entities, not replicated, invisible to every other player. `0` clears it. Raising it above 0 while world population suppression is active is refused with `population_suppressed` rather than silently doing nothing: the spawn policy is holding `SetSystemLock`/`SetBlockOnFootSpawn`/`SetBlockVehicleSpawn` on in that state, so the stage would change and no unit would ever arrive. Clearing to 0 is always allowed. Reaches the engine through the REDscript command queue, never a native call: all 276 methods on `PreventionSystem` are scripted, and invoking one from C++ without a script frame terminates the process. A `true` return therefore means *queued*, not applied; `Open77.prevention.state()` reports whether the script loop later acknowledged it. Requires `world.prevention`. A server that sets policy through `Open77.players.setWanted` reaches this same surface through the `open77_prevention` resource; a client resource calling it directly is overriding that.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `level` | `integer (0..5)` | required | — |

Returns: `true when queued, otherwise false`, `reason: permission_denied:world.prevention, invalid_wanted_level, population_suppressed, script_bridge_unavailable, script_queue_full`

```lua
-- Clear the local heat after the bribe
local ok, reason = Open77.prevention.setWanted(0)
if not ok then print("wanted not cleared: " .. reason) end
```

[Wanted level & NCPD dispatch](../../01%20Documentation/server-api.md#wanted-level-and-ncpd-dispatch)

Registered in `ResourceHost.cpp` (line 25240) as `LuaPreventionSetWanted`.

## state

```lua
Open77.prevention.state()
```

`GAME`: Requires a live game instance.

What was last requested of the prevention system, and whether the script loop acknowledged it.

**Never an engine reading.** There is no value channel back from REDscript, only a per-command acknowledgement, so this reports a request record and the field names say so: `requestedLevel`, `requestedMaxLevel`, `requestedEnabled`, plus `levelAcknowledged`, `maxLevelAcknowledged` and `enabledAcknowledged`. An acknowledgement means the script loop found `PreventionSystem` and ran the call; it does not mean the engine honoured it. `scriptAttached` is false until the REDscript loop has polled at least once - anything queued before that is waiting, not lost. `populationSuppressed` says whether the spawn policy is currently holding police off, which is why a `setWanted` raise would be refused. Requires `world.prevention`.

Returns: `table { scriptAttached, populationSuppressed, requestedLevel, requestedMaxLevel, requestedEnabled, levelAcknowledged, maxLevelAcknowledged, enabledAcknowledged }, or nil`, `reason: permission_denied:world.prevention, prevention_unavailable_on_this_host`

```lua
-- Did the script loop acknowledge the last request?
local state = Open77.prevention.state()
if state and not state.levelAcknowledged then print("wanted level still pending") end
```

[Wanted level & NCPD dispatch](../../01%20Documentation/server-api.md#wanted-level-and-ncpd-dispatch)

Registered in `ResourceHost.cpp` (line 25243) as `LuaPreventionState`.
