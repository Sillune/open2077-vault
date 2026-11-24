# Open77.vehicles (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-vehicles.md) · [Page web](https://open2077.net/docs/api/server/open77-vehicles)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

175 functions.

## adopt

```lua
Open77.vehicles.adopt(playerId, description)
```

`NETWORK`: Uses the network backend.

Turns a client's description of a vanilla car into a canonical vehicle.

Requires `world.vehicles` **and** `world.vehicles.adopt`, which is its own string because this is the one vehicle call that creates world state from something a client wrote. `playerId` must be the player the server believes sent the description — inside a net event handler that is `source`, and it must never be read out of the payload, because the placement checks are the entire defence. The description is what the client's `Open77.vehicles.adoptable()` hands back: `engineEntity`, `record`, `position`, and optionally `orientation`, `health`, `flags`, `primaryColor`, `secondaryColor`, `doors`, `windows`, `tires`, `brokenGlass`, `brokenLights` and `detachedParts`.

**Adoption takes the car's identity, not the engine's entity.** A traffic car is spawned, owned and despawned by REDengine population on one client, and the plugin has no notification it can rely on for a despawn it did not start — a canonical vehicle bound to such an entity would quietly become a ghost. So nothing is bound: the server creates an ordinary canonical vehicle from the description, with the same lifetime, the same reaper and the same projection path as one a garage spawned, and the client then hands its vanilla chassis back with `Open77.vehicles.releaseAdopted`.

