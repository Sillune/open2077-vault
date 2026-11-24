# Open77.elevators (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-elevators.md) · [Page web](https://open2077.net/docs/api/client/open77-elevators)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

4 functions.

## all

```lua
Open77.elevators.all()
```

`GAME`: Requires a live game instance.

Lists every replicated elevator state visible to the client.

Requires `elevators.read`; returns an empty array when the capability or backend is unavailable.

Returns: `array of elevator snapshots`

```lua
-- Show every managed elevator's floor on a debug overlay
for _, lift in ipairs(Open77.elevators.all()) do
    print(("elevator %d at floor %d (%s)"):format(lift.id, lift.activeFloor, lift.phase))
end
```

Registered in `ResourceHost.cpp` (line 25132) as `LuaElevatorsAll`.

## get

```lua
Open77.elevators.get(id)
```

`GAME`: Requires a live game instance.

Reads one replicated elevator by Open77 ID.

Requires `elevators.read`. Elevator IDs are server-assigned and should be kept unchanged.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `table { id, revision, bucket, engineEntity, activeFloor, originFloor, targetFloor, floorCount, travelMs, remainingMs, flags, phase, position, streamed, applied }, or nil`

```lua
-- Is the lift the player called still moving?
local lift = Open77.elevators.get(elevatorId)
if lift and lift.remainingMs > 0 then
    Open77.hud.notify(("Arriving in %d s"):format(math.ceil(lift.remainingMs / 1000)), { replace = true })
end
```

Registered in `ResourceHost.cpp` (line 25131) as `LuaElevatorGet`.

## nearby

```lua
Open77.elevators.nearby([radius])
```

`GAME`: Requires a live game instance.

Discovers streamed native elevators around the local player.

Requires `elevators.read`. Radius must be between 1 and 300 metres. Results distinguish unmanaged engine lifts from elevators already adopted by Open77.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `radius` | `number` | optional | `100` |

Returns: `array of { engineEntity, controllerEntity, position, distance, managed, floorCount, activeFloor, id }`

```lua
-- Find an adopted lift within 10 m to offer the call prompt
for _, lift in ipairs(Open77.elevators.nearby(10)) do
    if lift.managed then Open77.hud.notify("Press E to call the elevator", { replace = true }) end
end
```

Registered in `ResourceHost.cpp` (line 25133) as `LuaElevatorsNearby`.

## request

```lua
Open77.elevators.request(id, floor, action)
```

`GAME`: Requires a live game instance.

Submits a floor request to the authoritative server.

Requires `elevators.request`. `action` is normally `call` or `goto`; the server validates the ID, floor, player bucket, range, state and policy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `floor` | `integer` | required | — |
| `action` | `string` | required | — |

Returns: `true when submitted, otherwise false`, `reason`

```lua
-- Press E to call the nearest managed lift to the ground floor
RegisterKeyMapping("elevator_call", "Call elevator", "E", function()
    for _, lift in ipairs(Open77.elevators.nearby(5)) do
        if lift.managed then Open77.elevators.request(lift.id, 0, "call"); break end
    end
end)
```

Registered in `ResourceHost.cpp` (line 25134) as `LuaElevatorRequest`.
