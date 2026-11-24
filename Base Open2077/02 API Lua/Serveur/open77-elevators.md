# Open77.elevators (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-elevators.md) · [Page web](https://open2077.net/docs/api/server/open77-elevators)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

10 functions.

## adopt

```lua
Open77.elevators.adopt(def)
```

`NETWORK`: Uses the network backend.

Takes a native lift under authoritative control.

Requires `world.elevators`. The definition accepts `engineEntity`, `position`, `bucket`, `initialFloor`, `flags` (default powered plus interaction allowed) and a required `floorCount`. Adoption is what turns a scenery lift into a replicated object every client agrees about; before it, each client's lift is its own.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `def` | `table { engineEntity, position, bucket?, initialFloor?, flags?, floorCount }` | required | — |

Returns: `elevator id, or nil`, `reason`

```lua
-- Take the apartment lift under server control so everyone sees the same cab
local lift, reason = Open77.elevators.adopt({
    engineEntity = engineEntity,     -- from a client's Open77.elevators.nearby()
    position = { x = -1442.2, y = 127.4, z = 18.0 },
    floorCount = 12,
    initialFloor = 0,
})
if not lift then print("adopt failed: " .. reason) end
```

Registered in `LuaResourceRuntime.cs` (line 9821) as `LuaResourceRuntime.Bootstrap`.

## all

```lua
Open77.elevators.all([bucket])
```

`NETWORK`: Uses the network backend.

Every managed elevator, optionally filtered to one routing bucket.

Requires `world.elevators`. Returns the whole registry, adopted by whichever resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `array of elevator snapshots`

```lua
-- Send every lift back to the ground floor at server start
for _, lift in ipairs(Open77.elevators.all()) do
    Open77.elevators.goTo(lift.id, 0)
end
```

Registered in `LuaResourceRuntime.cs` (line 9840) as `LuaResourceRuntime.Bootstrap`.

## call

```lua
Open77.elevators.call(id, floor, [options])
```

`NETWORK`: Uses the network backend.

Alias of `Open77.elevators.goTo`.

Requires `world.elevators`. The same function under the name a call button would use; there is no separate queuing behaviour behind it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `elevator id` | required | — |
| `floor` | `integer` | required | — |
| `options` | `table { travelMs?, force? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A call button on floor 5
RegisterNetEvent("lift:button", function(elevatorId, floor)
    Open77.elevators.call(elevatorId, floor)
end)
```

Registered in `LuaResourceRuntime.cs` (line 9833) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.elevators.get(id)
```

`NETWORK`: Uses the network backend.

Reads one elevator's canonical snapshot.

Requires `world.elevators`. Answers `nil` for an id the registry does not know.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `elevator id` | required | — |

Returns: `elevator snapshot, or nil`

```lua
-- Only accept a new request when the lift is idle
local lift = Open77.elevators.get(elevatorId)
if lift and lift.phase == "idle" then Open77.elevators.goTo(elevatorId, 7) end
```

Registered in `LuaResourceRuntime.cs` (line 9839) as `LuaResourceRuntime.Bootstrap`.

## goTo

```lua
Open77.elevators.goTo(id, floor, [options])
```

`NETWORK`: Uses the network backend.

Starts authoritative travel to a floor.

Requires `world.elevators`. Options are `travelMs` (default 8000) and `force`, which starts travel even when the lift is not idle. The travel time is the server's contract with every viewer, so all clients animate the same journey.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `elevator id` | required | — |
| `floor` | `integer` | required | — |
| `options` | `table { travelMs?, force? }` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- A slow freight lift: 15 seconds to the top floor
Open77.elevators.goTo(elevatorId, 11, { travelMs = 15000 })
```

Registered in `LuaResourceRuntime.cs` (line 9829) as `LuaResourceRuntime.Bootstrap`.

## pause

```lua
Open77.elevators.pause(id)
```

`NETWORK`: Uses the network backend.

Pauses authoritative movement.

Requires `world.elevators`. The lift holds where it is and keeps its destination; `resume` continues the same journey.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `elevator id` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Power cut: every lift stops where it is
for _, lift in ipairs(Open77.elevators.all()) do Open77.elevators.pause(lift.id) end
```

Registered in `LuaResourceRuntime.cs` (line 9835) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.elevators.remove(id)
```

`NETWORK`: Uses the network backend.

Releases a managed elevator.

Requires `world.elevators`. The native lift stays in the world; only Open77's authority over it ends, so it reverts to ordinary client-local scenery.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `elevator id` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Hand the lift back to the engine when the resource no longer needs it
Open77.elevators.remove(elevatorId)
```

Registered in `LuaResourceRuntime.cs` (line 9838) as `LuaResourceRuntime.Bootstrap`.

## resume

```lua
Open77.elevators.resume(id)
```

`NETWORK`: Uses the network backend.

Continues a paused journey.

Requires `world.elevators`. Resuming a lift that is not paused is accepted and changes nothing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `elevator id` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Power is back
for _, lift in ipairs(Open77.elevators.all()) do Open77.elevators.resume(lift.id) end
```

Registered in `LuaResourceRuntime.cs` (line 9836) as `LuaResourceRuntime.Bootstrap`.

## setFlags

```lua
Open77.elevators.setFlags(id, flags)
```

`NETWORK`: Uses the network backend.

Sets the power, lock, interaction and door policy bits.

Requires `world.elevators`. The constants are in `Open77.elevators.flags`: `powered` (1), `locked` (2), `interactionAllowed` (4) and `doorsClosed` (8). This replaces the mask rather than merging it, so read the snapshot first when only one bit should change.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `elevator id` | required | — |
| `flags` | `integer bitmask` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Lock the lift for the heist: keep it powered but not usable
local f = Open77.elevators.flags
Open77.elevators.setFlags(elevatorId, f.powered | f.locked)
```

Registered in `LuaResourceRuntime.cs` (line 9837) as `LuaResourceRuntime.Bootstrap`.

## teleport

```lua
Open77.elevators.teleport(id, floor)
```

`NETWORK`: Uses the network backend.

Moves a lift to a floor with no travel.

Requires `world.elevators`. Administrative recovery: it skips the journey entirely, so it is the way out of a lift stuck mid-shaft, not a normal call.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `elevator id` | required | — |
| `floor` | `integer` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Admin recovery for a lift stuck between floors
RegisterCommand("liftfix", function(source, args)
    Open77.elevators.teleport(tonumber(args[1]), 0)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 9834) as `LuaResourceRuntime.Bootstrap`.