Refusals are `adopt_policy_closed` (the player's bucket is not open), `too_far`, `player_unavailable` (no position for them), `duplicate_adoption` (a car of the same record is already adopted within three metres of that spot, because traffic is spawned per client and two players on one corner each describe their own), `vehicle_limit`, and the `invalid_*` family. `already_adopted` is not a failure: it answers with the id of the vehicle the first proposal created, because a retry after a dropped acknowledgement is the ordinary way a client proposes the same chassis twice.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `description` | `table` | required | — |

Returns: `vehicle id, or nil`, `reason`

```lua
RegisterNetEvent("garage:adopt", function(description)
  local id, reason = Open77.vehicles.adopt(source, description)
  if not id then return print(reason) end
  -- Tell the proposing client to hand its vanilla chassis back.
  TriggerClientEvent("garage:adopted", source, description.engineEntity, id)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## all

```lua
Open77.vehicles.all([bucket])
```

`NETWORK`: Uses the network backend.

Lists authoritative server vehicles.

Returns snapshots ordered by Open77 ID. An optional routing bucket filters the result; omitting it returns every vehicle visible to the resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `array of vehicle tables`

```lua
-- /cars: how many server vehicles exist, and how many in the caller's bucket
RegisterCommand("cars", function(source)
    local everywhere = Open77.vehicles.all()
    local here = Open77.vehicles.all(Open77.players.get(source).bucket)
    Open77.chat.send(source, ("%d vehicles on the server, %d in your instance"):format(#everywhere, #here))
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 9222) as `LuaResourceRuntime.Bootstrap`.

## applyProperties

```lua
Open77.vehicles.applyProperties(id, props, [options])
```

`NETWORK`: Uses the network backend.

Alias for `setProperties`.

Identical contract. Useful to a codebase that reserves `set` for single-field writes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `props` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- Restore a garage car exactly as it was stored
local stored = json.decode(row.props)
local id = Open77.vehicles.create({ record = stored.record, position = spot })
local ok, reason = Open77.vehicles.applyProperties(id, stored)
if not ok then print("garage restore failed: " .. reason) end
```

[Vehicle properties, and what Cyberpunk does not have](../../01%20Documentation/vehicles.md#vehicle-properties-and-what-cyberpunk-does-not-have)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## areLightsOn

```lua
Open77.vehicles.areLightsOn(id)
```

`NETWORK`: Uses the network backend.

Whether a vehicle's headlights are lit at all.

Requires `world.vehicles`. True for both `"on"` and `"high"`; `getLights` tells them apart.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `boolean, or nil`, `reason: vehicle_not_found`

```lua
-- A traffic cop script: fine anyone driving at night with the lights off
for _, entry in ipairs(Open77.vehicles.all()) do
    local lit = Open77.vehicles.areLightsOn(entry.id)
    local driver = Open77.vehicles.getDriver(entry.id)
    if lit == false and driver then
        Open77.chat.send(driver, "Turn your headlights on!")
    end
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## breakAllGlass

```lua
Open77.vehicles.breakAllGlass(id, [count])
```

`NETWORK`: Uses the network backend.

Breaks a range of vehicle glass elements.

Requires `world.vehicles`. Breaks the first `count` destructible-glass bits. The optional count defaults to all 32 and must be in 0..32.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `count` | `integer` | optional | `32` |

Returns: `boolean success`

```lua
-- /wreck-glass <vehicleId>: shatter every window of a car
RegisterCommand("wreck-glass", function(source, args)
    local vehicleId = tonumber(args[1])
    if not vehicleId then return end
    Open77.vehicles.breakAllGlass(vehicleId)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 9143) as `LuaResourceRuntime.Bootstrap`.

## breakGlass

```lua
Open77.vehicles.breakGlass(id, glass)
```

`NETWORK`: Uses the network backend.

Breaks one vehicle glass element.

Requires `world.vehicles`. Convenience wrapper over `setGlassBroken(id, glass, true)`; the canonical bit is replicated and retained for late joiners.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `glass` | `integer|string` | required | — |

Returns: `boolean success`

```lua
-- The thief smashes the driver's window to get in
Open77.vehicles.breakGlass(vehicleId, 1)
```

Registered in `LuaResourceRuntime.cs` (line 9141) as `LuaResourceRuntime.Bootstrap`.

## breakLight

```lua
Open77.vehicles.breakLight(id, index)
```

`NETWORK`: Uses the network backend.

Breaks one vehicle light.

Requires `world.vehicles`. Convenience wrapper over `setLightBroken(id, index, true)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `index` | `integer` | required | — |

Returns: `boolean success`

```lua
-- A hit on the front of the car breaks the first two lights
Open77.vehicles.breakLight(vehicleId, 0)
Open77.vehicles.breakLight(vehicleId, 1)
```

Registered in `LuaResourceRuntime.cs` (line 9150) as `LuaResourceRuntime.Bootstrap`.

## breakTire

```lua
Open77.vehicles.breakTire(id, index)
```

`NETWORK`: Uses the network backend.

Breaks one vehicle tire.

Requires `world.vehicles`. Convenience wrapper over `setTireBroken(id, index, true)`. Tire indexes are 0..3.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `index` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Spike strip: burst the two front tires
Open77.vehicles.breakTire(vehicleId, 0)
Open77.vehicles.breakTire(vehicleId, 1)
```

Registered in `LuaResourceRuntime.cs` (line 9155) as `LuaResourceRuntime.Bootstrap`.

## cancelPlayerSeatSwitch

```lua
Open77.vehicles.cancelPlayerSeatSwitch(playerId, transitionId)
```

`NETWORK`: Uses the network backend.

Cancels exactly one player seat-transition token.

Requires world.vehicles. Releases the destination and retains the source. A stale token is refused with stale_transition and cannot affect a newer request.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `transitionId` | `string` | required | — |

Returns: `true or false`, `reason`

```lua
local s = Open77.vehicles.getPlayerSeatSwitch(playerId)
if s then Open77.vehicles.cancelPlayerSeatSwitch(playerId, s.id) end
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9014) as `LuaResourceRuntime.Bootstrap`.

## clearLightsHue

```lua
Open77.vehicles.clearLightsHue(id)
```

`NETWORK`: Uses the network backend.

Hands the lights back the record's own colour.

Requires `world.vehicles`. Exactly `setLightsHue(id, nil)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The tuning shop's "stock lights" option
RegisterCommand("lights-stock", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return Open77.chat.send(source, "Sit in the car first.") end
    Open77.vehicles.clearLightsHue(seat.vehicleId)
end, false)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## clearLockedForPlayer

```lua
Open77.vehicles.clearLockedForPlayer(id, playerId)
```

`NETWORK`: Uses the network backend.

Drops one player's lock exception.

Requires `world.vehicles`. Identical to `Open77.vehicles.setLockedForPlayer(id, playerId, nil)`: the player goes back to whatever the canonical bit says, and is re-sent that answer immediately.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `playerId` | `integer` | required | — |

Returns: `true, or false`, `reason: invalid_player_id, vehicle_not_found`

```lua
-- Taking the keys back: the player sees the car's real lock state again
RegisterCommand("keys-revoke", function(source, args)
    local vehicleId, target = tonumber(args[1]), tonumber(args[2])
    if not vehicleId or not target then return end
    Open77.vehicles.clearLockedForPlayer(vehicleId, target)
    Open77.chat.send(target, "Your keys were taken back.")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## clearNeon

```lua
Open77.vehicles.clearNeon(id)
```

`NETWORK`: Uses the network backend.

Removes a car's under-chassis lights.

Requires `world.vehicles`; a call that CHANGES the look also requires `world.vehicles.appearance` (`permission_denied:world.vehicles.appearance`), while restating the current value needs only the first -- the rule `setAppearance` follows, which keeps `setProperties(id, getProperties(id))` free. Takes the neon off entirely (`getVisual` then has no `neon`); `setNeon(id, { enabled = false })` instead keeps the colour for later.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `true, or false`, `reason: permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
Open77.vehicles.clearNeon(vehicleId)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## clearPerformance

```lua
Open77.vehicles.clearPerformance(id)
```

`NETWORK`: Uses the network backend.

Hands a vehicle back its record's rated performance.

Requires `world.vehicles`. Identical to `Open77.vehicles.setPerformance(id, nil)`. The withdrawal is replicated like the ceiling was, so every projection stops clamping rather than keeping the last value until it streams out.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `true, or false`, `reason: vehicle_not_found`

```lua
-- The race ended: hand every car back its real top speed
for _, vehicleId in ipairs(raceCars) do
    Open77.vehicles.clearPerformance(vehicleId)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## closeDoor

```lua
Open77.vehicles.closeDoor(id, door)
```

`NETWORK`: Uses the network backend.

Closes a door, trunk, or hood.

Requires `world.vehicles`. Convenience wrapper over `setDoorOpen(id, door, false)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `door` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Close the trunk once the delivery has been loaded
Open77.vehicles.closeDoor(vehicleId, Open77.vehicles.doors.trunk)
```

Registered in `LuaResourceRuntime.cs` (line 9190) as `LuaResourceRuntime.Bootstrap`.

## closest

```lua
Open77.vehicles.closest(anchor, [options])
```

`NETWORK`: Uses the network backend.

The nearest registered vehicle to a point or a player.

Requires `world.vehicles`. `Open77.vehicles.nearby` with a limit of one, and the same options. Nobody in range is a plain `nil` with NO reason, which is how a caller tells 'the search worked and the street is empty' from 'the search failed'. **Only server-spawned vehicles are in the registry.** A player standing next to a vanilla traffic car is standing next to something REDengine population spawned on their own client: it has no canonical id and this call cannot see it. Adopting vanilla traffic is a separate capability that does not exist yet.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `anchor` | `vector3|table { x, y, z }|playerId` | required | — |
| `options` | `table { radius?, bucket?, occupied? }` | optional | — |

Returns: `entry, or nil`, `reason (only on failure)`

```lua
-- /lock: lock the nearest server vehicle to the caller
RegisterCommand("lock", function(source)
    local nearest = Open77.vehicles.closest(source, { radius = 8 })
    if not nearest then return Open77.chat.send(source, "No vehicle nearby.") end
    Open77.vehicles.lock(nearest.id)
    Open77.chat.send(source, ("Locked %s (%.1f m away)"):format(nearest.record, nearest.distance))
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## closeWindow

```lua
Open77.vehicles.closeWindow(id, window)
```

`NETWORK`: Uses the network backend.

Closes one side window.

Requires `world.vehicles`. Convenience wrapper over `setWindowOpen(id, window, false)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `window` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Roll up the driver's window when it starts raining
AddEventHandler("onEnvironmentChanged", function(env)
    if env.weather ~= "rain" then return end
    for _, entry in ipairs(Open77.vehicles.all()) do
        Open77.vehicles.closeWindow(entry.id, Open77.vehicles.windows.frontLeft)
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 9200) as `LuaResourceRuntime.Bootstrap`.

## create

```lua
Open77.vehicles.create(definition)
```

`NETWORK`: Uses the network backend.

Creates a server-authoritative network vehicle.

Requires `world.vehicles`. The definition selects the record, transform, routing bucket, appearance, optional authoritative paint, flags, health, and initial damage. The server allocates the Open77 ID and replicates the canonical vehicle to interested clients. The creator name is retained as provenance and cleanup scope, never as an exclusive mutation lock. The definition also takes the three lifetime keys `ttlMs`, `persistent` and `despawnWhenUnobserved`, so a garage does not need a second call; a definition without them behaves exactly as it did before, and a lifetime that cannot be applied undoes the creation rather than half-honouring it -- the same rule the initial-damage path already follows.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `vehicle ID, or `nil, reason``

```lua
-- /car: spawn a Quadra a few metres in front of the caller and put them in the driver seat
RegisterCommand("car", function(source)
    local me = Open77.players.get(source)
    if not me or not me.position then return end
    local id, reason = Open77.vehicles.create({
        record = "Vehicle.v_sport1_quadra_turbo_r_player",
        position = { x = me.position.x + 4.0, y = me.position.y, z = me.position.z },
        yaw = me.heading,
        ttlMs = 30 * 60 * 1000, -- gone after 30 minutes if nobody removes it
    })
    if not id then return Open77.chat.send(source, "Spawn failed: " .. reason) end
    Open77.vehicles.warpPlayerIntoVehicle(source, id, Open77.vehicles.seats.driver)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 9085) as `LuaResourceRuntime.Bootstrap`.

## damageBodyCell

```lua
Open77.vehicles.damageBodyCell(id, cell, amount)
```

`NETWORK`: Uses the network backend.

Adds damage to one body cell.

Requires `world.vehicles`. Adds the amount to the current normalized cell and clamps the result to 0..1.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `cell` | `integer` | required | — |
| `amount` | `number` | required | — |

Returns: `boolean success`

```lua
-- A minor collision: add 25 % damage to body cell 12
Open77.vehicles.damageBodyCell(vehicleId, 12, 0.25)
```

Registered in `LuaResourceRuntime.cs` (line 9128) as `LuaResourceRuntime.Bootstrap`.

## damageBodyZone

```lua
Open77.vehicles.damageBodyZone(id, zone, amount)
```

`NETWORK`: Uses the network backend.

Adds damage across a body zone.

Requires `world.vehicles`. Adds the amount to each selected body cell and clamps every result to 0..1.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `zone` | `string|table` | required | — |
| `amount` | `number` | required | — |

Returns: `boolean success`

```lua
-- Rear-ended: damage the whole back zone by half
Open77.vehicles.damageBodyZone(vehicleId, "back", 0.5)
```

Registered in `LuaResourceRuntime.cs` (line 9134) as `LuaResourceRuntime.Bootstrap`.

## detachedParts

```lua
Open77.vehicles.detachedParts()
```

`NETWORK`: Uses the network backend.

Maps stable detached-part names to network bit indexes.

Read-only constants for trunk, hood variants, door variants and front/back bumpers. Indexes are append-only protocol values in the range 0..15.

Returns: `constant table`

```lua
-- List every panel name you can pass to detachPart
for name, bit in pairs(Open77.vehicles.detachedParts) do
    print(name, bit)
end
```

Registered in `LuaResourceRuntime.cs` (line 9059) as `LuaResourceRuntime.Bootstrap`.

## detachPart

```lua
Open77.vehicles.detachPart(id, part)
```

`NETWORK`: Uses the network backend.

Detaches one vehicle panel authoritatively.

Requires `world.vehicles`. Convenience wrapper over `setPartDetached(id, part, true)`. The state is replicated and retained for late join.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `part` | `integer|string` | required | — |

Returns: `boolean success`

```lua
-- Bring the wreck to life visually: tear off the front bumper and the hood
Open77.vehicles.detachPart(vehicleId, "bumperFront")
Open77.vehicles.detachPart(vehicleId, "hood")
```

Registered in `LuaResourceRuntime.cs` (line 9173) as `LuaResourceRuntime.Bootstrap`.

## doors

```lua
Open77.vehicles.doors()
```

`NETWORK`: Uses the network backend.

Door indexes for the door natives.

A public constant table: `frontLeft = 0`, `frontRight = 1`, `backLeft = 2`, `backRight = 3`, `trunk = 4`, `hood = 5`.

Returns: `table { frontLeft = 0, frontRight = 1, backLeft = 2, backRight = 3, trunk = 4, hood = 5 }`

```lua
Open77.vehicles.setDoorOpen(id, Open77.vehicles.doors.trunk, true)
```

Registered in `LuaResourceRuntime.cs` (line 9051) as `LuaResourceRuntime.Bootstrap`.

## explode

```lua
Open77.vehicles.explode(id)
```

`NETWORK`: Uses the network backend.

Detonates a vehicle through every viewer's own engine.

Requires `world.vehicles`. Writes the canonical ledger -- health to zero, `destroyed` and `exploded` raised, one revision -- and every client that has the vehicle streamed then reconciles its projection through the damage path that already replays a wreck to a late joiner, which ends in REDengine's own explosion on that machine. **Occupants stay occupants**: nobody is ejected, no seat is freed, no life state changes, and the server sends no damage to any player, inside the car or standing next to it. What the blast does to a body is the engine's own damage model running on each client, reaching the server the ordinary way through the player damage report -- a resource that wants the occupants dead has to say so itself. **Terminal and not repeatable**: a second call is refused with `already_exploded` rather than re-detonating a wreck, while a vehicle that is merely `destroyed` -- written off by collisions, never blown up -- is still a legitimate target. An owner report taken before the explosion and claiming a pristine car cannot undo it: health merges by minimum and the destruction bits by OR, both monotone. **Only server-spawned vehicles are in the registry.** A player standing next to a vanilla traffic car is standing next to something REDengine population spawned on their own client: it has no canonical id and this call cannot see it. Adopting vanilla traffic is a separate capability that does not exist yet.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `true, or false`, `reason: already_exploded, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- A car bomb: the fuse runs ten seconds after the player is warned
RegisterCommand("carbomb", function(source, args)
    local vehicleId = tonumber(args[1])
    if not vehicleId then return end
    local driver = Open77.vehicles.getDriver(vehicleId)
    if driver then Open77.chat.send(driver, "Something is beeping under your seat...") end
    SetTimeout(10000, function()
        local ok, reason = Open77.vehicles.explode(vehicleId)
        if not ok then print("explode refused: " .. reason) end
    end)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## flags

```lua
Open77.vehicles.flags()
```

`NETWORK`: Uses the network backend.

Bit values of a vehicle's `flags` field.

A public constant table for the `flags` integer a vehicle snapshot carries: `engineOn = 1`, `locked = 2`, `destroyed = 4`, `exploded = 8`, `invulnerable = 16`, `immortal = 32`, `lightsOn = 64`, `highBeams = 128`, `sirenOn = 256`, `paintApplied = 512` (set by the runtime once a paint job has been applied; not a `create` option). Test a bit with `flags & Open77.vehicles.flags.locked ~= 0`; `create` accepts the other nine names as booleans in its definition.

Returns: `table { engineOn = 1, locked = 2, destroyed = 4, exploded = 8, invulnerable = 16, immortal = 32, lightsOn = 64, highBeams = 128, sirenOn = 256, paintApplied = 512 }`

```lua
local car = Open77.vehicles.get(id)
if car and (car.flags & Open77.vehicles.flags.locked) ~= 0 then
    Open77.chat.send(source, "That car is locked.")
end
```

Registered in `LuaResourceRuntime.cs` (line 9046) as `LuaResourceRuntime.Bootstrap`.

## forcePlayerOutOfVehicle

```lua
Open77.vehicles.forcePlayerOutOfVehicle(playerId, [vehicleId])
```

`NETWORK`: Uses the network backend.

Forces a player out of a network vehicle.

Requires `world.vehicles`. Publishes a durable forced-exit order and keeps the canonical seat reserved until the client confirms native unmount or the bounded transition timeout expires. This operation overrides an active exit lock. Supplying `vehicleId` protects against ejecting the player from a newer assignment by mistake.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Police stop: pull the driver out of whatever they are driving
RegisterCommand("pullout", function(source, args)
    local target = tonumber(args[1])
    if not target then return end
    local ok, reason = Open77.vehicles.forcePlayerOutOfVehicle(target)
    if not ok then Open77.chat.send(source, "Could not eject: " .. reason) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 9077) as `LuaResourceRuntime.Bootstrap`.

## freeSeats

```lua
Open77.vehicles.freeSeats(id)
```

`NETWORK`: Uses the network backend.

Every free seat of a vehicle, in seat order.

Requires `world.vehicles`. Canonical `seat_*` names in `driver, frontPassenger, rearLeft, rearRight` order, so picking `[1]` gives a deterministic 'put them anywhere' seat. Every seat of the four-seat model is considered; a two-seat record still reports its rear seats free, because the registry does not know the model's seat count.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `array of seat names, or nil`, `reason: vehicle_not_found`

```lua
-- Taxi: put the customer in the first free seat, whatever it is
local seats = Open77.vehicles.freeSeats(taxiId)
if seats and seats[1] then
    Open77.vehicles.warpPlayerIntoVehicle(customerId, taxiId, seats[1])
else
    Open77.chat.send(customerId, "The taxi is full.")
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.vehicles.get(id)
```

`NETWORK`: Uses the network backend.

Returns one authoritative server vehicle snapshot.

Reads canonical identity, transform, motion, flags, colors, all damage channels including detached parts, openings, occupants, lifetime, any performance ceiling, and authority state. Returns `nil` for an unknown ID. The transform half is `position` (a vector3), `heading` and `yaw` (one value, both names), `orientation` as `{ x, y, z, w }`, and the unchanged `x`/`y`/`z`. The motion half is `velocity`, `angularVelocity`, `speed` (m/s), `speedKph`, `onGround`, `reversing`, `moving` and `driverPlayerId` -- all of it the last thing the physics owner reported, and all of it zeroed when the vehicle has no owner, because nobody is simulating it then. `exploded` and `destroyed` are derived booleans over the same flags; `persistent`, `despawnWhenUnobserved` and `ttlMs` report the lifetime policy, `ttlMs` counted from now and absent when there is no deadline.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `vehicle table, or `nil``

```lua
-- Speed camera: check every car's speed once a second
CreateThread(function()
    while true do
        Wait(1000)
        for _, entry in ipairs(Open77.vehicles.all()) do
            local car = Open77.vehicles.get(entry.id)
            if car and car.speedKph > 120 and car.driverPlayerId then
                Open77.chat.send(car.driverPlayerId, ("Slow down! %.0f km/h"):format(car.speedKph))
            end
        end
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 9221) as `LuaResourceRuntime.Bootstrap`.

## getAdoptPolicy

```lua
Open77.vehicles.getAdoptPolicy([bucket])
```

`NETWORK`: Uses the network backend.

One routing bucket's adoption policy.

Requires `world.vehicles`. Deliberately **not** gated by `world.vehicles.adopt`: a resource may see that a bucket is closed without being allowed to open it. Answers `"none"`, `"driver"` or `"any"`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `policy name`

```lua
-- Tell a joining player whether they may steal street cars in this instance
AddEventHandler("onPlayerReady", function(playerId)
    local bucket = Open77.players.get(playerId).bucket
    if Open77.vehicles.getAdoptPolicy(bucket) ~= "none" then
        Open77.chat.send(playerId, "Street cars can be taken here.")
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getAppearance

```lua
Open77.vehicles.getAppearance(id)
```

`NETWORK`: Uses the network backend.

The entity appearance variant this car wears.

Requires `world.vehicles`. Answers the variant name, or `""` for the record's own default. The appearance variant is Cyberpunk's equivalent of a livery: a name defined by the vehicle's entity template, which is how the game ships one model in several trims.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `string, or nil`, `reason: vehicle_not_found, permission_denied:world.vehicles`

```lua
-- /livery: show which trim the caller's car wears
RegisterCommand("livery", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return end
    local variant = Open77.vehicles.getAppearance(seat.vehicleId)
    Open77.chat.send(source, variant == "" and "Stock trim" or ("Trim: " .. variant))
end, false)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getDamage

```lua
Open77.vehicles.getDamage(id)
```

`NETWORK`: Uses the network backend.

Returns the canonical damage snapshot.

Reads the server-owned body grid plus broken glass, light, tire, and detached-part masks. It returns `nil` when the vehicle ID is unknown.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `damage table, or `nil``

```lua
-- Mechanic's quote: count broken tires and detached panels
local damage = Open77.vehicles.getDamage(vehicleId)
if damage then
    print("tires mask:", damage.tires, "detached parts mask:", damage.detachedParts)
end
```

Registered in `LuaResourceRuntime.cs` (line 9124) as `LuaResourceRuntime.Bootstrap`.

## getDamageProfile

```lua
Open77.vehicles.getDamageProfile(record)
```

`NETWORK`: Uses the network backend.

Returns a resource-local vehicle damage profile.

Reads the profile registered by the current server resource for the specified vehicle record. Profiles are not shared implicitly between resources.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `profile table, or `nil``

```lua
-- Confirm the profile this resource registered for the Hella is in place
local profile = Open77.vehicles.getDamageProfile("Vehicle.v_standard2_archer_hella_player")
if not profile then
    print("no damage profile registered for the Hella yet")
end
```

Registered in `LuaResourceRuntime.cs` (line 9211) as `LuaResourceRuntime.Bootstrap`.

## getDecalBudget

```lua
Open77.vehicles.getDecalBudget(id)
```

`NETWORK`: Uses the network backend.

Reads how many of a car's 12 decals are used.

Requires `world.vehicles`. Answers `used, maximum`: the decals the car shows now (its livery's plus its stickers') and the cap, 12.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `used, maximum, or nil`, `reason: vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
local used, maximum = Open77.vehicles.getDecalBudget(vehicleId)
print(("%d of %d decals"):format(used, maximum))
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## getDriver

```lua
Open77.vehicles.getDriver(id)
```

`NETWORK`: Uses the network backend.

The player driving this vehicle, or nil when nobody is.

Requires `world.vehicles`. The front-left occupant from the canonical seat ledger. Absent rather than zero when the seat is empty, and absent while an entry animation is still running, because a reserved seat is not yet a driver. Also on the read as `Open77.vehicles.get(id).driverPlayerId`, which is the field the occupants list is scanned for nine times out of ten.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `player id, or nil`, `reason: vehicle_not_found`

```lua
-- Charge the driver, not the passengers, when a car passes the toll gate
local driver = Open77.vehicles.getDriver(vehicleId)
if driver then
    Open77.chat.send(driver, "Toll: 50 eddies were taken from your account.")
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getEngineHealth

```lua
Open77.vehicles.getEngineHealth(id)
```

`NETWORK`: Uses the network backend.

Vehicle health under its FiveM name.

Requires `world.vehicles`. `Open77.vehicles.getHealth`; there is no separate engine pool on this build.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `number, or nil`, `reason`

```lua
-- Ported FiveM code: the engine "health" is the car's single health pool, 0..1
local health = Open77.vehicles.getEngineHealth(vehicleId)
if health and health < 0.3 then
    Open77.chat.send(driverId, "Your car is about to give up. Find a mechanic.")
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getEngineSound

```lua
Open77.vehicles.getEngineSound(id)
```

`NETWORK`: Uses the network backend.

Reads the engine sound set a car carries.

Requires `world.vehicles`. Answers the sound id the car's record carries, or `nil` for the factory sound -- also while the pack that declared the sound is stopped.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `soundId, or nil`, `reason: vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
print(Open77.vehicles.getEngineSound(vehicleId) or "factory")
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## getHeading

```lua
Open77.vehicles.getHeading(id)
```

`NETWORK`: Uses the network backend.

The vehicle's heading in degrees.

Requires `world.vehicles`. Extracted from the full orientation quaternion rather than from `qz`/`qw` alone, because a replicated vehicle carries roll and pitch too and a car on a slope would otherwise report a heading it is not facing. Identical to the client snapshot's `heading`, and in the same degrees as `Open77.players.get(...).heading` and the `yaw` that `create` and `setTransform` take. `Open77.vehicles.getYaw` is the same function under the other name.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `heading in degrees, or nil`, `reason: vehicle_not_found`

```lua
-- Spawn a second car parked right behind the first, facing the same way
local car = Open77.vehicles.get(vehicleId)
local heading = Open77.vehicles.getHeading(vehicleId)
Open77.vehicles.create({
    record = car.record,
    position = { x = car.position.x, y = car.position.y - 6.0, z = car.position.z },
    yaw = heading,
})
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getHeadlightHue

```lua
Open77.vehicles.getHeadlightHue(id)
```

`NETWORK`: Uses the network backend.

Alias for `getLightsHue`.

Identical contract, under the name a creator looking for headlights will try first.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `number 0..1 or nil, or nil`, `reason`

```lua
-- Save the headlight tint with the rest of the garage record
local hue = Open77.vehicles.getHeadlightHue(vehicleId)
saved.lightsHue = hue -- nil means stock lights
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getHealth

```lua
Open77.vehicles.getHealth(id)
```

`NETWORK`: Uses the network backend.

A vehicle's health, 0 to 1.

Requires `world.vehicles`. Canonical, and merged by MIN from the physics owner's report: an observer cannot heal a car by claiming it is fine.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `number, or nil`, `reason: vehicle_not_found`

```lua
-- Refuse the race entry with a damaged car
RegisterCommand("race-join", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return Open77.chat.send(source, "Sit in your race car first.") end
    local health = Open77.vehicles.getHealth(seat.vehicleId)
    if health < 0.9 then
        return Open77.chat.send(source, "Repair your car before joining.")
    end
    enterRace(source, seat.vehicleId)
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getLightColors

```lua
Open77.vehicles.getLightColors(id)
```

`NETWORK`: Uses the network backend.

Reads a car's light-group colours.

Requires `world.vehicles`. Answers `{ brake = {r,g,b}?, reverse?, blinkers?, interior? }`, or `nil` for factory lights.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `colors, or nil`, `reason: vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
local colors = Open77.vehicles.getLightColors(vehicleId) or {}
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## getLights

```lua
Open77.vehicles.getLights(id)
```

`NETWORK`: Uses the network backend.

A vehicle's headlight mode as a word.

Requires `world.vehicles`. Answers `"off"`, `"on"` or `"high"` -- the same three words `setLights` accepts, so a read feeds straight back into a write.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `string, or nil`, `reason: vehicle_not_found`

```lua
-- Flash the high beams twice, then put the lights back as they were
CreateThread(function()
    local before = Open77.vehicles.getLights(vehicleId)
    for _ = 1, 2 do
        Open77.vehicles.setLights(vehicleId, "high"); Wait(300)
        Open77.vehicles.setLights(vehicleId, "off"); Wait(300)
    end
    Open77.vehicles.setLights(vehicleId, before)
end)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getLightsHue

```lua
Open77.vehicles.getLightsHue(id)
```

`NETWORK`: Uses the network backend.

The CrystalCoat headlight hue in force, or nil.

Requires `world.vehicles`. Answers a number in `0..1`, or `nil` when the car uses its record's own light colour. `nil` rather than zero, because zero is a real hue -- red -- and the engine struct this mirrors carries its own `lightsColorDefined` boolean for exactly that reason. `getHeadlightHue` is the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `number 0..1 or nil, or nil`, `reason: vehicle_not_found, permission_denied:world.vehicles`

```lua
-- Tuning shop: show the current tint before offering a new one
local hue = Open77.vehicles.getLightsHue(vehicleId)
if hue == nil then
    Open77.chat.send(source, "Stock headlights.")
else
    Open77.chat.send(source, ("Current tint: %.2f"):format(hue))
end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getNeonAnimation

```lua
Open77.vehicles.getNeonAnimation(id)
```

`NETWORK`: Uses the network backend.

Reads how a car's neon moves.

Requires `world.vehicles`, like `getVisual`. Answers `{ mode, periodMs, color2 = {r,g,b}? }`, or `nil` for a steady neon.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `animation, or nil`, `reason: vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
local animation = Open77.vehicles.getNeonAnimation(vehicleId)
if animation then print(animation.mode, animation.periodMs) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## getPaint

```lua
Open77.vehicles.getPaint(id)
```

`NETWORK`: Uses the network backend.

Reads authoritative vehicle paint.

Returns `applied` plus keyed primary and secondary RGB tables. Pure black remains distinguishable from reset because `applied` is a separate canonical flag. Returns `nil` for an unknown vehicle ID.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `paint table, or `nil``

```lua
-- Save the paint job so the garage can restore it later
local paint = Open77.vehicles.getPaint(vehicleId)
if paint and paint.applied then
    saveGarage(ownerId, { primary = paint.primary, secondary = paint.secondary })
end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9123) as `LuaResourceRuntime.Bootstrap`.

## getPerformance

```lua
Open77.vehicles.getPerformance(id)
```

`NETWORK`: Uses the network backend.

The performance ceiling in force, or nil.

Requires `world.vehicles`. `{ topSpeedKph, accelerationScale, taperKph }`, or `nil` when the vehicle runs at its record's rated performance. Also on the read as `Open77.vehicles.get(id).performance`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `profile table, or nil`, `reason: vehicle_not_found`

```lua
-- Show the active speed cap to the driver
local cap = Open77.vehicles.getPerformance(vehicleId)
if cap then
    Open77.chat.send(driverId, ("This car is limited to %d km/h"):format(cap.topSpeedKph))
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getPlayerSeat

```lua
Open77.vehicles.getPlayerSeat(playerId)
```

`NETWORK`: Uses the network backend.

Reads one player's canonical vehicle-seat assignment.

Requires `world.vehicles`. Returns the server ledger entry for the player: a table `{ playerId, vehicleId (integer), seat (the canonical name only: `seat_front_left`, `seat_front_right`, `seat_back_left` or `seat_back_right` -- never `driver`, which is an *accepted* alias, not a reported one), flags (integer, bits of `Open77.vehicles.occupantFlags`), entering, exiting, forcedEntry, exitLocked, forcedExit (booleans) }`. A single `nil`, with no reason, when the player is not seated, is unknown, or the permission is missing. The same table shape describes each entry of a vehicle snapshot's `occupants` list and the second value of `occupantInSeat`. `vehicleId` is a plain integer; a vehicle id delivered by a host event (`onPlayerLeftVehicle`) is the same number as a string, so `tostring(seat.vehicleId)` compares equal to it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `seat assignment table { playerId, vehicleId, seat, flags, entering, exiting, forcedEntry, exitLocked, forcedExit }, or a single nil`

```lua
-- Only the driver may pay for fuel
RegisterCommand("refuel", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return Open77.chat.send(source, "You are not in a vehicle.") end
    if seat.seat ~= "seat_front_left" then
        return Open77.chat.send(source, "Only the driver can refuel.")
    end
    Open77.vehicles.repair(seat.vehicleId, "mechanical")
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 9080) as `LuaResourceRuntime.Bootstrap`.

## getPlayerSeatSwitch

```lua
Open77.vehicles.getPlayerSeatSwitch(playerId)
```

`NETWORK`: Uses the network backend.

Reads a player's active authoritative seat transition.

Requires world.vehicles. Returns id (string), playerId, vehicleId, fromSeat, toSeat and automatic. Returns nil when no transition is active; invalid players and missing permission return false plus reason.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `transition table, nil when idle, or false on refusal`, `reason`

```lua
local state = Open77.vehicles.getPlayerSeatSwitch(playerId)
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9014) as `LuaResourceRuntime.Bootstrap`.

## getPosition

```lua
Open77.vehicles.getPosition(id)
```

`NETWORK`: Uses the network backend.

The vehicle's canonical position as a vector3.

Requires `world.vehicles`. A plain `{ x, y, z }` table, which is what an Open77 vector3 is, so `#(a - b)` works on it without a conversion. The same point `Open77.vehicles.get(id).position` carries, and the same one `x`/`y`/`z` have always carried. **Only server-spawned vehicles are in the registry.** A player standing next to a vanilla traffic car is standing next to something REDengine population spawned on their own client: it has no canonical id and this call cannot see it. Adopting vanilla traffic is a separate capability that does not exist yet.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `position table, or nil`, `reason: vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- Tow truck job: is the caller close enough to hook the car?
local car = Open77.vehicles.getPosition(vehicleId)
local me = Open77.players.position(source)
if car and me and #(vector3(car.x, car.y, car.z) - vector3(me.x, me.y, me.z)) < 5.0 then
    startTow(source, vehicleId)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getProperties

```lua
Open77.vehicles.getProperties(id)
```

`NETWORK`: Uses the network backend.

Everything a garage has to store to put this exact car back.

Requires `world.vehicles`. Answers one plain table -- strings, numbers, booleans and tables, so it survives `json.encode` and a database column unchanged -- or `nil, "vehicle_not_found"`. This is the Cyberpunk counterpart of the `GetVehicleProperties` helper every FiveM framework ships, and `setProperties` is its other half: **`setProperties(id, getProperties(id))` leaves the vehicle observably unchanged**, and the same table applied to a fresh vehicle of the same record reproduces it.

The shape is `{ version = 1, record, appearance, paint = { applied, primary = {r,g,b}, secondary = {r,g,b}, lightsHue? }, health, engineOn, engineHold, lights = "off"|"on"|"high", siren, locked, drivable, invulnerable, immortal, doors, windows, damage = { body = {30 cells}, glass, lights, tires, detachedParts, destroyed, exploded }, performance? }`.

**Deliberately not where the car was standing.** Position, heading, velocity, occupants, routing bucket and revision are not properties of the car, and a table that carried them would tempt a caller into teleporting by accident; `setTransform` and `setBucket` still own those.

**What Cyberpunk does not have is the real story here.** Vehicle visual customisation on build 2.31 is one struct, `GenericTemplatePersistentData` -- primary RGB, secondary RGB and a lights hue -- plus the entity appearance variant. The engine has no mod slots, no wheel types, no neon, no window tint and no numbered extras: not unimplemented, absent, with no RTTI type for any of them. `Open77.vehicles.unsupportedProperties` names every FiveM property that therefore has no counterpart, and `setProperties` refuses such a key by name rather than dropping it. Stable 147 includes engineHold in the property table; setProperties applies it through the canonical engine-hold setter.

**From 2.50.0-unstable+op77.205:** the table also carries `visual = { profile?, neon = { enabled, color = {r,g,b}, intensity, radius, layout }?, parts = { [partId] = variant | true }, wheels = { rim, variant? }?, plate?, tint?, neonAnimation = { mode, periodMs, color2? }?, lightColors = { brake?, reverse?, blinkers?, interior? }?, engineSound?, stickers = { id, ... }? }`, the look Open77 adds on the server: a TwinTone paint profile (`setPaintProfile`), neon (`setNeon`), kit parts and rims (`registerParts`, `setParts`, `setWheels`), plate text (`setPlate`), a window tint level (`setWindowTint`), a neon animation (`setNeonAnimation`), light-group colours (`setLightColors`) and stickers (`setStickers`); `visual.engineSound` names the engine sound the car's record carries (`setEngineSound`). `visual` is `getVisual(id)` without `rev`: `parts` is always there (empty when the car wears none, and it lists the rim too); the other fields are absent when unset. Neon, its animation and the light colours need no pack; rims, kit parts, plates, tint, pack liveries, engine sounds and stickers are drawn only by a client that has the pack. A server on the current public build answers no `visual`, and its `setProperties` refuses one as `unsupported_property:visual`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `table, or nil`, `reason: vehicle_not_found, permission_denied:world.vehicles`

```lua
-- /park: store the caller's car so it can be brought back after a restart
RegisterCommand("park", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return end
    local props = Open77.vehicles.getProperties(seat.vehicleId)
    Open77.kvp.set("garage:" .. Open77.players.identifier(source), json.encode(props))
    Open77.vehicles.remove(seat.vehicleId)
    Open77.chat.send(source, "Car stored.")
end, false)
```

[Vehicle properties, and what Cyberpunk does not have](../../01%20Documentation/vehicles.md#vehicle-properties-and-what-cyberpunk-does-not-have)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## getSpeed

```lua
Open77.vehicles.getSpeed(id)
```

`NETWORK`: Uses the network backend.

Scalar speed in metres per second.

Requires `world.vehicles`. The physics owner's own drivetrain reading, in the unit every other Open77 read uses. **Motion is the last thing the owner reported**: a vehicle with no physics owner is not being simulated by anyone, so losing the lease -- a driver stepping out, disconnecting, or the lease expiring -- zeroes this rather than leaving a parked car claiming the speed it had when its driver got out. A server `setTransform` clears it for the same reason.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `speed in m/s, or nil`, `reason: vehicle_not_found`

```lua
-- Rear-end insurance: only pay out if the car was actually moving
local speed = Open77.vehicles.getSpeed(vehicleId)
if speed and speed > 5.0 then
    payInsurance(driverId)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getSpeedKph

```lua
Open77.vehicles.getSpeedKph(id)
```

`NETWORK`: Uses the network backend.

Scalar speed in kilometres per hour.

Requires `world.vehicles`. The same reading as `Open77.vehicles.getSpeed` converted once, because speed limits and performance ceilings are authored in km/h and every caller converting it themselves is a rounding bug waiting to happen.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `speed in km/h, or nil`, `reason: vehicle_not_found`

```lua
-- Radar: report who is over the limit on this road
for _, entry in ipairs(Open77.vehicles.nearby(radarPosition, 60)) do
    local kph = Open77.vehicles.getSpeedKph(entry.id)
    if kph and kph > 80 then
        print(("vehicle %d flashed at %.0f km/h"):format(entry.id, kph))
    end
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getStickers

```lua
Open77.vehicles.getStickers(id)
```

`NETWORK`: Uses the network backend.

Reads the stickers a car wears.

Requires `world.vehicles`. Answers `{ stickerId, ... }` in the order they were set, or `nil` for none.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `stickers, or nil`, `reason: vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
local stickers = Open77.vehicles.getStickers(vehicleId) or {}
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## getVehicleProperties

```lua
Open77.vehicles.getVehicleProperties(id)
```

`NETWORK`: Uses the network backend.

The FiveM-shaped name for `getProperties`.

Identical contract. The name exists so a resource ported from a FiveM framework, where the helper is spelled `GetVehicleProperties`, does not have to be renamed to be read.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `table, or nil`, `reason`

```lua
-- FiveM-style garage code works unchanged
RegisterCommand("park", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return end
    local props = Open77.vehicles.getVehicleProperties(seat.vehicleId)
    MySQL.update.await("UPDATE owned_vehicles SET props = ? WHERE owner = ?", { json.encode(props), Open77.players.identifier(source) })
end, false)
```

[Vehicle properties, and what Cyberpunk does not have](../../01%20Documentation/vehicles.md#vehicle-properties-and-what-cyberpunk-does-not-have)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getVelocity

```lua
Open77.vehicles.getVelocity(id)
```

`NETWORK`: Uses the network backend.

The vehicle's linear velocity as a vector3, in metres per second.

Requires `world.vehicles`. A plain `{ x, y, z }` table. Like `getSpeed`, it is the last accepted owner report and is zeroed when the physics lease is lost or the server teleports the vehicle. `Open77.vehicles.get(id).angularVelocity` carries the rotational half.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `velocity table, or nil`, `reason: vehicle_not_found`

```lua
-- Is the car sliding sideways? Compare velocity with the heading
local v = Open77.vehicles.getVelocity(vehicleId)
if v and math.abs(v.x) + math.abs(v.y) > 20 then
    print("fast lateral movement on vehicle", vehicleId)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## getVisual

```lua
Open77.vehicles.getVisual(id)
```

`NETWORK`: Uses the network backend.

Reads the customization a car wears: livery, neon, parts, rims, plate, tint, neon animation, light colours, engine sound and stickers.

Requires `world.vehicles`. Answers `{ rev, profile?, neon = { enabled, color = {r,g,b}, intensity, radius, layout }?, parts = { [partId] = variant | true }, wheels = { rim, variant? }?, plate?, tint?, neonAnimation = { mode, periodMs, color2? }?, lightColors = { brake?, reverse?, blinkers?, interior? }?, engineSound?, stickers = { id, ... }? }`. `plate` is the normalized (upper-case) text; `tint` is absent for factory glass. The wave 2 fields are absent when unset; `engineSound` is the sound id the car's record carries (nil for the factory sound, or while its pack is stopped), and `stickers` the chosen sticker ids in order. `profile` and `parts` are the choices: one whose pack is stopped, or that the car's current record does not list, is still answered while it is not shown. `rev` grows with every change. The host event `onVehicleVisualChanged(id, rev, fields)` reports changes, `fields` listing what moved among `profile`, `neon`, `parts`, `wheels`, `plate`, `tint`, `neonAnimation`, `lightColors` and `stickers`; an engine sound is a record change and is not listed. `getProperties` carries the same table (without `rev`) as `visual`, which `setProperties` restores.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `visual table, or nil`, `reason: vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- Save the car's look with the garage row
local visual = Open77.vehicles.getVisual(vehicleId)
if visual then MySQL.update("UPDATE garage SET visual = ? WHERE id = ?", { json.encode(visual), rowId }) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## getWindowTint

```lua
Open77.vehicles.getWindowTint(id)
```

`NETWORK`: Uses the network backend.

Reads a car's window tint level.

Requires `world.vehicles`, like `getVisual`. Answers `0` for factory glass, else the level from 1 to 3.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `level, or nil`, `reason: vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
local level = Open77.vehicles.getWindowTint(vehicleId) or 0
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## getYaw

```lua
Open77.vehicles.getYaw(id)
```

`NETWORK`: Uses the network backend.

Alias of `Open77.vehicles.getHeading`.

Requires `world.vehicles`. `heading` is the name the player read and `respawn` use; `yaw` is the name `create` and `setTransform` use. One value, both names, so nobody has to remember which surface they are on.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `heading in degrees, or nil`, `reason: vehicle_not_found`

```lua
-- Respawn the player's car at the same spot after a repair, facing the same way
local pos = Open77.vehicles.getPosition(vehicleId)
local yaw = Open77.vehicles.getYaw(vehicleId)
Open77.vehicles.remove(vehicleId)
Open77.vehicles.create({ record = record, position = pos, yaw = yaw })
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## honk

```lua
Open77.vehicles.honk(id, [durationMs])
```

`NETWORK`: Uses the network backend.

Alias of `Open77.vehicles.triggerHorn`.

Requires `world.vehicles` and uses the same current-owner routing, duration bounds, and transient synchronization contract as `triggerHorn`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `durationMs` | `integer` | optional | `250` |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The taxi has arrived: honk twice
CreateThread(function()
    Open77.vehicles.honk(taxiId, 300)
    Wait(500)
    Open77.vehicles.honk(taxiId, 300)
end)
```

Registered in `LuaResourceRuntime.cs` (line 9120) as `LuaResourceRuntime.Bootstrap`.

## isDoorOpen

```lua
Open77.vehicles.isDoorOpen(id, door)
```

`NETWORK`: Uses the network backend.

Reads one canonical door opening state.

Server-side authoritative query covering four doors, trunk, and hood. Returns `nil` for an unknown vehicle ID.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `door` | `integer` | required | — |

Returns: `boolean, or `nil``

```lua
-- Refuse to drive off with the trunk open
if Open77.vehicles.isDoorOpen(vehicleId, Open77.vehicles.doors.trunk) then
    Open77.chat.send(driverId, "Close the trunk first.")
end
```

Registered in `LuaResourceRuntime.cs` (line 9191) as `LuaResourceRuntime.Bootstrap`.

## isDrivable

```lua
Open77.vehicles.isDrivable(id)
```

`NETWORK`: Uses the network backend.

Whether a vehicle is in service.

Requires `world.vehicles`. False once `setDrivable(id, false)` has been applied. Independent of health: a wreck with a live engine is still 'drivable' by this flag.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `boolean, or nil`, `reason: vehicle_not_found`

```lua
-- Impound: a car that was taken out of service cannot be sold
if not Open77.vehicles.isDrivable(vehicleId) then
    return Open77.chat.send(source, "This car is impounded.")
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## isEngineHeld

```lua
Open77.vehicles.isEngineHeld(id)
```

`SERVER`: Available in server resources only.

Read the canonical engine-hold flag.

Returns the server-owned hold, default false, rather than native ignition readiness. Requires world.vehicles and Stable 147 or later. An unknown vehicle returns nil, vehicle_not_found.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean, or nil`, `reason on failure`

```lua
local held, reason = Open77.vehicles.isEngineHeld(vehicleId)
```

[Keep an empty vehicle running](../../01%20Documentation/vehicles.md#keep-an-empty-vehicle-running)

Registered in `LuaResourceRuntime.Vehicles.cs` (line 810) as `LuaResourceRuntime.Bootstrap`.

## isEngineOn

```lua
Open77.vehicles.isEngineOn(id)
```

`NETWORK`: Uses the network backend.

Whether a vehicle's engine is running.

Requires `world.vehicles`. Reads the canonical `engineOn` flag, which is what every client applies -- not a local guess. A read answers `nil` plus a reason where a write answers `false`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `boolean, or nil`, `reason: vehicle_not_found`

```lua
-- Fuel drains only while the engine runs
CreateThread(function()
    while true do
        Wait(5000)
        for _, entry in ipairs(Open77.vehicles.all()) do
            if Open77.vehicles.isEngineOn(entry.id) then
                fuel[entry.id] = math.max(0, (fuel[entry.id] or 100) - 1)
            end
        end
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## isFrozen

```lua
Open77.vehicles.isFrozen(id)
```

`NETWORK`: Uses the network backend.

Whether a vehicle is pinned by the server.

Requires `world.vehicles`. True once `setFrozen(id, true)` has been applied; the same bit as `get(id).flags & Open77.vehicles.flags.frozen`, and the `frozen` field of the client snapshot. Independent of `isDrivable`: the two bits compose.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `boolean, or nil`, `reason: vehicle_not_found`

```lua
-- A starting grid releases only cars it froze itself
for _, id in ipairs(gridCars) do
    if Open77.vehicles.isFrozen(id) then Open77.vehicles.setFrozen(id, false) end
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## isLocked

```lua
Open77.vehicles.isLocked(id)
```

`NETWORK`: Uses the network backend.

Reads a vehicle's canonical entry-lock state.

Returns the derived durable lock boolean without requiring callers to decode the complete flags mask. Unknown vehicles return `nil`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean, or `nil``

```lua
-- /lock toggles the lock of the caller's car
RegisterCommand("lock", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return end
    local locked = Open77.vehicles.isLocked(seat.vehicleId)
    Open77.vehicles.setLocked(seat.vehicleId, not locked)
    Open77.chat.send(source, locked and "Unlocked." or "Locked.")
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 9115) as `LuaResourceRuntime.Bootstrap`.

## isLockedForPlayer

```lua
Open77.vehicles.isLockedForPlayer(id, playerId)
```

`NETWORK`: Uses the network backend.

Is this vehicle locked as far as this one player is concerned?

Requires `world.vehicles`. The player's exception if they hold one, otherwise the canonical bit -- the single function that decides what a client is told, so a caller can never disagree with the fan-out. `Open77.vehicles.isLocked` still answers the canonical question, which is what a late joiner with no exception will be told.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `playerId` | `integer` | required | — |

Returns: `boolean, or nil`, `reason: invalid_player_id, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- Tell the player whether their own keys open this car
RegisterCommand("trykeys", function(source, args)
    local vehicleId = tonumber(args[1])
    if not vehicleId then return end
    local locked = Open77.vehicles.isLockedForPlayer(vehicleId, source)
    Open77.chat.send(source, locked and "Your keys do not fit." or "The door opens.")
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## isPartDetached

```lua
Open77.vehicles.isPartDetached(id, part)
```

`NETWORK`: Uses the network backend.

Reads one canonical detached-part bit.

Returns whether a named/indexed panel is detached, or `nil` when the vehicle ID is unknown.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `part` | `integer|string` | required | — |

Returns: `boolean, or `nil``

```lua
-- The scrapyard pays less for a car missing its hood
local price = 1000
if Open77.vehicles.isPartDetached(vehicleId, "hood") then
    price = price - 200
end
```

Registered in `LuaResourceRuntime.cs` (line 9176) as `LuaResourceRuntime.Bootstrap`.

## isPlayerAutoSeatSwitchEnabled

```lua
Open77.vehicles.isPlayerAutoSeatSwitchEnabled(playerId)
```

`NETWORK`: Uses the network backend.

Reads the player's effective server takeover policy.

Requires world.vehicles. False while any server resource holds an automatic takeover veto. Client resources can independently veto their local automatic behavior.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean`, `reason on refusal`

```lua
local enabled = Open77.vehicles.isPlayerAutoSeatSwitchEnabled(playerId)
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9014) as `LuaResourceRuntime.Bootstrap`.

## isPlayerExitLocked

```lua
Open77.vehicles.isPlayerExitLocked(playerId)
```

`NETWORK`: Uses the network backend.

Reports whether a player's canonical seat has exit lock enabled.

Returns true only when the player currently has a seat assignment carrying the durable `exitLocked` policy. An unseated or unknown player returns false.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |

Returns: `boolean`

```lua
-- The arrested player stays in the police car until released
if Open77.vehicles.isPlayerExitLocked(suspectId) then
    Open77.chat.send(suspectId, "You cannot leave the vehicle.")
end
```

Registered in `LuaResourceRuntime.cs` (line 9081) as `LuaResourceRuntime.Bootstrap`.

## isSirenOn

```lua
Open77.vehicles.isSirenOn(id)
```

`NETWORK`: Uses the network backend.

Whether a vehicle's siren is on.

Requires `world.vehicles`. Canonical: no client can report this bit, so the server's value is the only value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `boolean, or nil`, `reason: vehicle_not_found`

```lua
-- Only a cruiser with its siren on may run a red light
if Open77.vehicles.isSirenOn(vehicleId) then
    return -- no fine
end
```

[Vehicle sirens & key binding](../../01%20Documentation/vehicle-sirens.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## isVisibleTo

```lua
Open77.vehicles.isVisibleTo(vehicleId, playerId)
```

`NETWORK`: Uses the network backend.

Reads actual server vehicle visibility for one player.

Requires `world.vehicles`. Arguments are vehicleId first, playerId second. Returns whether the server currently includes this vehicle in that player's visibility set, not a distance estimate or proof of native projection readiness. Returns false when it is not visible or the vehicle subsystem is unavailable. Malformed IDs return false, invalid_argument; missing permission returns false, permission_denied:world.vehicles.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `playerId` | `integer` | required | — |

Returns: `boolean`, `reason on refusal: invalid_argument, permission_denied:world.vehicles`

```lua
-- Keep an observed chassis out of an off-screen cleanup pass
local visible, reason = Open77.vehicles.isVisibleTo(vehicleId, playerId)
if reason then error(reason) end
if visible then return end
```

Registered in `LuaResourceRuntime.cs` (line 9111) as `LuaResourceRuntime.Bootstrap`.

## isWindowOpen

```lua
Open77.vehicles.isWindowOpen(id, window)
```

`NETWORK`: Uses the network backend.

Reads one canonical side-window opening state.

Server-side authoritative query for indexes 0..3. It does not report broken glass and returns `nil` for an unknown vehicle.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `window` | `integer` | required | — |

Returns: `boolean, or `nil``

```lua
-- Drive-through: the customer must have the window down
if not Open77.vehicles.isWindowOpen(vehicleId, Open77.vehicles.windows.frontLeft) then
    Open77.chat.send(driverId, "Roll down your window to order.")
end
```

Registered in `LuaResourceRuntime.cs` (line 9201) as `LuaResourceRuntime.Bootstrap`.

## listEngineSounds

```lua
Open77.vehicles.listEngineSounds(idOrRecord)
```

`NETWORK`: Uses the network backend.

Lists the engine sound sets a car can take.

Takes a vehicle id (`world.vehicles`) or a record name (no permission). Answers `{ { id, label, class }, ... }`: the sets of the car's class (`car` or `bike`) whose pack generated a record for this model at its current stage, in catalogue order. Empty when no running resource registered one.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `idOrRecord` | `vehicle id|string` | required | — |

Returns: `list, or nil`, `reason: invalid_record, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
for _, sound in ipairs(Open77.vehicles.listEngineSounds(vehicleId) or {}) do print(sound.id, sound.label) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## listPaintProfiles

```lua
Open77.vehicles.listPaintProfiles(recordOrId)
```

`NETWORK`: Uses the network backend.

Lists the TwinTone colour templates a record or a car accepts.

Answers `{ { id, label?, main = {r,g,b}?, sec = {r,g,b}?, appearances = {...}, hasPattern, resource?, decals? } ... }`: the build's own templates first, in table order (a FiveM livery index counts in this list), then those running resources declared with `registerPaintProfiles`, each carrying its declaring `resource`. A record name needs no permission (it is catalogue data); a vehicle id reads a car and needs `world.vehicles`. A record without customization answers `{}`; a performance-stage record (`Vehicle.<base>_o77sN`) answers its base record's list plus anything registered on the stage. `decals` is the number of decals the template spends of the car's 12-decal budget (see `getDecalBudget`), present only when it spends any.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `recordOrId` | `string|vehicle id` | required | — |

Returns: `profile list, or nil`, `reason: invalid_record, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- Offer the liveries this car accepts
local profiles = Open77.vehicles.listPaintProfiles(vehicleId) or {}
for _, p in ipairs(profiles) do
    print(p.id, p.label or p.id, p.hasPattern and "pattern" or "two-tone")
end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## listParts

```lua
Open77.vehicles.listParts(recordOrId)
```

`NETWORK`: Uses the network backend.

Lists the registered kit parts and rims that fit a record or a car.

Answers `{ { id, kind, label?, slot?, carrier?, modType?, variants = { [variant] = model } } ... }`, ids fully qualified (`<resource>:<id>`). A record name needs no permission; a vehicle id needs `world.vehicles`. `carrier` is the opening panel slot a body part rides on (`trunk_a`, `hood_a`), as `registerParts` declared it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `recordOrId` | `string|vehicle id` | required | — |

Returns: `part list, or nil`, `reason: invalid_record, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
for _, part in ipairs(Open77.vehicles.listParts(vehicleId) or {}) do
    print(part.id, part.kind, part.slot or "")
end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## listStickers

```lua
Open77.vehicles.listStickers(idOrRecord)
```

`NETWORK`: Uses the network backend.

Lists the stickers that fit a car.

Takes a vehicle id (`world.vehicles`) or a record name (no permission; stickers bound to an appearance are then not listed). Answers `{ { id, label, group, decals }, ... }`, by group then id; `decals` is what one sticker costs against the 12-decal budget.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `idOrRecord` | `vehicle id|string` | required | — |

Returns: `list, or nil`, `reason: invalid_record, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
for _, s in ipairs(Open77.vehicles.listStickers(vehicleId) or {}) do print(s.group, s.label, s.decals) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## lock

```lua
Open77.vehicles.lock(id)
```

`NETWORK`: Uses the network backend.

Locks a vehicle against ordinary entry.

Convenience alias for `setLocked(id, true)`. Requires `world.vehicles` and updates durable canonical state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The rental is over: lock the car so nobody else drives it away
Open77.vehicles.lock(rentalId)
Open77.chat.send(renterId, "Your rental has ended.")
```

Registered in `LuaResourceRuntime.cs` (line 9113) as `LuaResourceRuntime.Bootstrap`.

## lockForAll

```lua
Open77.vehicles.lockForAll(id)
```

`NETWORK`: Uses the network backend.

Locks a vehicle for everyone, revoking every exception.

Requires `world.vehicles`. `Open77.vehicles.setLockedForAll(id, true)` under a name that reads as an action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `true, or false`, `reason: vehicle_not_found`

```lua
-- Impound a car: everyone loses access, including previous key holders
RegisterCommand("impound", function(source, args)
    local vehicleId = tonumber(args[1])
    if not vehicleId then return end
    Open77.vehicles.lockForAll(vehicleId)
    Open77.vehicles.setDrivable(vehicleId, false)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## nearby

```lua
Open77.vehicles.nearby(anchor, [radius], [options])
```

`NETWORK`: Uses the network backend.

Every registered vehicle within a radius of a point or a player, nearest first.

Requires `world.vehicles`. Deliberately the same anchor, bucket and limit conventions as `Open77.players.nearby`: the anchor is a position table (a vector3 is one) or a player id, a player anchor defaults to that player's own routing bucket, `bucket = false` opts back out to every bucket, and ties are broken by vehicle id so the order does not reshuffle between calls. `occupied = true` keeps only cars with someone in them and `occupied = false` only empty ones. Each entry is `{ id, record, resource, bucket, distance, occupants, locked, position }` -- the placement, not the whole snapshot; call `Open77.vehicles.get` on the one you picked. **Only server-spawned vehicles are in the registry.** A player standing next to a vanilla traffic car is standing next to something REDengine population spawned on their own client: it has no canonical id and this call cannot see it. Adopting vanilla traffic is a separate capability that does not exist yet.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `anchor` | `vector3|table { x, y, z }|playerId` | required | — |
| `radius` | `number` | optional | — |
| `options` | `table { bucket?, limit?, occupied? }` | optional | — |

Returns: `array of entries, or nil`, `reason: invalid_argument, invalid_position, invalid_radius, invalid_bucket, invalid_limit, invalid_options, player_not_found, position_unknown`

```lua
-- /nearcars: list the server vehicles within 30 m of the caller
RegisterCommand("nearcars", function(source)
    local list = Open77.vehicles.nearby(source, 30, { limit = 5 })
    if not list or #list == 0 then return Open77.chat.send(source, "Nothing nearby.") end
    for _, entry in ipairs(list) do
        Open77.chat.send(source, ("#%d %s at %.1f m"):format(entry.id, entry.record, entry.distance))
    end
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## occupantFlags

```lua
Open77.vehicles.occupantFlags()
```

`NETWORK`: Uses the network backend.

Bit values of a seat assignment's `flags` field.

A public constant table for the `flags` integer of a seat assignment (`getPlayerSeat`, occupants): `entering = 1`, `exiting = 2`, `forcedEntry = 4`, `exitLocked = 8`, `forcedExit = 16`. The same assignment table also carries each of them as a boolean field, which is the easier read.

Returns: `table { entering = 1, exiting = 2, forcedEntry = 4, exitLocked = 8, forcedExit = 16 }`

```lua
local seat = Open77.vehicles.getPlayerSeat(playerId)
if seat and (seat.flags & Open77.vehicles.occupantFlags.entering) ~= 0 then
    -- still climbing in; wait for the seat to settle
end
```

Registered in `LuaResourceRuntime.cs` (line 9056) as `LuaResourceRuntime.Bootstrap`.

## occupantInSeat

```lua
Open77.vehicles.occupantInSeat(id, seat)
```

`NETWORK`: Uses the network backend.

Who is in one seat of a vehicle.

Requires `world.vehicles`. The `GetPedInVehicleSeat` of FiveM. Three answers, told apart by the second value: `playerId, record` when someone holds the seat (the record is the seat-assignment table `getPlayerSeat` returns: `playerId`, `vehicleId`, `seat`, `flags`, `entering`, `exiting`, `forcedEntry`, `exitLocked`, `forcedExit`); a single `nil` when the seat is free; `nil, reason` (a string: `invalid_seat`, `vehicle_not_found`) when the call itself failed. So `local who, extra = ...; if who then ... elseif type(extra) == "string" then error ... else free`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `seat` | `integer|string` | required | — |

Returns: `playerId (integer) and the occupant's seat-assignment table; a single nil for a free seat`, `reason (string) only when the call failed: invalid_seat, vehicle_not_found`

```lua
-- The passenger seat decides who pays the taxi fare
local passenger = Open77.vehicles.occupantInSeat(taxiId, Open77.vehicles.seats.frontPassenger)
if passenger then
    chargeFare(passenger, 120)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## openDoor

```lua
Open77.vehicles.openDoor(id, door)
```

`NETWORK`: Uses the network backend.

Opens a door, trunk, or hood.

Requires `world.vehicles`. Convenience wrapper over `setDoorOpen(id, door, true)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `door` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Pop the trunk when the delivery is loaded
Open77.vehicles.openDoor(vehicleId, Open77.vehicles.doors.trunk)
```

Registered in `LuaResourceRuntime.cs` (line 9189) as `LuaResourceRuntime.Bootstrap`.

## openWindow

```lua
Open77.vehicles.openWindow(id, window)
```

`NETWORK`: Uses the network backend.

Opens one side window.

Requires `world.vehicles`. Convenience wrapper over `setWindowOpen(id, window, true)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `window` | `integer` | required | — |

Returns: `boolean success`

```lua
-- The driver stops at the ticket booth: roll down the window
Open77.vehicles.openWindow(vehicleId, Open77.vehicles.windows.frontLeft)
```

Registered in `LuaResourceRuntime.cs` (line 9199) as `LuaResourceRuntime.Bootstrap`.

## owner

```lua
Open77.vehicles.owner(id)
```

`NETWORK`: Uses the network backend.

Who is simulating a vehicle, in what capacity, and since when.

Requires `world.vehicles`. FiveM's `NetworkGetEntityOwner` with more than a number, because the lease has more than one shape. `physicsOwner` and `authorityPlayerId` carry the same value (the vehicle read has always said the first, the NPC read says the second; a script reading both should not learn two words) and are `0` when the car is parked. `driver` is the seated controller -- the front-left occupant, or any occupant of an AV -- whether or not they own the lease, and is absent when nobody is. `steward` is true when the owner holds the lease without being the driver: the parked-car simulation the platform hands to the nearest occupant, or a script's own grant. `aiDriven` says an NPC drive task holds the lease. `epoch` is the authority epoch; `since` is when the owner last changed on the `GetGameTimer()` clock and `ageMs` how long ago that was; `reason` is why (`driverclaim`, `driverrelease`, `leaseexpired`, `playerdisconnected`, `validationfailed`, `serverrevoke`, or `none` for a platform steward grant and a car nobody has owned yet); `leaseMs` is how long the current lease still has, when there is one. There is no `Open77.vehicles.entity(id)`: the client read's `entity` FIELD and `Open77.vehicles.fromEntity` already cover `NetworkGetNetworkIdFromEntity`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `{ physicsOwner, authorityPlayerId, driver?, steward, aiDriven, epoch, since, ageMs, reason, leaseMs? }, or nil`, `reason: invalid_vehicle_id, vehicle_not_found`

```lua
-- A speed camera only fines a car somebody is actually driving
local owner = Open77.vehicles.owner(vehicleId)
if owner and owner.driver and not owner.aiDriven then
    local kph = Open77.vehicles.getSpeedKph(vehicleId)
    if kph and kph > 80 then Open77.chat.send(owner.driver, ("Flashed at %d km/h"):format(kph)) end
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## positions

```lua
Open77.vehicles.positions([bucket])
```

`NETWORK`: Uses the network backend.

Reads where the server vehicles are, as one flat array of numbers.

Requires `world.vehicles`. The light twin of `Open77.vehicles.all`: the same vehicles, in the same order (ascending Open77 ID) and with the same optional routing-bucket filter, but each vehicle is six consecutive numbers -- `id, x, y, z, yaw, bucket` -- in one table, followed by the vehicle count as a second return value. Vehicle `i` (from 1) starts at index `(i - 1) * 6 + 1`. A row of `all()` is about fifty fields and six sub-tables per car; a resource that reads its fleet's poses several times a second (`open77_traffic` does, for spacing and area counts) should read this instead and call `Open77.vehicles.get` only for the cars it acts on. The pose is the server registry's last accepted transform, the one `get` reports as `x`/`y`/`z`/`yaw`. Like `all()`, it lists every vehicle on the server, not only this resource's. Without `world.vehicles` it answers an empty table and `0`, not a refusal. `bucket` must be an integer from 0 to 4294967295; `nil` lists every bucket.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `flat array: id, x, y, z, yaw, bucket per vehicle`, `integer count of vehicles`

```lua
-- /nearcars: count the cars within 50 m of the caller, in their instance, from one flat read
RegisterCommand("nearcars", function(source)
    local me = Open77.players.get(source)
    if not me or not me.position then return end
    local flat, count = Open77.vehicles.positions(me.bucket)
    local near = 0
    for at = 0, (count - 1) * 6, 6 do
        local dx, dy = flat[at + 2] - me.position.x, flat[at + 3] - me.position.y
        if dx * dx + dy * dy <= 50 * 50 then near = near + 1 end
    end
    Open77.chat.send(source, ("%d of %d cars within 50 m"):format(near, count))
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 9723) as `LuaResourceRuntime.Bootstrap`.

## registerDamageProfile

```lua
Open77.vehicles.registerDamageProfile(record, profile)
```

`NETWORK`: Uses the network backend.

Registers resource-local named damage zones and glass parts.

Requires `world.vehicles`. Profiles are scoped to the calling server resource VM and can extend or override the default body-zone mapping for one vehicle record.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |
| `profile` | `table` | required | — |

Returns: ``true``

```lua
-- Name the body cells so later calls can say "rear" instead of listing indexes
Open77.vehicles.registerDamageProfile("Vehicle.v_standard2_archer_hella_player", {
    bodyZones = { hood = { 13, 14, 15 }, trunkLid = { 1, 2, 3 } },
    glass = { windscreen = 0, rearWindow = 1 },
})
Open77.vehicles.damageBodyZone(vehicleId, "hood", 0.5)
Open77.vehicles.breakGlass(vehicleId, "windscreen")
```

Registered in `LuaResourceRuntime.cs` (line 9206) as `LuaResourceRuntime.Bootstrap`.

## registerEngineSounds

```lua
Open77.vehicles.registerEngineSounds(catalogue)
```

`NETWORK`: Uses the network backend.

Declares the engine sound records a resource's pack adds.

Needs no permission: it changes no car. `catalogue` is `{ { id, label, class, set, index, models, maxStage? }, ... }`. `id`: lower-case letters, digits, `_ . -`, at most 32. `label`: 1-64 characters naming no make. `class`: `car` or `bike`. `set`: the vanilla sound set the records use (kept internal, never listed). `index`: the `k` of the record suffix, 1-999. `models`: `{ [baseRecord] = { stage, ... } }` (0 = factory stage), or a list of base records covering stages 0 to `maxStage` (0 when absent) -- exactly the records the pack generated, since the server never asks for a record the clients lack. Ids and indices belong to the first running resource that registers them and go when it stops. Atomic.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `catalogue` | `table` | required | — |

Returns: `true, or false`, `reason: invalid_engine_sound_catalogue, invalid_engine_sound:<id>.<field> (or #<n>.<field>), duplicate_engine_sound:<id>, engine_sound_taken:<id>, engine_sound_index_taken:<index>, engine_sound_limit, vehicles_unavailable`

```lua
-- s01 uses the vanilla set of a city compact; the pack generated its records
-- for this model at stages 0 to 3
Open77.vehicles.registerEngineSounds({
  { id = "s01", label = "Compact, city buzz", class = "car", set = "v_car_makigai_maimai", index = 1,
    models = { "Vehicle.v_standard2_makigai_maimai_player" }, maxStage = 3 },
})
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## registerPaintProfiles

```lua
Open77.vehicles.registerPaintProfiles(catalogue)
```

`NETWORK`: Uses the network backend.

Declares the colour template records this resource's TweakXL pack adds, per model.

`catalogue` is `{ [record] = { { id, label?, main?, sec?, pattern?, match?, decals? }, ... } }`. `record` is a vehicle record (`Vehicle.` + letters, digits, `_`; a stage record inherits its base's rows). `id` is the template record, `Vehicle.o77_` + letters, digits, `_`, at most 128 characters: the prefix keeps every vanilla template out of reach, and an id belongs to the first running resource that registers it. `label` (1-64 characters) and `main` / `sec` (`{r,g,b}`, `{r=,g=,b=}` or `"#rrggbb"`) are for a UI's swatches; `pattern` (default false) is `hasPattern`; `match` (appearance names) is listed as `appearances`. A vehicle kit `profiles.json` `records` table is accepted as is.

**A template the pack adds must carry at most 12 `customDecals`**: 13 or more crash every client that spawns the car (2.5, `Cyberpunk2077.exe+0x37DD77`), whatever the number of cars nearby. The server cannot read the pack, so the cap is the pack builder's job.

Merged into this resource's set: a row restating an id on the same record replaces it, others are added, and nothing is taken away until the resource stops or is reloaded: a hot reload declares the set again from scratch, so what the new generation does not register goes, and a reload that fails keeps the running set. A resource that fails to load releases whatever it registered first. Then its profiles go: a car wearing one keeps the choice (`getVisual` still answers it) but shows generic paint until the resource registers it again. **Needs no permission**: it paints no car, and applying a profile is still `setPaintProfile`. `decals` (0-12, default 0) is the template's `customDecals` count: it counts against the car's 12-decal budget with its stickers, and a livery above 12 is refused, since it would crash every client on its own. Give it for a livery with decals: left out, the livery counts as 0, `getDecalBudget` under-reports, and stickers past the real budget are accepted, then dropped by the client. Caps: 64 rows per record, 256 records, 1024 rows per resource; one call is read with a budget of 262,144 Lua values (not the 4096 of a network event), so a catalogue within these caps registers in one call. Atomic.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `catalogue` | `table` | required | — |

Returns: `true, or false`, `reason: invalid_profile_catalogue, invalid_profile_record:<record>, invalid_paint_profile:<id>.<field>, duplicate_paint_profile:<id>, profile_taken:<id>, profile_limit, vehicles_unavailable`

```lua
-- At resource start: the livery pack's records for one model
local ok, reason = Open77.vehicles.registerPaintProfiles({
    ["Vehicle.v_sport2_quadra_type66"] = {
        { id = "Vehicle.o77_type66_stripe", label = "Racing stripe", main = "#d11f2a", sec = "#f2f2f2", pattern = true },
    },
})
if not ok then print("liveries refused: " .. reason) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## registerParts

```lua
Open77.vehicles.registerParts(catalogue)
```

`NETWORK`: Uses the network backend.

Declares this resource's kit parts and rims, and how they fit each model.

Ids are namespaced by resource (`<resource>:<id>`) and vanish when the resource stops; cars wearing a vanished part republish without it and get it back when the kit is registered again. A hot reload declares the kit again from scratch: a part the new generation does not register vanishes the same way, and a reload that fails keeps the running kit whole. A resource that fails to load releases whatever it registered first. **Register at the top level of the script**: a reload republishes the cars as soon as the top level has run, so a kit registered later (a `CreateThread`, a database callback) first shows them without its parts, then with them, and a write naming one in between is refused (`unknown_part`). A part a failed reload put on a car stays chosen but dormant until the kit registers it again. **Needs no permission**: it changes no car, only what this resource's own ids mean.

Entry: `{ kind = "rim" | "part" | "toggle", label?, slot? (part: the vehicle component it attaches to), carrier? (part: the slot of the opening panel it rides on), modType?, variants = { [variant] = model }, fit = { [record] = { slot?, carrier?, offset?, rotation?, scale?, hide?, show?, wheels?, appearances? } } }`. `carrier = "trunk_a"` makes a part fitted on the body follow the trunk lid as it opens: the offsets stay those of `slot` with the lid shut, and the client moves the part as far as the carrier slot has moved (it measures the shut pose itself, and follows `slot` alone until then or on a car without that slot). A fit's `carrier = false` means none on that record; a rim or a toggle has none. A rim's `wheels = { { slot, offset?, rotation?, scale? }, ... }` (1-8 entries, unique slots) replaces its default wheel slots on that record, one prop per entry. `appearances = { ["<appearance>"] = { offset?, rotation?, scale?, wheels?, hide?, show? } }` (at most 32, never `slot` or `carrier`) fits the part per trim: a car wearing that appearance gets the record fit with those fields replaced. The look follows the car's appearance on its own: `setAppearance`, `setProperties` and `respawnAs` republish it in the same call. A model is `asset:<resource>/<model>`, a prop a pack resource registered, resolved like `Open77.props.create`; the client skips and logs a model it cannot resolve (`model_not_registered`, for instance from an archive the game did not mount at start). A part applies only to the records its `fit` lists (`part_not_for_model`). Atomic.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `catalogue` | `table` | required | — |

Returns: `true, or false`, `reason: invalid_part_catalogue, invalid_part:<id>.<field>, part_limit, vehicles_unavailable`

```lua
-- Models are props a pack resource registered with Open77.props.registerModel
Open77.vehicles.registerParts({
    wing_gt = {
        kind = "part", slot = "trunk_body", label = "GT wing",
        variants = { carbon = "asset:my_kit/wing_gt_carbon" },
        fit = { ["Vehicle.v_standard2_archer_hella_player"] = { offset = { 0, -0.2, 0.35 } } },
    },
    rim_fork = {
        kind = "rim", label = "Fork 5",
        variants = { black = "asset:my_kit/rim_fork_black" },
        fit = { ["Vehicle.v_standard2_archer_hella_player"] = {
            hide = { "wheel_01_fl_a", "wheel_01_fr_a", "wheel_01_bl_a", "wheel_01_br_a" } } },
    },
    no_antenna = { kind = "toggle", fit = { ["Vehicle.v_standard2_archer_hella_player"] = {
        hide = { "attachment_antenna_1_n_roof_radar_cables" } } } },
})
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## registerStickers

```lua
Open77.vehicles.registerStickers(catalogue)
```

`NETWORK`: Uses the network backend.

Declares the stickers a resource's pack adds.

Needs no permission: it changes no car. `catalogue` is `{ { id, label, group, records | parts, model?, appearances?, decals?, record? }, ... }`. `records`: 1 to 12 distinct `VehicleDecalAttachment` records of the pack (`Vehicle.o77_...`); `parts`, the vehicle kit's form, gives the same records as `{ record, component, decal }` rows (with both, they must agree). `model`: a record or an appearance name, or a list of them (records start with `Vehicle.`; a stage or sound record fits its base). `decals`, when given, must equal the number of records. `record`: the pack's own colour template listing the decals (kept, never published). Police (NCPD included), sheriff, border patrol, Arasaka and Delamain sets are refused by name, in the id, the label or the group (`sticker_excluded`). Ids belong to the first running resource that registers them and go when it stops. Atomic.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `catalogue` | `table` | required | — |

Returns: `true, or false`, `reason: invalid_sticker_catalogue, invalid_sticker:<id>.<field> (or #<n>.<field>), duplicate_sticker:<id>, sticker_taken:<id>, sticker_excluded:<id>, sticker_limit, vehicles_unavailable`

```lua
Open77.vehicles.registerStickers({
  { id = "st.hella.valentinos_01", label = "Valentinos heart", group = "Valentinos",
    records = { "Vehicle.o77_st_hella_valentinos_01" }, model = "Vehicle.v_standard2_archer_hella_player" },
})
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.vehicles.remove(id)
```

`NETWORK`: Uses the network backend.

Removes a server-authoritative network vehicle.

Requires `world.vehicles`. Any authorized resource may remove any canonical vehicle, regardless of which resource created it. Removal clears occupancy and authority before notifying streamed clients.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean success`

```lua
-- /dv: delete the vehicle the caller is sitting in
RegisterCommand("dv", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return Open77.chat.send(source, "You are not in a vehicle.") end
    Open77.vehicles.remove(seat.vehicleId)
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 9220) as `LuaResourceRuntime.Bootstrap`.

## removePlayerFromVehicle

```lua
Open77.vehicles.removePlayerFromVehicle(playerId, [vehicleId])
```

`NETWORK`: Uses the network backend.

Alias for forced authoritative vehicle exit.

Same contract as `forcePlayerOutOfVehicle`, including the optional expected vehicle ID, exit-lock override, client confirmation, and timeout cleanup.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Car-jacking mini-game: the victim is thrown out when the thief wins
Open77.vehicles.removePlayerFromVehicle(victimId, vehicleId)
Open77.vehicles.warpPlayerIntoVehicle(thiefId, vehicleId, Open77.vehicles.seats.driver)
```

Registered in `LuaResourceRuntime.cs` (line 9078) as `LuaResourceRuntime.Bootstrap`.

## repair

```lua
Open77.vehicles.repair(id, [scope])
```

`NETWORK`: Uses the network backend.

Repairs a selected vehicle damage scope.

Requires `world.vehicles`. Supported scopes are `glass`, `body`, `lights`, `tires`, `visual`, `mechanical`, and `full`. The default is `full`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `scope` | `string` | optional | `full` |

Returns: `boolean success`

```lua
-- The body shop fixes the paintwork only; the mechanic does the rest
RegisterCommand("bodyshop", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if seat then Open77.vehicles.repair(seat.vehicleId, "visual") end
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 9205) as `LuaResourceRuntime.Bootstrap`.

## repairAllGlass

```lua
Open77.vehicles.repairAllGlass(id)
```

`NETWORK`: Uses the network backend.

Repairs all vehicle glass.

Requires `world.vehicles`. Clears the complete canonical broken-glass mask.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean success`

```lua
-- The glass shop fixes every window for 200 eddies
if takeMoney(source, 200) then
    Open77.vehicles.repairAllGlass(vehicleId)
end
```

Registered in `LuaResourceRuntime.cs` (line 9147) as `LuaResourceRuntime.Bootstrap`.

## repairAllLights

```lua
Open77.vehicles.repairAllLights(id)
```

`NETWORK`: Uses the network backend.

Repairs every vehicle light.

Requires `world.vehicles`. Clears the complete canonical broken-light mask.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Part of the "full service" menu at the mechanic
Open77.vehicles.repairAllLights(vehicleId)
```

Registered in `LuaResourceRuntime.cs` (line 9152) as `LuaResourceRuntime.Bootstrap`.

## repairAllTires

```lua
Open77.vehicles.repairAllTires(id)
```

`NETWORK`: Uses the network backend.

Repairs all four vehicle tires.

Requires `world.vehicles`. Clears the complete canonical broken-tire mask.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Tire shop: four new tires
Open77.vehicles.repairAllTires(vehicleId)
Open77.chat.send(source, "New tires fitted.")
```

Registered in `LuaResourceRuntime.cs` (line 9157) as `LuaResourceRuntime.Bootstrap`.

## repairBodyCell

```lua
Open77.vehicles.repairBodyCell(id, cell)
```

`NETWORK`: Uses the network backend.

Repairs one body cell.

Requires `world.vehicles`. Sets the selected 1-based body cell to zero damage and replicates the repaired state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `cell` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Repair the single cell the player pointed at with the body-shop tool
Open77.vehicles.repairBodyCell(vehicleId, cellIndex)
```

Registered in `LuaResourceRuntime.cs` (line 9132) as `LuaResourceRuntime.Bootstrap`.

## repairBodyZone

```lua
Open77.vehicles.repairBodyZone(id, zone)
```

`NETWORK`: Uses the network backend.

Repairs every body cell in a zone.

Requires `world.vehicles`. Clears all cells selected by a registered zone name or explicit index array.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `zone` | `string|table` | required | — |

Returns: `boolean success`

```lua
-- The body shop repairs one side of the car at a time
Open77.vehicles.repairBodyZone(vehicleId, "left")
```

Registered in `LuaResourceRuntime.cs` (line 9135) as `LuaResourceRuntime.Bootstrap`.

## repairGlass

```lua
Open77.vehicles.repairGlass(id, glass)
```

`NETWORK`: Uses the network backend.

Repairs one vehicle glass element.

Requires `world.vehicles`. Convenience wrapper over `setGlassBroken(id, glass, false)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `glass` | `integer|string` | required | — |

Returns: `boolean success`

```lua
-- Replace just the windscreen
Open77.vehicles.repairGlass(vehicleId, 0)
```

Registered in `LuaResourceRuntime.cs` (line 9142) as `LuaResourceRuntime.Bootstrap`.

## repairLight

```lua
Open77.vehicles.repairLight(id, index)
```

`NETWORK`: Uses the network backend.

Repairs one vehicle light.

Requires `world.vehicles`. Convenience wrapper over `setLightBroken(id, index, false)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `index` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Replace the left headlight
Open77.vehicles.repairLight(vehicleId, 0)
```

Registered in `LuaResourceRuntime.cs` (line 9151) as `LuaResourceRuntime.Bootstrap`.

## repairTire

```lua
Open77.vehicles.repairTire(id, index)
```

`NETWORK`: Uses the network backend.

Repairs one vehicle tire.

Requires `world.vehicles`. Convenience wrapper over `setTireBroken(id, index, false)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `index` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Change the front-left tire
Open77.vehicles.repairTire(vehicleId, 0)
```

Registered in `LuaResourceRuntime.cs` (line 9156) as `LuaResourceRuntime.Bootstrap`.

## requestAuthority

```lua
Open77.vehicles.requestAuthority(id, playerId)
```

`NETWORK`: Uses the network backend.

Asks that one client simulate a vehicle. A request, never an order.

Requires `world.vehicles`. FiveM's `NetworkRequestControlOfEntity`, with the difference that Open77 leases authority from the server and the lease already moves on its own, so a script's request goes through the election's own eligibility and is refused BY NAME when it fails rather than forced: `driven` when somebody else is the seated controller (a driven car is never taken from its driver -- the rule that makes this a request), `ai_driven` when an NPC drive task holds the lease, `not_streamed` when the player has no body for the car yet, `too_far` beyond the 350 m stream-in radius, `wrong_bucket`, `player_unavailable` when the player has no fresh position (the same two-second rule every claim applies). The requested player being the driver already is the one case that is not a steal, and it is granted; asking for a car the player already owns answers the read with `already_owner` and re-grants nothing. On success the owner table (see `owner`) after the grant. A grant is a lease like any other: the client keeps it by reporting, which its parked-pose loop does twice a second once the car has been attached for five seconds, so a grant to a client that has only just streamed the car in can lapse two seconds later, and `owner` then says so.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `playerId` | `player id` | required | — |

Returns: `the owner table after the grant, or nil`, `reason: driven, ai_driven, not_streamed, too_far, wrong_bucket, player_unavailable, invalid_player_id, vehicle_not_found`

```lua
-- Hand a parked car to the player who just bought it, so their client simulates it
-- while they walk around it; refused by name if somebody is sitting in it
local read, reason = Open77.vehicles.requestAuthority(vehicleId, buyerId)
if read == nil then
    Open77.chat.send(buyerId, "Not yet: " .. reason)
else
    print(("vehicle %d now simulated by %d (steward=%s)"):format(vehicleId, read.physicsOwner, tostring(read.steward)))
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## resetPaint

```lua
Open77.vehicles.resetPaint(id)
```

`NETWORK`: Uses the network backend.

Restores a vehicle's authored paint.

Requires `world.vehicles`. Clears the canonical custom-paint flag without treating stored RGB bytes as a sentinel, then reliably publishes the reset to all current and future viewers.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean success`

```lua
-- The player sold the car: back to factory paint for the next owner
Open77.vehicles.resetPaint(vehicleId)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9122) as `LuaResourceRuntime.Bootstrap`.

## respawnAs

```lua
Open77.vehicles.respawnAs(id, record)
```

`NETWORK`: Uses the network backend.

Rebuilds a car on another record at the same place, under the same id.

Requires `world.vehicles`. Swaps the car's record -- a performance stage `Vehicle.<base>_o77sN`, or any other -- and keeps paint, appearance, lights hue, health, the visual state and every state bag key. The car must be empty. A record of the same model without a sound suffix keeps the car's engine sound (`Vehicle.x_o77s2` on a car with sound 3 respawns it on `Vehicle.x_o77s2a3`): `engine_sound_not_for_stage` when the sound pack generated no record for that stage, `engine_sound_unregistered` while the sound pack is stopped. `setEngineSound(id, nil)` takes the sound off first. Answers the id, which never changes, so every reference a resource holds stays valid.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `record` | `string` | required | — |

Returns: `the same id, or nil`, `reason: vehicle_occupied, engine_sound_not_for_stage, engine_sound_unregistered, invalid_record, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- Apply a performance stage from the tuning shop
local id, reason = Open77.vehicles.respawnAs(vehicleId, "Vehicle.v_sport2_quadra_type66_o77s2")
if not id then print("stage refused: " .. reason) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## seatFree

```lua
Open77.vehicles.seatFree(id, seat)
```

`NETWORK`: Uses the network backend.

Whether one seat of a vehicle is free.

Requires `world.vehicles`. A seat held by an occupant whose entry animation is still running counts as TAKEN: the seat is reserved and a second warp into it is refused, so reporting it free would be a lie the caller then trips over. Seats accept `-1..2`, the `Open77.vehicles.seats` constants, aliases like `driver`, and canonical `seat_*` names.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `seat` | `integer|string` | required | — |

Returns: `true or false for a known vehicle and seat; nil when the call failed`, `reason: invalid_seat, vehicle_not_found (only with nil)`

```lua
-- Let a passenger in only if the seat is really empty
if Open77.vehicles.seatFree(vehicleId, "frontPassenger") then
    Open77.vehicles.warpPlayerIntoVehicle(playerId, vehicleId, "frontPassenger")
else
    Open77.chat.send(playerId, "That seat is taken.")
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## seatName

```lua
Open77.vehicles.seatName(seat)
```

`NETWORK`: Uses the network backend.

The canonical spelling of a seat name or index.

No permission. Turns `-1`, `"driver"`, `"frontLeft"` or `"seat_front_left"` into `"seat_front_left"`, which is the spelling every occupant record and every seat event uses. `nil` for anything that names no seat.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `seat` | `integer|string` | required | — |

Returns: `string, or nil`

```lua
-- Whatever the player typed, store the canonical seat name
RegisterCommand("sit", function(source, args)
    local seat = Open77.vehicles.seatName(args[1] or "driver")
    if not seat then return Open77.chat.send(source, "Unknown seat.") end
    print(seat) -- seat_front_left, seat_front_right, ...
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## seats

```lua
Open77.vehicles.seats()
```

`NETWORK`: Uses the network backend.

The four seat numbers; every seat native also takes the canonical names and their aliases.

A public constant table, not a function: `driver = -1`, `frontPassenger = 0`, `rearLeft = 1`, `rearRight = 2` -- the FiveM numbering. The **canonical** seat names, the only spelling the runtime ever reports (`getPlayerSeat(...).seat`, `occupants[]`, `freeSeats`, `occupantInSeat`, `onPlayerEnteredVehicle`), are `seat_front_left`, `seat_front_right`, `seat_back_left` and `seat_back_right`. Wherever a seat is *accepted* (`warpPlayerIntoVehicle`, `taskPlayerEnter`, `occupantInSeat`, `seatFree`, `seatName`) the server takes any of: these numbers; the canonical names; or the aliases `driver`/`frontLeft`/`front_left`, `frontPassenger`/`frontRight`/`front_right`, `rearLeft`/`backLeft`/`back_left`, `rearRight`/`backRight`/`back_right`, matched case-insensitively. `passenger`, `rear_left` and index 3 are `invalid_seat`. `Open77.vehicles.seatName(seat)` normalises any accepted form to the canonical one. The client host accepts the same aliases case-sensitively and without the snake_case forms.

Returns: `table { driver = -1, frontPassenger = 0, rearLeft = 1, rearRight = 2 }`

```lua
Open77.vehicles.warpPlayerIntoVehicle(playerId, vehicleId, Open77.vehicles.seats.driver)
-- the same, by alias
Open77.vehicles.warpPlayerIntoVehicle(playerId, vehicleId, "driver")
-- what the ledger reports afterwards is the canonical name
assert(Open77.vehicles.getPlayerSeat(playerId).seat == "seat_front_left")
```

Registered in `LuaResourceRuntime.cs` (line 9055) as `LuaResourceRuntime.Bootstrap`.

## setAdoptPolicy

```lua
Open77.vehicles.setAdoptPolicy(policy, [options])
```

`NETWORK`: Uses the network backend.

Opens or closes one routing bucket to vehicle adoption.

Requires `world.vehicles` and `world.vehicles.adopt`. `"none"` is the default and every bucket starts there: nothing is adoptable until a resource says so. `"driver"` accepts only a car the proposing player is standing at, within six metres — the car-theft shape, where you make canonical the car you are about to take and nothing else. `"any"` widens that to sixty metres; it does not remove the check. `options.bucket` selects the routing bucket (default `0`). The policy is server-side only: a client never learns it, and a proposal into a closed bucket is simply refused.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `policy` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Street cars may be stolen in the free-roam bucket, but only the one you stand at
Open77.vehicles.setAdoptPolicy("driver", { bucket = 0 })
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setAppearance

```lua
Open77.vehicles.setAppearance(id, appearance)
```

`NETWORK`: Uses the network backend.

Swaps the appearance variant on a live car, on every viewer.

Requires `world.vehicles`, and a change requires `world.vehicles.appearance` too. The second string exists because this is the one vehicle write that changes what the world LOOKS like rather than what a car does: it reaches a native entity-appearance reschedule on every viewer at once, for every car a resource can name, and a resource that wants to repaint the city should have to say so -- the same argument that gave adoption its own string. **It gates a CHANGE, never a restatement**: writing the variant a car already wears is accepted without it, which is what makes `setProperties(id, getProperties(id))` work for a resource that never changes a look.

`nil` or `""` restores the record's default variant. The name is not validated here, because only the client holds the entity template: a variant the model does not define is a no-op on each projection and the canonical value stays as written. `Open77.data.vehicle(record)` looks a record up but does not enumerate variants -- the shipped 2.31 catalogue does not carry them.

**No intent window, and that is structural rather than lucky.** The physics owner's durable report has no appearance field and no colour field of any kind, so the twice-a-second merge that used to undo server writes cannot reach this one. On the client the variant is applied through `entEntity::ScheduleAppearanceChange` and held on the replica until the projection attaches -- measured at up to about 3.7 s after the create -- then re-applied if the entity is torn down and streamed back.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `appearance` | `string|nil` | required | — |

Returns: `true, or false`, `reason: invalid_appearance, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles, permission_denied:world.vehicles.appearance`

```lua
-- Livery shop: switch the caller's car to a different factory trim
RegisterCommand("livery", function(source, args)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat or not args[1] then return end
    local ok, reason = Open77.vehicles.setAppearance(seat.vehicleId, args[1])
    if not ok then Open77.chat.send(source, "Livery refused: " .. reason) end
end, false)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setBodyCell

```lua
Open77.vehicles.setBodyCell(id, cell, value)
```

`NETWORK`: Uses the network backend.

Sets one body-damage cell.

Requires `world.vehicles`. Lua body-cell indexes are 1..30 and the value is clamped or validated in the normalized 0..1 range.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `cell` | `integer` | required | — |
| `value` | `number` | required | — |

Returns: `boolean success`

```lua
-- Restore one body cell from a stored damage record
Open77.vehicles.setBodyCell(vehicleId, 7, 0.4)
```

Registered in `LuaResourceRuntime.cs` (line 9127) as `LuaResourceRuntime.Bootstrap`.

## setBodyDamage

```lua
Open77.vehicles.setBodyDamage(id, values)
```

`NETWORK`: Uses the network backend.

Replaces the complete body-damage grid.

Requires `world.vehicles`. The table must contain exactly 30 normalized finite values in the range 0..1.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `values` | `table` | required | — |

Returns: `boolean success`

```lua
-- A freshly spawned car with a fully repaired body
local pristine = {}
for i = 1, 30 do pristine[i] = 0.0 end
Open77.vehicles.setBodyDamage(vehicleId, pristine)
```

Registered in `LuaResourceRuntime.cs` (line 9126) as `LuaResourceRuntime.Bootstrap`.

## setBodyZone

```lua
Open77.vehicles.setBodyZone(id, zone, value)
```

`NETWORK`: Uses the network backend.

Sets all body cells in a zone.

Requires `world.vehicles`. The zone may be a registered zone name or an explicit array of cell indexes. Every selected cell receives the normalized value.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `zone` | `string|table` | required | — |
| `value` | `number` | required | — |

Returns: `boolean success`

```lua
-- Crash scene prop: a car with a crushed front
Open77.vehicles.setBodyZone(vehicleId, "front", 0.9)
```

Registered in `LuaResourceRuntime.cs` (line 9133) as `LuaResourceRuntime.Bootstrap`.

## setDamage

```lua
Open77.vehicles.setDamage(id, damage)
```

`NETWORK`: Uses the network backend.

Replaces multiple canonical damage components at once.

Requires `world.vehicles`. Accepts a combined damage table and validates body cells and bit masks before publishing one authoritative update.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `damage` | `table` | required | — |

Returns: `boolean success`

```lua
-- Restore several damage channels in one replicated update
Open77.vehicles.setDamage(vehicleId, {
    tires = 0,           -- no broken tires
    glass = 0,           -- no broken glass
    health = 0.75,
})
```

Registered in `LuaResourceRuntime.cs` (line 9125) as `LuaResourceRuntime.Bootstrap`.

## setDespawnWhenUnobserved

```lua
Open77.vehicles.setDespawnWhenUnobserved(id, despawn)
```

`NETWORK`: Uses the network backend.

Removes a vehicle once nobody has had it streamed for thirty seconds.

Requires `world.vehicles`. The window is not zero because interest is recomputed against the last received player snapshot, and a loading screen or a bucket change briefly empties a viewer set that is about to refill; thirty seconds rides that out. The clock runs for a vehicle nobody has **ever** seen too, which is what a garage wants -- a car spawned for a player who never turns up still goes. A viewer coming back resets the clock rather than pausing it, and turning the flag off forgets it entirely so that turning it back on later does not inherit a deadline from a window nobody was counting. A persistent vehicle is exempt. Read back as `Open77.vehicles.get(id).despawnWhenUnobserved`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `despawn` | `boolean` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- Rental cars vanish once everybody has driven away from them
local id = Open77.vehicles.create({ record = "Vehicle.v_standard2_archer_hella_player", position = spot })
Open77.vehicles.setDespawnWhenUnobserved(id, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setDetachedPartMask

```lua
Open77.vehicles.setDetachedPartMask(id, mask)
```

`NETWORK`: Uses the network backend.

Adds a canonical detached-part mask.

Requires `world.vehicles`. The 16-bit mask is monotonic on a live projection. Removing a previously detached bit is rejected because REDengine 2.31 has no validated safe reattachment primitive.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `mask` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Reproduce a stored wreck: bits are one-way, so apply the whole mask at once
Open77.vehicles.setDetachedPartMask(vehicleId, storedWreck.detachedParts)
```

Registered in `LuaResourceRuntime.cs` (line 9158) as `LuaResourceRuntime.Bootstrap`.

## setDoorMask

```lua
Open77.vehicles.setDoorMask(id, mask)
```

`NETWORK`: Uses the network backend.

Replaces the vehicle opening mask.

Requires `world.vehicles`. The six-bit mask covers four doors, trunk, and hood. Valid values are 0..63.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `mask` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Close everything at once (doors, trunk and hood)
Open77.vehicles.setDoorMask(vehicleId, 0)
```

Registered in `LuaResourceRuntime.cs` (line 9180) as `LuaResourceRuntime.Bootstrap`.

## setDoorOpen

```lua
Open77.vehicles.setDoorOpen(id, door, opened)
```

`NETWORK`: Uses the network backend.

Sets one door, trunk, or hood opening state.

Requires `world.vehicles`. `door` is an index 0..5 or one of the constants in `Open77.vehicles.doors`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `door` | `integer` | required | — |
| `opened` | `boolean` | required | — |

Returns: `boolean success`

```lua
-- Pop the hood for the mechanic
Open77.vehicles.setDoorOpen(vehicleId, Open77.vehicles.doors.hood, true)
```

Registered in `LuaResourceRuntime.cs` (line 9185) as `LuaResourceRuntime.Bootstrap`.

## setDrivable

```lua
Open77.vehicles.setDrivable(id, drivable)
```

`NETWORK`: Uses the network backend.

Puts a vehicle in or out of service.

Requires `world.vehicles`. Out of service cuts the engine in the SAME canonical revision and locks the simulated driver's throttle and brake on every projection, so the car cannot be pushed along by its own inputs; a car whose throttle is dead while its engine idles is a bug report waiting to happen. Putting it back in service does NOT restart the engine -- that is `setEngine`. This is the `SetVehicleUndriveable` of FiveM, and like the entry lock it is server-authored: no owner report can clear it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `drivable` | `boolean` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found`

```lua
-- Wheel clamp: the car stays intact but will not move
RegisterCommand("clamp", function(source, args)
    local vehicleId = tonumber(args[1])
    if not vehicleId then return end
    Open77.vehicles.setDrivable(vehicleId, false)
    Open77.chat.send(source, "Clamped.")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setEngine

```lua
Open77.vehicles.setEngine(id, on)
```

`NETWORK`: Uses the network backend.

Starts or stops a vehicle's engine.

Requires `world.vehicles`. Updates ignition while preserving unrelated flags. Stopping ignition also cancels engineHold. The server retains a ten-second hold against conflicting owner reports, releasing each held bit when the owner reports the requested value. Separately, VehicleState carries an explicit electrical intent revision, mask and value: even a same-value setter is a new command, while ordinary owner-report echoes are observations, not commands. Owners initialize canonical control once per native/authority incarnation; subsequent explicit intents accumulate by mask and failed native calls retry. Local engine/light readbacks update the owner baseline only outside buffered or unapplied intent. Revoke/park authors explicit electrical OFF unless engineHold is enabled; a driver exit can retain coast/contact authority and is not itself always a revoke.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `on` | `boolean` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- Remote engine kill from the dispatcher
RegisterCommand("killengine", function(source, args)
    local vehicleId = tonumber(args[1])
    if not vehicleId then return end
    local ok, reason = Open77.vehicles.setEngine(vehicleId, false)
    if not ok then Open77.chat.send(source, reason) end
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setEngineHealth

```lua
Open77.vehicles.setEngineHealth(id, health)
```

`NETWORK`: Uses the network backend.

Sets vehicle health under its FiveM name.

Requires `world.vehicles`. **Cyberpunk models one vehicle health pool.** There is no separate engine pool to damage, so this is `Open77.vehicles.setHealth` under the name a FiveM port reaches for, and at 0 the car is a wreck rather than a running car with a dead engine. If you want the car to stay whole but refuse to move, that is `setDrivable(id, false)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `health` | `number 0..1` | required | — |

Returns: `true, or false`, `reason`

```lua
-- FiveM port: remember the scale is 0..1 here, not 0..1000
Open77.vehicles.setEngineHealth(vehicleId, 0.5)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setEngineHold

```lua
Open77.vehicles.setEngineHold(id, hold)
```

`SERVER`: Available in server resources only.

Keep an empty vehicle engine running on every client.

Requires matching Stable 147 client/server (protocol 1.48). Enabling starts ignition. The hold survives driver exit, parking, owner changes and stream-in. Clearing it keeps current ignition; setEngine(id, false), zero health, destruction or undriveable state cancel it. No restart timer is needed. getProperties/setProperties include engineHold. See [Vehicles](https://open2077.net/docs/api/server/vehicles.md#keep-an-empty-vehicle-running).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `hold` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason on failure`

```lua
local ok, reason = Open77.vehicles.setEngineHold(vehicleId, true)
```

[Keep an empty vehicle running](../../01%20Documentation/vehicles.md#keep-an-empty-vehicle-running)

Registered in `LuaResourceRuntime.Vehicles.cs` (line 806) as `LuaResourceRuntime.Bootstrap`.

## setEngineSound

```lua
Open77.vehicles.setEngineSound(id, soundId)
```

`NETWORK`: Uses the network backend.

Gives a car another engine sound set, horn included.

Requires `world.vehicles` only: like a stage, a sound is a record (`Vehicle.<base>_o77a<k>`, or `_o77s<n>a<k>` on a stage) that the pack adds, and the car is respawned on it in place under the same id, keeping paint, appearance, health, the visual state and every state bag key. `soundId` is an id `listEngineSounds` lists, or `nil` for the factory sound of the current stage. Restating the current sound changes nothing. The car must be empty (`vehicle_occupied`). Every client, the driver included, hears what the record says.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `soundId` | `string|nil` | required | — |

Returns: `true, or false`, `reason: invalid_engine_sound, unknown_engine_sound, engine_sound_not_for_class, engine_sound_not_for_model, vehicle_occupied, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
local ok, reason = Open77.vehicles.setEngineSound(vehicleId, "s01")
if not ok then print("sound refused: " .. reason) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setFrozen

```lua
Open77.vehicles.setFrozen(id, frozen)
```

`NETWORK`: Uses the network backend.

Pins a vehicle where it stands, or lets it go.

Requires `world.vehicles`. The server half of FiveM's `FreezeEntityPosition` for a vehicle, and only that half: the control lock is `setUndriveable`, and the two compose (an impound wants both, a starting grid wants the freeze alone with the engine running). Three things happen in ONE canonical revision: the `frozen` bit goes up (`Open77.vehicles.flags.frozen`, bit 11), the motion facts are pinned (`speed`, `velocity`, `angularVelocity`, `onGround`, `reversing` read as a car at rest), and the physics owner's transform reports are discarded while the bit is up -- discarded, not punished: the lease is renewed and the tick consumed as an accepted report would be, so nothing flaps and nothing is logged. Every projection maps the bit to the native physics mask, so inertia, gravity and impulses stop too; a driver pressing the throttle revs an engine that goes nowhere. Nothing electrical changes. Like `undriveable` the bit is server-authored, absent from every owner report and preserved across them. The pinned transform is wherever the last ACCEPTED report put the car: frozen at speed, the owner's screen stops one latency past the pin and the two agree again after the release. Widening the accepted flag mask is a wire change -- server and client must be built together.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `frozen` | `boolean` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found`

```lua
-- Impound: nobody drives it, nobody pushes it, and it survives the lot emptying
Open77.vehicles.setUndriveable(vehicleId, true)
Open77.vehicles.setFrozen(vehicleId, true)
Open77.vehicles.setPersistent(vehicleId, true)

-- Release, inputs first, then the body
Open77.vehicles.setFrozen(vehicleId, false)
Open77.vehicles.setDrivable(vehicleId, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setGlassBroken

```lua
Open77.vehicles.setGlassBroken(id, glass, broken)
```

`NETWORK`: Uses the network backend.

Sets or clears one broken-glass bit.

Requires `world.vehicles`. `glass` may be a numeric 0..31 index or a name supplied by the resource's registered damage profile.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `glass` | `integer|string` | required | — |
| `broken` | `boolean` | required | — |

Returns: `boolean success`

```lua
-- The thief smashes the driver's window to get in
Open77.vehicles.setGlassBroken(vehicleId, 1, true)
```

Registered in `LuaResourceRuntime.cs` (line 9137) as `LuaResourceRuntime.Bootstrap`.

## setGlassMask

```lua
Open77.vehicles.setGlassMask(id, mask)
```

`NETWORK`: Uses the network backend.

Replaces the broken-glass bit mask.

Requires `world.vehicles`. The 32-bit mask describes destructible glass records, not the four reversible side-window opening states.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `mask` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Restore a stored broken-glass pattern on respawn
Open77.vehicles.setGlassMask(vehicleId, stored.glass)
```

Registered in `LuaResourceRuntime.cs` (line 9136) as `LuaResourceRuntime.Bootstrap`.

## setHeadlightColor

```lua
Open77.vehicles.setHeadlightColor(id, color)
```

`NETWORK`: Uses the network backend.

The same tint, from an RGB colour.

Requires `world.vehicles`. Accepts the same colour forms as `setPaint` -- `"#RRGGBB"`, `{ r, g, b }` or a positional table -- converts it to a hue and applies that. `nil` clears the tint.

**Only the hue survives.** The engine stores one float, so `#800000` and `#FF0000` light the car identically, and a grey has no hue at all and reads as red. The function exists because a porting creator has a colour in hand and not a hue; `setLightsHue` is the honest form. `setHeadlightColour` is the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `color` | `string|table|nil` | required | — |

Returns: `true, or false`, `reason: invalid_argument, invalid_lights_hue, vehicle_not_found, permission_denied:world.vehicles`

```lua
-- The tuning shop sells a cyan headlight tint
RegisterCommand("lights", function(source, args)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if seat then Open77.vehicles.setHeadlightColor(seat.vehicleId, args[1] or "#00D8FF") end
end, false)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setHeadlightColour

```lua
Open77.vehicles.setHeadlightColour(id, color)
```

`NETWORK`: Uses the network backend.

Alias for `setHeadlightColor`.

Identical contract, in the other spelling.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `color` | `string|table|nil` | required | — |

Returns: `true, or false`, `reason`

```lua
-- British spelling, same tint
Open77.vehicles.setHeadlightColour(vehicleId, "#FF00AA")
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setHealth

```lua
Open77.vehicles.setHealth(id, health)
```

`NETWORK`: Uses the network backend.

Sets a vehicle's health, 0 to 1.

Requires `world.vehicles`. The same normalized pool `get(id).health` reads and `setDamage{ health = }` writes, under a name that does one thing. Out-of-range values are refused rather than clamped, so a caller passing FiveM's 0..1000 scale finds out.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `health` | `number 0..1` | required | — |

Returns: `true, or false`, `reason: invalid_argument, invalid_health, vehicle_not_found`

```lua
-- /fix restores the caller's car completely
RegisterCommand("fix", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return end
    Open77.vehicles.setHealth(seat.vehicleId, 1.0)
    Open77.vehicles.repair(seat.vehicleId, "full")
end, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setHighBeams

```lua
Open77.vehicles.setHighBeams(id, on)
```

`NETWORK`: Uses the network backend.

Raises or drops the high beams without going dark.

Requires `world.vehicles`. Turning them off leaves the side lights on if they were on, rather than switching the car dark -- which is what a naive `setLights("on"/"off")` pair gets wrong. Turning them on implies side lights, because the native mode cannot express one without the other.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `on` | `boolean` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found`

```lua
-- Flash the high beams at the car in front
CreateThread(function()
    Open77.vehicles.setHighBeams(vehicleId, true)
    Wait(400)
    Open77.vehicles.setHighBeams(vehicleId, false) -- side lights stay on
end)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setLaunchVelocity

```lua
Open77.vehicles.setLaunchVelocity(id, velocity)
```

`NETWORK`: Uses the network backend.

Starts a car nobody has simulated yet moving.

Requires `world.vehicles`. Sets the canonical linear velocity (m/s, world axes) of a car that has never had a physics owner, so the create every projection receives carries it and the first client to simulate the car writes it into the chassis when it takes over -- the same body write the client uses to keep a car's momentum across an ownership change. The drivetrain `speed` reading is not touched. Refused once any client has simulated the car, even after its ownership was revoked (`vehicle_simulated`; its velocity is then its simulator's report). Viewers that already stream the car receive the velocity at once. A frozen car is refused (`vehicle_frozen`; it is pinned at rest, so thaw it first). Each component is bounded to ±250. A client that only takes the car over more than 6 s after receiving its create starts it from rest.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `velocity` | `{ x, y, z? }` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found, vehicle_frozen, vehicle_simulated`

```lua
-- Ambient traffic: the car joins its lane already rolling at 10 m/s
local id = assert(Open77.vehicles.create({ record = record, position = point, yaw = yaw }))
local forward = { x = -math.sin(math.rad(yaw)), y = math.cos(math.rad(yaw)) }
Open77.vehicles.setLaunchVelocity(id, { x = forward.x * 10, y = forward.y * 10 })
```

Registered in `LuaResourceRuntime.cs` (line 9035) as `LuaResourceRuntime.Bootstrap`.

## setLightBroken

```lua
Open77.vehicles.setLightBroken(id, index, broken)
```

`NETWORK`: Uses the network backend.

Sets or clears one broken-light bit.

Requires `world.vehicles`. The light index must be in 0..31.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `index` | `integer` | required | — |
| `broken` | `boolean` | required | — |

Returns: `boolean success`

```lua
-- Repair the right headlight only
Open77.vehicles.setLightBroken(vehicleId, 1, false)
```

Registered in `LuaResourceRuntime.cs` (line 9149) as `LuaResourceRuntime.Bootstrap`.

## setLightColors

```lua
Open77.vehicles.setLightColors(id, colors)
```

`NETWORK`: Uses the network backend.

Colours a car's brake, reverse, blinker and interior lights.

Requires `world.vehicles`; a change also requires `world.vehicles.appearance`. `colors` is `{ brake?, reverse?, blinkers?, interior? }`, each a colour (`{r,g,b}`, `{r=,g=,b=}` or `"#rrggbb"`); the table is the whole set, so a group it does not name goes back to its factory colour, and `nil` or `{}` gives every group its factory colour back. Every client applies it through the car's light controller, again after every rebuild, stream-in and paint change. Headlights are not a group: `setPaint`'s lights hue keeps them. Red or blue blinkers and blue brake lights are refused: they read as an emergency vehicle.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `colors` | `table|nil` | required | — |

Returns: `true, or false`, `reason: invalid_light_colors, invalid_light_colors:<group>, emergency_look:blinkers, emergency_look:brake, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
Open77.vehicles.setLightColors(vehicleId, { brake = "#ff3300", interior = { 120, 0, 255 } })
-- factory lights again
Open77.vehicles.setLightColors(vehicleId, nil)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setLightMask

```lua
Open77.vehicles.setLightMask(id, mask)
```

`NETWORK`: Uses the network backend.

Replaces the broken-light bit mask.

Requires `world.vehicles`. Replaces the complete 32-bit canonical broken-light mask.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `mask` | `integer` | required | — |

Returns: `boolean success`

```lua
-- Restore the stored broken-light pattern
Open77.vehicles.setLightMask(vehicleId, stored.lights)
```

Registered in `LuaResourceRuntime.cs` (line 9148) as `LuaResourceRuntime.Bootstrap`.

## setLights

```lua
Open77.vehicles.setLights(id, mode)
```

`NETWORK`: Uses the network backend.

Sets a vehicle's headlight mode.

Requires `world.vehicles`. One call for the whole mode, because `lightsOn` and `highBeams` are NOT independent on this build: the client maps the pair to a single native LightMode and 'high beams with no side lights' is not a state REDengine can express. Accepts `"off"`, `"on"`, `"high"` (aliases `highBeams`, `high_beams`, `full`) or a boolean. Carries the same explicit revision/mask/value command and separate server ten-second hold as `setEngine`, including same-value commands and native-call retries. The Stable 147 client fixes application to the driver and late-joining observers. The setter acknowledges canonical state; native application is asynchronous. It does not start ignition.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `mode` | `string "off"|"on"|"high", or boolean` | required | — |

Returns: `true, or false`, `reason: invalid_light_mode, vehicle_not_found`

```lua
-- Street lights on: every parked car turns its lights on at 20:00
AddEventHandler("onEnvironmentChanged", function(env)
    if env.hour ~= 20 then return end
    for _, entry in ipairs(Open77.vehicles.all()) do
        Open77.vehicles.setLights(entry.id, "on")
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setLightsHue

```lua
Open77.vehicles.setLightsHue(id, hue)
```

`NETWORK`: Uses the network backend.

Tints the car's own lights.

Requires `world.vehicles`. `hue` is `0..1` -- the H of the HSB triple the engine's visual template carries -- and `nil` hands the lights back the record's colour.

**This is the closest build 2.31 has to `SetVehicleNeonLights` or `SetVehicleHeadlightsColour`, and it is not the same thing.** There is no underglow and no xenon palette in this game; what exists is the vehicle's own emissive colour, continuous rather than indexed, in the same `GenericTemplatePersistentData` struct as the two paint colours. Because it is the same struct, the hue and the paint reach the engine in one event -- which is why the hue is carried inside `paint` in a property table, and why it is unaffected by `paintApplied`.

Replicated to every viewer and replayed on stream-in. It is not in the physics owner's durable report, so it needs no intent window. In the current public build the hue is stored, replicated and read back (`getLightsHue`), but the client does not show it on the headlights, which stay white: the engine keeps the hue in the template without colouring the lamps.

**From 2.50.0-unstable+op77.205:** every client colours the car's headlights, the lamp glow and the beam, from the hue at full saturation and brightness, on every new body, after a stream-in and after each paint. Brake, reverse, blinker and interior lamps keep their own colours (`setLightColors`), and Open77's under-chassis neon is a separate native, `setNeon`. A client older than that release shows white headlights whatever the hue.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `hue` | `number 0..1|nil` | required | — |

Returns: `true, or false`, `reason: invalid_lights_hue, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- A tint slider from 0 to 1, and a "stock" button that clears it
RegisterNetEvent("tuning:lightsHue", function(hue)
    if hue then Open77.vehicles.setLightsHue(vehicleId, hue) else Open77.vehicles.clearLightsHue(vehicleId) end
end)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setLocked

```lua
Open77.vehicles.setLocked(id, locked)
```

`NETWORK`: Uses the network backend.

Sets or clears a vehicle's durable entry lock.

Requires `world.vehicles`. Atomically changes only `VehicleStateFlags.Locked`, publishes the canonical result to current viewers and late joiners, and does not overwrite unrelated flags. This blocks ordinary entry, not exit by an existing occupant; forced server seat assignment may deliberately bypass it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `locked` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Personal vehicles spawn locked; the owner unlocks with /unlock
local id = Open77.vehicles.create({ record = record, position = spot })
Open77.vehicles.setLocked(id, true)
Open77.vehicles.setLockedForPlayer(id, ownerId, false)
```

Registered in `LuaResourceRuntime.cs` (line 9112) as `LuaResourceRuntime.Bootstrap`.

## setLockedForAll

```lua
Open77.vehicles.setLockedForAll(id, locked)
```

`NETWORK`: Uses the network backend.

Sets the canonical entry lock and revokes every per-player exception.

Requires `world.vehicles`. Wiping the exceptions is the point, not a side effect: "lock this for everyone" has to mean everyone, including the key holders excepted a minute ago. `Open77.vehicles.setLocked` is unchanged and moves only the canonical bit, leaving exceptions standing -- use that one when the exceptions should survive. When the canonical bit was already at the requested value but exceptions were dropped, the whole audience is re-stated explicitly, because nothing else would tell the excepted players their exception is gone.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `locked` | `boolean` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- Lock the car for the street but keep it open for the two key holders
Open77.vehicles.setLockedForAll(vehicleId, true)
for _, holder in ipairs({ ownerId, partnerId }) do
    Open77.vehicles.setLockedForPlayer(vehicleId, holder, false)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setLockedForPlayer

```lua
Open77.vehicles.setLockedForPlayer(id, playerId, locked)
```

`NETWORK`: Uses the network backend.

Decides one player's view of a vehicle's entry lock.

Requires `world.vehicles`. `true` or `false` pins that player's view regardless of the canonical bit; `nil` drops the exception and puts them back on it. This is the shape a keys resource needs: `setLockedForAll(id, true)` for the street, then `setLockedForPlayer(id, holder, false)` for each key holder. The canonical bit stays `locked`, so a late joiner is told the right thing with no bookkeeping, and a key holder who disconnects takes their exception with them. The substitution happens at the single point where state reaches a socket, so no fan-out path can disagree with `isLockedForPlayer`. Per-player state is never written to the canonical flags, which is also why no owner report can disturb it -- `Locked` is outside the set an owner report is allowed to carry.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `playerId` | `integer` | required | — |
| `locked` | `boolean|nil` | required | — |

Returns: `true, or false`, `reason: invalid_argument, invalid_player_id, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
Open77.vehicles.setLockedForAll(id, true)
for _, holder in ipairs(keyHolders) do
  Open77.vehicles.setLockedForPlayer(id, holder, false)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setNeon

```lua
Open77.vehicles.setNeon(id, neon)
```

`NETWORK`: Uses the network backend.

Turns on or changes the under-chassis lights of a car.

Requires `world.vehicles`; a call that CHANGES the look also requires `world.vehicles.appearance` (`permission_denied:world.vehicles.appearance`), while restating the current value needs only the first -- the rule `setAppearance` follows, which keeps `setProperties(id, getProperties(id))` free. `neon` is `{ enabled?, color = {r,g,b} | {r=,g=,b=} | "#rrggbb", intensity? (0-5000, default 600), radius? (0.5-8 m, default 3), layout? ("auto") }`. Absent fields keep the current neon's values; `color` is required on a car without a neon. `enabled = false` keeps the colour and switches the lights off. The server owns the look: every visual setter writes one JSON value, the platform-reserved `open77.visual` key of the vehicle's state bag, which each client applies on stream-in, after an appearance change and after a rebuild, so it reaches current viewers, late joiners and reconnects alike. A resource cannot write that key itself (`reserved_state_key`).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `neon` | `table` | required | — |

Returns: `true, or false`, `reason: invalid_neon, invalid_neon_color, invalid_neon_intensity, invalid_neon_radius, invalid_neon_layout, emergency_look:neonAnimation, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
Open77.vehicles.setNeon(vehicleId, { enabled = true, color = "#00e5ff", intensity = 900 })
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setNeonAnimation

```lua
Open77.vehicles.setNeonAnimation(id, animation)
```

`NETWORK`: Uses the network backend.

Makes a car's neon pulse, breathe, strobe, chase, cycle hues or follow its speed or brakes.

Requires `world.vehicles`; a call that CHANGES the look also requires `world.vehicles.appearance`, while restating the current value needs only the first. `animation` is `{ mode, periodMs?, color2? }`, or `nil` for a steady neon. `mode` is one of `pulse`, `breathe`, `strobe`, `chase`, `rainbow`, `speed`, `brake`. `periodMs` is an integer from 250 to 10000 (a strobe at least 333, so never faster than 3 Hz); without it the mode's default is stored (breathe 4000, strobe 500, chase 1000, rainbow 6000, the others 1200). `color2` (`{r,g,b}`, `{r=,g=,b=}` or `"#rrggbb"`) is the second colour of pulse, breathe, strobe and chase, the colour `speed` blends to at speed and the colour `brake` flares to while braking; it is refused on `rainbow`. Every client computes the light from the server clock, so all of them show the same moment. The neon itself is still `setNeon`: without one, nothing shows. A red or blue strobe, or a red/blue pair in pulse, breathe, strobe or chase, is refused (`emergency_look:neonAnimation`), judged with the neon colour, so a later `setNeon` can be refused for it too.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `animation` | `table|nil` | required | — |

Returns: `true, or false`, `reason: invalid_neon_animation, invalid_neon_animation:mode, invalid_neon_animation:periodMs, invalid_neon_animation:color2, emergency_look:neonAnimation, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
Open77.vehicles.setNeon(vehicleId, { color = "#00e5ff" })
Open77.vehicles.setNeonAnimation(vehicleId, { mode = "chase", periodMs = 800, color2 = "#ff2bd6" })
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setPaint

```lua
Open77.vehicles.setPaint(id, paint)
```

`NETWORK`: Uses the network backend.

Sets authoritative paint on a canonical vehicle.

Requires `world.vehicles`. Accepts primary and optional secondary RGB colors as `#RRGGBB`, keyed `{ r, g, b }`, or positional tables. Omitting secondary mirrors primary. Paint is cross-resource and replicated to current viewers, stream-in projections, reconnects, and late joiners.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `paint` | `table` | required | — |

Returns: `boolean success`

```lua
Open77.vehicles.setPaint(vehicleId, {
  primary = "#00d8ff",
  secondary = { r = 12, g = 24, b = 36 },
})
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9121) as `LuaResourceRuntime.Bootstrap`.

## setPaintProfile

```lua
Open77.vehicles.setPaintProfile(id, profileId)
```

`NETWORK`: Uses the network backend.

Applies a TwinTone colour template (a livery) to a car, on every viewer.

Requires `world.vehicles`; a call that CHANGES the look also requires `world.vehicles.appearance` (`permission_denied:world.vehicles.appearance`), while restating the current value needs only the first -- the rule `setAppearance` follows, which keeps `setProperties(id, getProperties(id))` free. `profileId` is an id from `listPaintProfiles` for this car's record, or `nil` to go back to generic paint; an id the record does not list is `invalid_profile`. Restating the car's current profile always passes, even one whose pack is stopped. A later `setPaint`, `resetPaint` or repaint through `update` clears it. The server owns the look: every visual setter writes one JSON value, the platform-reserved `open77.visual` key of the vehicle's state bag, which each client applies on stream-in, after an appearance change and after a rebuild, so it reaches current viewers, late joiners and reconnects alike. A resource cannot write that key itself (`reserved_state_key`). On a car with stickers, a livery whose decals would take the car past 12 with them is refused (`decal_budget_exceeded`, see `getDecalBudget`).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `profileId` | `string|nil` | required | — |

Returns: `true, or false`, `reason: invalid_profile, decal_budget_exceeded, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- A garage's livery picker
local ok, reason = Open77.vehicles.setPaintProfile(vehicleId, profileId)
if not ok then TriggerClientEvent("garage:refused", playerId, reason) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setPartDetached

```lua
Open77.vehicles.setPartDetached(id, part, detached)
```

`NETWORK`: Uses the network backend.

Detaches one standard vehicle part.

Requires `world.vehicles`. The part may be an index 0..15 or a name from `Open77.vehicles.detachedParts`. Only `true` is accepted; respawn the vehicle to restore panels.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `part` | `integer|string` | required | — |
| `detached` | `boolean` | required | — |

Returns: `boolean success`

```lua
-- The crash tore off the trunk lid
Open77.vehicles.setPartDetached(vehicleId, "trunk", true)
```

Registered in `LuaResourceRuntime.cs` (line 9166) as `LuaResourceRuntime.Bootstrap`.

## setParts

```lua
Open77.vehicles.setParts(id, parts)
```

`NETWORK`: Uses the network backend.

Puts kit parts on a car, or takes them off.

Requires `world.vehicles`; a call that CHANGES the look also requires `world.vehicles.appearance` (`permission_denied:world.vehicles.appearance`), while restating the current value needs only the first -- the rule `setAppearance` follows, which keeps `setProperties(id, getProperties(id))` free. `parts` is `{ [partId] = variantId | true | false }`: `true` picks the only variant, `false` takes the part off; parts not named are left as they are. A bare id names a part of the calling resource, a qualified one (`<resource>:<id>`) any running resource's. A rim named here replaces the car's current rim, and two rims in one call answer `conflicting_parts`; which body parts exclude each other (two front lips) is the resource's to enforce. The server owns the look: every visual setter writes one JSON value, the platform-reserved `open77.visual` key of the vehicle's state bag, which each client applies on stream-in, after an appearance change and after a rebuild, so it reaches current viewers, late joiners and reconnects alike. A resource cannot write that key itself (`reserved_state_key`).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `parts` | `table` | required | — |

Returns: `true, or false`, `reason: unknown_part, part_not_for_model, unknown_variant, variant_required, conflicting_parts, invalid_parts, attach_limit, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
Open77.vehicles.setParts(vehicleId, { wing_gt = "carbon", no_antenna = true })
-- later: take the wing off
Open77.vehicles.setParts(vehicleId, { wing_gt = false })
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setPerformance

```lua
Open77.vehicles.setPerformance(id, profile)
```

`NETWORK`: Uses the network backend.

Caps one vehicle's top speed and pickup, from the server.

Requires `world.vehicles`. The profile is `{ topSpeedKph, accelerationScale, taperKph }`: a speed ceiling in km/h in `0..1000` (zero means none), throttle authority at every speed in `(0, 1]`, and how many km/h below the ceiling the throttle starts easing off, `3..200` with a default of `12`. **The taper is never zero**: below about 3 km/h the roll-off is short enough that the limiter hunts around the ceiling and it reads as stutter, which is why the client-local governor has refused that band since it shipped -- the server form enforces the identical envelope on the Lua boundary, in the registry and on both sides of the wire, so a server cap cannot be a way around a rule somebody measured. `nil` clears the ceiling, and so does a profile that asks for nothing -- no ceiling and full throttle authority. The taper is not part of that test: a roll-off width around a ceiling that does not exist changes nothing, which is how the native profile decides it too. This is the server-authoritative form of the client-local governor -- the same three numbers, decided by the server instead, which is what a tuning shop or a speed limit inside an instance needs. The ceiling is replicated to **every viewer**, not only to the client that currently owns the physics: ownership changes on every driver entry and exit, and a ceiling chased across those transitions would be missing for exactly the first second of every drive. It survives the physics owner's twice-a-second durable report by construction -- that report carries no performance field, so the merge has nothing to overwrite -- and it survives a projection that has not attached yet, because the client holds the intent on the replica and applies it when the entity appears. **Only server-spawned vehicles are in the registry.** A player standing next to a vanilla traffic car is standing next to something REDengine population spawned on their own client: it has no canonical id and this call cannot see it. Adopting vanilla traffic is a separate capability that does not exist yet.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `profile` | `table { topSpeedKph?, accelerationScale?, taperKph? }|nil` | required | — |

Returns: `true, or false`, `reason: invalid_argument, invalid_top_speed, invalid_acceleration, invalid_taper, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
Open77.vehicles.setPerformance(id, {
  topSpeedKph = 60.0, accelerationScale = 0.5, taperKph = 10.0,
})
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setPersistent

```lua
Open77.vehicles.setPersistent(id, persistent)
```

`NETWORK`: Uses the network backend.

Exempts a vehicle from every automatic removal.

Requires `world.vehicles`. One meaning, deliberately: no time-to-live takes it, the unobserved sweep does not take it, and it survives the stop of the resource that created it. An explicit `Open77.vehicles.remove` still takes it -- persistence is protection from the reapers, not from the owner. Taking persistence away hands the vehicle back to any deadline it still carries. Read back as `Open77.vehicles.get(id).persistent`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `persistent` | `boolean` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- The faction's tank stays in the world across resource restarts and reaper sweeps
local id = Open77.vehicles.create({ record = "Vehicle.v_militech_basilisk", position = base })
Open77.vehicles.setPersistent(id, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setPlate

```lua
Open77.vehicles.setPlate(id, text)
```

`NETWORK`: Uses the network backend.

Sets the text on a car's licence plate.

Requires `world.vehicles`; a call that CHANGES the look also requires `world.vehicles.appearance` (`permission_denied:world.vehicles.appearance`), while restating the current value needs only the first -- the rule `setAppearance` follows, which keeps `setProperties(id, getProperties(id))` free. The text is trimmed, then upper-cased; A-Z, 0-9, space and `-` only, 1 to 10 characters, no two spaces in a row, anything else is `invalid_plate`. `nil` or `""` goes back to the model's own plate. The platform rule is permissive on purpose: format, uniqueness and banned words are the resource's to enforce. Shown on models whose plate pack carries the plate widget; on a model without it the car keeps its stock plate and the text is still stored. The same plate in another case (`"nc 01"` after `"NC 01"`) is a restatement.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `text` | `string|nil` | required | — |

Returns: `true, or false`, `reason: invalid_plate, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
local ok, reason = Open77.vehicles.setPlate(vehicleId, "NC 2077")
if not ok then print("plate refused: " .. reason) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setPlayerAutoSeatSwitchEnabled

```lua
Open77.vehicles.setPlayerAutoSeatSwitchEnabled(playerId, enabled)
```

`NETWORK`: Uses the network backend.

Controls a resource-owned automatic takeover policy.

Requires world.vehicles. false adds this resource's veto for the player; true removes only its own veto. Explicit seat requests remain available. Policies are released on resource stop and player disconnect.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true or false`, `reason`

```lua
Open77.vehicles.setPlayerAutoSeatSwitchEnabled(playerId, false)
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9014) as `LuaResourceRuntime.Bootstrap`.

## setPlayerExitLocked

```lua
Open77.vehicles.setPlayerExitLocked(playerId, locked, [vehicleId])
```

`NETWORK`: Uses the network backend.

Prevents or permits a player-controlled vehicle exit.

Requires `world.vehicles`. While locked, native unmount and manual seat-switch attempts are rejected and reconciliation remounts any divergent local state. Seat policy is cross-resource, so an authorized gamemode or utility resource may coordinate vehicles created elsewhere. A server forced exit still overrides the lock. The optional vehicle ID is an optimistic guard against stale assignments.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `locked` | `boolean` | required | — |
| `vehicleId` | `integer` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Arrest: the suspect cannot leave the cruiser until released
Open77.vehicles.warpPlayerIntoVehicle(suspectId, cruiserId, Open77.vehicles.seats.rearLeft)
Open77.vehicles.setPlayerExitLocked(suspectId, true, cruiserId)
```

Registered in `LuaResourceRuntime.cs` (line 9079) as `LuaResourceRuntime.Bootstrap`.

## setPlayerIntoVehicle

```lua
Open77.vehicles.setPlayerIntoVehicle(playerId, vehicleId, seat, [options])
```

`NETWORK`: Uses the network backend.

Alias for authoritative player seat assignment.

Same contract as `warpPlayerIntoVehicle`. This name is useful to codebases that reserve `warp` for client presentation, while both server names create the same canonical seat assignment.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | required | — |
| `seat` | `integer|string` | required | — |
| `options` | `table` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- FiveM-style helper name for the same seat assignment
Open77.vehicles.setPlayerIntoVehicle(playerId, vehicleId, "driver")
```

Registered in `LuaResourceRuntime.cs` (line 9074) as `LuaResourceRuntime.Bootstrap`.

## setProperties

```lua
Open77.vehicles.setProperties(id, props, [options])
```

`NETWORK`: Uses the network backend.

Applies a stored property table to a vehicle.

Requires `world.vehicles`, plus `world.vehicles.appearance` if -- and only if -- the table CHANGES the appearance variant. Restating the variant a car already wears is free, which is what keeps the round trip working for a resource that never changes a look.

**`version` decides how absence is read, and it is the only thing that does.** A table carrying `version = 1` -- which is every table `getProperties` hands back -- is a COMPLETE description, so an absent optional field means "this car has none": no lights hue, no performance ceiling. A table without `version` is a PATCH, so an absent field is left exactly as it was, and `setProperties(id, { paint = { primary = "#FF0000" } })` is a repaint and nothing else.

**A property with no Cyberpunk counterpart is refused by name**, as `unsupported_property:modEngine` or `unsupported_property:paint.neonColor` -- the offending key alphabetically first, so the refusal is reproducible whatever `pairs` decides that run. Any key the table format does not name is refused the same way, `visual` included, and so are the FiveM keys `neonEnabled`, `neonColor`, `livery` and `windowTint`. Pass `{ ignoreUnsupported = true }` to skip them instead, in which case the third return value lists what was skipped: the alphabetically first unsupported key of the top level, of `paint` and of `damage`. Refusing is deliberate: a garage that silently loses two thirds of its table is worse than one that fails loudly on day one. `Open77.vehicles.unsupportedProperties` maps every such name to the reason.

`record` is checked, never applied: REDengine cannot change a spawned entity's record, so a Quadra table handed to a Thorton answers `record_mismatch` rather than half-restoring. Detached parts are one-way on a live projection, so a table asking for fewer than the car has already lost answers `detached_parts_not_reattachable`; respawn and apply the table to the new car.

**The order of application is deliberate.** Health, paint, doors, windows and the durable damage go in ONE `update`, so no viewer ever sees a half-restored car. The electrical bits then go through `setEngine`, `setLights`, `setSiren` and `setDrivable`, because those are what record the server intent that stops the physics owner's next durable report from undoing them -- writing them as raw flags would look like it worked and quietly revert about half a second later. The visual template and the performance ceiling follow on their own durable channels. Stable 147 includes engineHold in the property table; setProperties applies it through the canonical engine-hold setter.

**From 2.50.0-unstable+op77.205:** `setProperties` restores the vehicle customization too. The unsupported check also covers the `visual` table and runs before the vehicle is read or anything is applied, and `ignoreUnsupported` hands back every skipped key, sorted, nested ones prefixed (`paint.`, `damage.`, `visual.`). `visual` is applied as `getVisual` shapes it (`rev` is ignored), its wave 2 fields `neonAnimation`, `lightColors` and `stickers` included, with their setters' refusals, and the whole look is validated before anything is applied. **One exception, for a restore**: in a versioned table a sticker past the decal budget stays chosen but undrawn (dormant), as when a heavier livery is registered under the car, instead of refusing the table, since the table describes a car whose sticker was already dormant. A patch that adds a sticker is a purchase and is still refused with `decal_budget_exceeded`, as by `setStickers`. `neonEnabled` (a boolean, or FiveM's four per-side booleans) and `neonColor` become `visual.neon`; `livery` is a 0-based index into `listPaintProfiles(record)`, `-1` for none; `windowTint` is the FiveM index mapped onto `visual.tint`, GREEN (`6`) refused as `unsupported_property:windowTint`. A FiveM key wins over its `visual` field. In a versioned table a field absent from `visual` means none, but a table with no `visual` at all leaves the look alone. Changing the look needs `world.vehicles.appearance`; restating it does not. **Only a choice the car already holds survives a stopped pack:** on a freshly spawned car every part, livery and sticker is a new choice, so one that no running resource has registered is refused (`unknown_part`, `invalid_profile`, `unknown_sticker`) and the call applies nothing. `record` is then applied within the same model: a stage or engine-sound record of the same model (`_o77s<n>`, `_o77a<k>`, `_o77s<n>a<k>`) respawns the car onto it first, in place and under the same id, after every check and before the update, exactly as `respawnAs` would (`vehicle_occupied` while anyone is in it), so a garage that spawns the base model gets the tuned car back from the table. A sound record no running resource covers is not spawned blind: the sound is dropped and the stage kept. `visual.engineSound` must agree with `record` (`engine_sound_mismatch`); in a table without `record` it is applied like `setEngineSound`.

**Nothing is applied before the whole table is proved**: the record, the visual, the update, the electrical values, the appearance and hue, the ceiling, each by the checks of the setter that then applies it. A refused table leaves the car exactly as it was: not respawned, not repainted, its profile untouched. **A repaint replaces a TwinTone profile**, as it does through `update`: a table whose colours change the paint clears the profile whatever else it carries, unless the table states the profile itself (`visual.profile`, `livery`, or a `visual` in a versioned table), which then wins.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `props` | `table` | required | — |
| `options` | `table { ignoreUnsupported? }` | optional | — |

Returns: `true, or false`, `reason: unsupported_property:<name>, unsupported_version, record_mismatch, detached_parts_not_reattachable, invalid_argument, invalid_paint, invalid_damage, invalid_health, invalid_appearance, invalid_lights_hue, vehicle_not_found, permission_denied:world.vehicles, permission_denied:world.vehicles.appearance; from 2.50.0-unstable+op77.205 also vehicle_occupied, engine_sound_mismatch, invalid_engine_sound, invalid_visual and the refusals of the visual setters (unknown_part, invalid_profile, unknown_sticker, ...)`, `skipped: only with ignoreUnsupported, the unsupported names (the first of each table today; every one, sorted and prefixed, from 2.50.0-unstable+op77.205)`

```lua
local stored = json.decode(row.props)
local id = Open77.vehicles.create({ record = stored.record, position = spot })
local ok, reason = Open77.vehicles.setProperties(id, stored)
if not ok then print("garage restore failed: " .. reason) end
```

[Vehicle properties, and what Cyberpunk does not have](../../01%20Documentation/vehicles.md#vehicle-properties-and-what-cyberpunk-does-not-have)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setSiren

```lua
Open77.vehicles.setSiren(id, on)
```

`NETWORK`: Uses the network backend.

Turns a vehicle's siren on or off.

Requires `world.vehicles`. **The light bar and the sound are one bit and cannot be separated.** The only native this build exposes is `vehicleBaseObject.ToggleSiren(Bool)`, which takes no channel argument, so there is no `setSirenLights` that would not also make noise. The siren is also the one electrical bit no owner snapshot can read back, which is why the client echoes the canonical value instead of reporting it -- without that echo a cop car's bar went dark about five seconds after it was set.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `on` | `boolean` | required | — |

Returns: `true, or false`, `reason: invalid_argument, vehicle_not_found`

```lua
-- /siren toggles the caller's cruiser
RegisterCommand("siren", function(source)
    local seat = Open77.vehicles.getPlayerSeat(source)
    if not seat then return end
    local on = Open77.vehicles.isSirenOn(seat.vehicleId)
    Open77.vehicles.setSiren(seat.vehicleId, not on)
end, false)
```

[Vehicle sirens & key binding](../../01%20Documentation/vehicle-sirens.md)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setStickers

```lua
Open77.vehicles.setStickers(id, stickers)
```

`NETWORK`: Uses the network backend.

Sets the stickers on a car.

Requires `world.vehicles`; a change also requires `world.vehicles.appearance`. `stickers` is the whole list in order, at most 12 ids, each once; `nil` or `{}` takes them off. With the livery's own decals they must fit 12 decals: a thirteenth crashes every client that sees the car, so the server refuses it (`decal_budget_exceeded`, see `getDecalBudget`). A decal two stickers share counts once. A sticker whose pack stops stays chosen but is not shown until the pack comes back.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `stickers` | `table|nil` | required | — |

Returns: `true, or false`, `reason: invalid_stickers, unknown_sticker, sticker_not_for_model, decal_budget_exceeded, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
local ok, reason = Open77.vehicles.setStickers(vehicleId, { "st.hella.valentinos_01" })
if not ok then print("stickers refused: " .. reason) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setTimeToLive

```lua
Open77.vehicles.setTimeToLive(id, ttlMs)
```

`NETWORK`: Uses the network backend.

Gives a vehicle a deadline, in milliseconds from now.

Requires `world.vehicles`. `0` or `nil` cancels a deadline the vehicle already had, so a caller does not have to recreate it. Capped at seven days, mirroring the prop registry. The reaper runs once per tick, before interest is reconciled, so a vehicle whose lifetime ends leaves as a real removal the client tears down rather than quietly dropping out of the next interest pass. Read back as `Open77.vehicles.get(id).ttlMs`, counted from now and absent rather than zero once cancelled. A vehicle marked persistent ignores its deadline while it stays persistent. A deadline takes an **occupied** vehicle too, deliberately: a time-to-live silently declined because someone happened to be sitting in the car is a worse surprise than the removal, and the seat ledger is closed out properly either way. Clear the deadline on `onPlayerEnteredVehicle`, or mark the vehicle persistent, if that matters.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `ttlMs` | `number|nil` | required | — |

Returns: `true, or false`, `reason: invalid_ttl, vehicle_not_found, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- A job car is removed one hour after it was handed out
Open77.vehicles.setTimeToLive(vehicleId, 60 * 60 * 1000)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setTireBroken

```lua
Open77.vehicles.setTireBroken(id, index, broken)
```

`NETWORK`: Uses the network backend.

Sets or clears one broken tire.

Requires `world.vehicles`. Mutates one tire index in 0..3 and replicates the canonical state to streamed observers.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `index` | `integer` | required | — |
| `broken` | `boolean` | required | — |

Returns: `boolean success`

```lua
-- Spike strip: burst the two front tires
Open77.vehicles.setTireBroken(vehicleId, 0, true)
Open77.vehicles.setTireBroken(vehicleId, 1, true)
```

Registered in `LuaResourceRuntime.cs` (line 9154) as `LuaResourceRuntime.Bootstrap`.

## setTireMask

```lua
Open77.vehicles.setTireMask(id, mask)
```

`NETWORK`: Uses the network backend.

Replaces the broken-tire bit mask.

Requires `world.vehicles`. Only the lower four bits are valid and correspond to tire indexes 0..3.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `mask` | `integer` | required | — |

Returns: `boolean success`

```lua
-- All four tires burst at once (bits 0..3)
Open77.vehicles.setTireMask(vehicleId, 15)
```

Registered in `LuaResourceRuntime.cs` (line 9153) as `LuaResourceRuntime.Bootstrap`.

## setTransform

```lua
Open77.vehicles.setTransform(id, transform)
```

`NETWORK`: Uses the network backend.

Moves a vehicle authoritatively.

Requires `world.vehicles`. Updates position and yaw, revokes any active physics lease, and publishes the new canonical transform to current viewers. Two table shapes are accepted and read identically: `{ x, y, z, yaw }` at the top level, or `{ position = { x, y, z }, yaw }` (the wrapper reads `def.position or def`, then `def.yaw`). A missing coordinate keeps the vehicle's current value; a missing `yaw` becomes **0**, not the current heading, so always pass it. `yaw` is degrees about Z, as `create` takes it. `SetVehicleTransform(id, x, y, z, yaw)` is the positional twin. There is no native that attaches a vehicle to another vehicle or to a player -- `Open77.props.attach` only parents a *prop* -- so a tow or a trailer is a resource calling this per update, not a lease.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `transform` | `table { x, y, z, yaw } or { position = { x, y, z }, yaw }` | required | — |

Returns: `boolean success`

```lua
-- Tow the car to the impound lot; both shapes below are the same call
Open77.vehicles.setTransform(vehicleId, {
    position = { x = -1442.2, y = 127.4, z = 18.0 },
    yaw = 180.0,   -- omitted yaw would become 0, not the current heading
})
Open77.vehicles.setTransform(vehicleId, { x = -1442.2, y = 127.4, z = 18.0, yaw = 180.0 })
```

Registered in `LuaResourceRuntime.cs` (line 9213) as `LuaResourceRuntime.Bootstrap`.

## setTtl

```lua
Open77.vehicles.setTtl(id, ttlMs)
```

`NETWORK`: Uses the network backend.

Alias of `Open77.vehicles.setTimeToLive`.

Requires `world.vehicles`. The short spelling, matching the `ttlMs` key on `Open77.vehicles.create`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `ttlMs` | `number|nil` | required | — |

Returns: `true, or false`, `reason: invalid_ttl, vehicle_not_found`

```lua
-- Same as setTimeToLive: this rental disappears after 15 minutes
Open77.vehicles.setTtl(vehicleId, 15 * 60 * 1000)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setUndriveable

```lua
Open77.vehicles.setUndriveable(id, undriveable)
```

`NETWORK`: Uses the network backend.

Takes a vehicle out of service, or puts it back.

Requires `world.vehicles`. The inverted spelling of `Open77.vehicles.setDrivable`, for callers porting `SetVehicleUndriveable`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `undriveable` | `boolean` | required | — |

Returns: `true, or false`, `reason`

```lua
-- FiveM port: SetVehicleUndriveable(veh, true) becomes
Open77.vehicles.setUndriveable(vehicleId, true)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setVehicleProperties

```lua
Open77.vehicles.setVehicleProperties(id, props, [options])
```

`NETWORK`: Uses the network backend.

The FiveM-shaped name for `setProperties`.

Identical contract, for a resource ported from a framework that spells it `SetVehicleProperties`. `applyProperties` is a third name for the same function.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `props` | `table` | required | — |
| `options` | `table` | optional | — |

Returns: `true, or false`, `reason`

```lua
-- FiveM-style garage restore works unchanged
local ok, reason = Open77.vehicles.setVehicleProperties(vehicleId, json.decode(row.props))
if not ok then print(reason) end
```

[Vehicle properties, and what Cyberpunk does not have](../../01%20Documentation/vehicles.md#vehicle-properties-and-what-cyberpunk-does-not-have)

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## setWheels

```lua
Open77.vehicles.setWheels(id, wheels)
```

`NETWORK`: Uses the network backend.

Fits a rim design on every wheel of a car, or puts the factory rims back.

Requires `world.vehicles`; a call that CHANGES the look also requires `world.vehicles.appearance` (`permission_denied:world.vehicles.appearance`), while restating the current value needs only the first -- the rule `setAppearance` follows, which keeps `setProperties(id, getProperties(id))` free. `wheels` is `{ rim = partId, variant? }`; `nil` or `{}` takes the rim off. A bare `rim` names a part of the calling resource; a qualified id (`<resource>:<id>`, the form `listParts` answers) names a part any running resource registered, so a shop resource selling another resource's rims passes the qualified id. `variant` may be left out when the rim has a single variant. The rim goes on every wheel slot (four on a car, two on a bike, or the fit's own `wheels` list for this record) and hides the factory rim components its fit lists for the model, or for the car's current appearance when the fit has an override for it. The server owns the look: every visual setter writes one JSON value, the platform-reserved `open77.visual` key of the vehicle's state bag, which each client applies on stream-in, after an appearance change and after a rebuild, so it reaches current viewers, late joiners and reconnects alike. A resource cannot write that key itself (`reserved_state_key`).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `wheels` | `table|nil` | required | — |

Returns: `true, or false`, `reason: invalid_wheels, unknown_part, part_not_a_rim, part_not_for_model, unknown_variant, variant_required, attach_limit, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
-- a rim this resource registered
Open77.vehicles.setWheels(vehicleId, { rim = "rim_fork", variant = "black" })
-- a rim another resource registered: pass the qualified id listParts answers
Open77.vehicles.setWheels(vehicleId, { rim = "my_kit:rim_fork", variant = "black" })
-- back to the factory rims
Open77.vehicles.setWheels(vehicleId, nil)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## setWindowOpen

```lua
Open77.vehicles.setWindowOpen(id, window, opened)
```

`NETWORK`: Uses the network backend.

Sets one reversible side-window opening state.

Requires `world.vehicles`. This controls the four openable side windows and is separate from destructible broken-glass state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `window` | `integer` | required | — |
| `opened` | `boolean` | required | — |

Returns: `boolean success`

```lua
-- Open the rear-right window for the passenger
Open77.vehicles.setWindowOpen(vehicleId, Open77.vehicles.windows.backRight, true)
```

Registered in `LuaResourceRuntime.cs` (line 9195) as `LuaResourceRuntime.Bootstrap`.

## setWindowTint

```lua
Open77.vehicles.setWindowTint(id, level)
```

`NETWORK`: Uses the network backend.

Sets the window tint level of a car.

Requires `world.vehicles`; a call that CHANGES the look also requires `world.vehicles.appearance` (`permission_denied:world.vehicles.appearance`), while restating the current value needs only the first -- the rule `setAppearance` follows, which keeps `setProperties(id, getProperties(id))` free. `level` is an integer from 1 (light smoke) to 3 (limo), or `0` / `nil` for factory glass. A fraction, a string or a value out of range is `invalid_tint`. The level is stored on any car; it is drawn on the side and rear glass of the models the client's tint pack covers (`customizationAvailable` on the client names the others), never on the windshield. Which models a resource offers tint on is its own choice.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `level` | `integer 0..3|nil` | required | — |

Returns: `true, or false`, `reason: invalid_tint, visual_too_large, permission_denied:world.vehicles.appearance, vehicle_not_found, invalid_vehicle_id, vehicles_unavailable, permission_denied:world.vehicles`

```lua
Open77.vehicles.setWindowTint(vehicleId, 2)
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## startEngine

```lua
Open77.vehicles.startEngine(id)
```

`NETWORK`: Uses the network backend.

Starts a vehicle's engine.

Requires `world.vehicles`. `Open77.vehicles.setEngine(id, true)` under a name that reads as an action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Remote start from the key fob
RegisterCommand("remotestart", function(source)
    local nearest = Open77.vehicles.closest(source, { radius = 20 })
    if nearest then Open77.vehicles.startEngine(nearest.id) end
end, false)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## stopEngine

```lua
Open77.vehicles.stopEngine(id)
```

`NETWORK`: Uses the network backend.

Stops a vehicle's engine.

Requires `world.vehicles`. `Open77.vehicles.setEngine(id, false)` under a name that reads as an action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `true, or false`, `reason`

```lua
-- Cut the engine when the driver is arrested
Open77.vehicles.stopEngine(vehicleId)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## switchPlayerSeat

```lua
Open77.vehicles.switchPlayerSeat(playerId, seat)
```

`NETWORK`: Uses the network backend.

Reserves an animated seat change for a seated player.

Requires world.vehicles and protocol 1.44. Both seats are held until native completion. Returns true and a string transition ID on reservation, or false and a reason. Completion/failure is reported by onPlayerVehicleSeatSwitchCompleted/Failed. Does not board on-foot players or evict occupants. Explicit requests remain allowed when automatic takeover is disabled.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `seat` | `string|integer` | required | — |

Returns: `true or false`, `transitionId on acceptance, otherwise reason`

```lua
local ok, id = Open77.vehicles.switchPlayerSeat(playerId, 'rearRight')
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#server-api)

Registered in `LuaResourceRuntime.cs` (line 9014) as `LuaResourceRuntime.Bootstrap`.

## taskPlayerEnter

```lua
Open77.vehicles.taskPlayerEnter(playerId, vehicleId, seat, [options])
```

`NETWORK`: Uses the network backend.

Seats a player through the vanilla entry animation.

Requires `world.vehicles`. The same authoritative reservation as `warpPlayerIntoVehicle`, taking the same `options` and answering with the same refusal reasons; the only difference is that the affected client is asked to reach the seat by mounting through the mounting facility and playing the authored entry, instead of being placed in the seat in one frame. It does **not** walk the player to the door: the approach is an NPC behaviour, and driving a network puppet through it crashed the observing client when it was measured, so the player is mounted where they stand and slides in. The animation is bounded — a client whose mounting relation does not become readable within three seconds warps instead — so the seat is always reached and `warpPlayerIntoVehicle` remains the path that never depends on an animation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | required | — |
| `seat` | `integer|string` | required | — |
| `options` | `table` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
local ok, reason = Open77.vehicles.taskPlayerEnter(playerId, vehicleId, Open77.vehicles.seats.driver)
if not ok then
  -- every refusal is the warp's refusal, verbatim
  print(reason)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## taskPlayerEnterVehicle

```lua
Open77.vehicles.taskPlayerEnterVehicle(playerId, vehicleId, seat, [options])
```

`NETWORK`: Uses the network backend.

Alias for the animated authoritative seat assignment.

Same contract as `Open77.vehicles.taskPlayerEnter`. The longer spelling mirrors the client-side `Open77.vehicles.taskPlayerEnterVehicle`, which presents an already-authorised **remote** occupant; this server name creates the canonical assignment that such a presentation would be showing.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | required | — |
| `seat` | `integer|string` | required | — |
| `options` | `table` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The player walks up and climbs in with the animation
Open77.vehicles.taskPlayerEnterVehicle(playerId, vehicleId, Open77.vehicles.seats.frontPassenger)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## taskPlayerLeave

```lua
Open77.vehicles.taskPlayerLeave(playerId, [vehicleId])
```

`NETWORK`: Uses the network backend.

Ejects a player from a network vehicle.

Requires `world.vehicles`. The FiveM-shaped name for the same durable forced exit as `forcePlayerOutOfVehicle`, and it is honest about not being the mirror image of `taskPlayerEnter`: **the leaving player's own exit is instant on this build.** That is a measurement, not an omission — driving the local player's authored exit workspot inside the mount release window faulted on the engine's release-dispatch thread and the workspot never started once, so the client does not attempt it. Every other player still sees this occupant's exit animated, because a remote proxy's exit has always been; only the ejected player's own screen skips it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The taxi reached its destination: the customer gets out
Open77.vehicles.taskPlayerLeave(customerId, taxiId)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## taskPlayerLeaveVehicle

```lua
Open77.vehicles.taskPlayerLeaveVehicle(playerId, [vehicleId])
```

`NETWORK`: Uses the network backend.

Alias for the authoritative vehicle exit.

Same contract as `Open77.vehicles.taskPlayerLeave`, spelled to match `taskPlayerEnterVehicle`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The ride is over: the passenger gets out
Open77.vehicles.taskPlayerLeaveVehicle(passengerId, taxiId)
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## triggerHorn

```lua
Open77.vehicles.triggerHorn(id, [durationMs])
```

`NETWORK`: Uses the network backend.

Sounds a synchronized vehicle horn.

Requires `world.vehicles`. Sends a reliable, authority-epoch-bound one-shot command to the client currently simulating the vehicle. An unoccupied parked vehicle intentionally has no physics owner, so the server addresses all current viewers directly without granting a lease. Duration is 100..2000 ms and defaults to 250 ms. The event is transient and is not late-join state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `durationMs` | `integer` | optional | `250` |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Honk when the player's rental is unlocked remotely
Open77.vehicles.unlock(vehicleId)
Open77.vehicles.triggerHorn(vehicleId, 200)
```

Registered in `LuaResourceRuntime.cs` (line 9119) as `LuaResourceRuntime.Bootstrap`.

## unlock

```lua
Open77.vehicles.unlock(id)
```

`NETWORK`: Uses the network backend.

Unlocks a vehicle for ordinary entry.

Convenience alias for `setLocked(id, false)`. Requires `world.vehicles` and updates durable canonical state.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The renter paid: unlock their car
Open77.vehicles.unlock(rentalId)
```

Registered in `LuaResourceRuntime.cs` (line 9114) as `LuaResourceRuntime.Bootstrap`.

## unlockForAll

```lua
Open77.vehicles.unlockForAll(id)
```

`NETWORK`: Uses the network backend.

Unlocks a vehicle for everyone, revoking every exception.

Requires `world.vehicles`. `Open77.vehicles.setLockedForAll(id, false)` under a name that reads as an action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |

Returns: `true, or false`, `reason: vehicle_not_found`

```lua
-- Event: every car in the lot is free to take
for _, entry in ipairs(Open77.vehicles.all()) do
    Open77.vehicles.unlockForAll(entry.id)
end
```

Registered in `LuaResourceRuntime.cs` (line 8999) as `LuaResourceRuntime.Bootstrap`.

## unsupportedProperties

```lua
Open77.vehicles.unsupportedProperties()
```

`NETWORK`: Uses the network backend.

The FiveM vehicle-property names with no Cyberpunk counterpart, and why.

A table, not a function: each key is a FiveM `GetVehicleProperties` field this engine cannot express, and each value says what is missing. `setProperties` refuses every one of them by name. Read it to decide, before writing, what a ported script has to drop -- and read it instead of guessing, because several of the absences are surprising: there are no colour INDICES (Cyberpunk paint is RGB), no livery index (the appearance variant is a name, not a number), and no plate at all.

In the current public build the list covers `modKit`, `mods`, `modEngine`, `modBrakes`, `modTransmission`, `modSuspension`, `modArmour`, `modTurbo`, `wheels`, `wheelType`, `wheelColor`, `windowTint`, `neonEnabled`, `neonColor`, `xenonColor`, `livery`, `roofLivery`, `extras`, `color1`, `color2`, `pearlescentColor`, `interiorColor`, `dashboardColor`, `dirtLevel`, `tyreSmokeColor`, `plateText`, `plateIndex` and `fuelLevel`.

**From 2.50.0-unstable+op77.205:** `neonEnabled`, `neonColor` and `livery` leave the table, because `setProperties` reads them into `visual`; `windowTint` stays only for its GREEN index (`6`); and several values name the Open77 native to use instead: rims are `setWheels`, kit parts `registerParts` / `setParts`, plate text `setPlate` (`plateText` itself is still refused in a property table). Read the table at run time rather than copying either list.

Returns: `table mapping property name to the reason it has no counterpart`

```lua
for name, why in pairs(Open77.vehicles.unsupportedProperties) do
  print(("%s: %s"):format(name, why))
end
```

[Vehicle properties, and what Cyberpunk does not have](../../01%20Documentation/vehicles.md#vehicle-properties-and-what-cyberpunk-does-not-have)

Registered in `LuaResourceRuntime.cs` (line 9499) as `LuaResourceRuntime.Bootstrap`.

## update

```lua
Open77.vehicles.update(id, patch)
```

`NETWORK`: Uses the network backend.

Updates canonical durable vehicle fields.

Requires `world.vehicles`. Applies only supplied fields, such as health, flags, colors, openings, tires, body damage, broken glass/lights, or detached parts. Any authorized resource may update any canonical vehicle; the authoritative result is replicated to current viewers and late joiners.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `patch` | `table` | required | — |

Returns: `boolean success`

```lua
-- Patch just the health and colours of a canonical vehicle
Open77.vehicles.update(vehicleId, {
    health = 0.8,
    primaryColor = { r = 200, g = 20, b = 20 },
})
```

Registered in `LuaResourceRuntime.cs` (line 9110) as `LuaResourceRuntime.Bootstrap`.

## warpPlayerIntoVehicle

```lua
Open77.vehicles.warpPlayerIntoVehicle(playerId, vehicleId, seat, [options])
```

`NETWORK`: Uses the network backend.

Assigns a player to an authoritative vehicle seat.

Requires `world.vehicles`. The server reserves the seat immediately, optionally moves the player into the vehicle routing bucket, and keeps a durable forced-entry order until the target streams and the client confirms the native mount. Seat coordination is intentionally cross-resource: any resource with the permission may target any canonical vehicle. Proximity and the vehicle locked flag are bypassed; destroyed/exploded vehicles and occupied seats are rejected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `vehicleId` | `integer` | required | — |
| `seat` | `integer|string` | required | — |
| `options` | `table` | optional | — |

Returns: `true on success, otherwise false`, `reason`

```lua
local ok, reason = Open77.vehicles.warpPlayerIntoVehicle(playerId, vehicleId, Open77.vehicles.seats.driver, {
  moveBucket = true,
  exitLocked = true,
})
```

Registered in `LuaResourceRuntime.cs` (line 9060) as `LuaResourceRuntime.Bootstrap`.

## windows

```lua
Open77.vehicles.windows()
```

`NETWORK`: Uses the network backend.

Window indexes for the glass natives.

A public constant table: `frontLeft = 0`, `frontRight = 1`, `backLeft = 2`, `backRight = 3`.

Returns: `table { frontLeft = 0, frontRight = 1, backLeft = 2, backRight = 3 }`

```lua
Open77.vehicles.setWindowOpen(id, Open77.vehicles.windows.frontLeft, true)
```

Registered in `LuaResourceRuntime.cs` (line 9054) as `LuaResourceRuntime.Bootstrap`.
