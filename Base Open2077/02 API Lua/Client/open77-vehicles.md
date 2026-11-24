# Open77.vehicles (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-vehicles.md) · [Page web](https://open2077.net/docs/api/client/open77-vehicles)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

51 functions.

## adoptable

```lua
Open77.vehicles.adoptable([options])
```

`GAME`: Requires a live game instance.

Every engine-spawned vehicle near the player that the server does not own.

Requires `vehicles.read`. This is the only read on either side that can see a car REDengine spawned: `Open77.vehicles.get`, `nearby`, `closest` and `aimed` all answer from the replicated registry, which is why `aimed()` reports `unknown_entity` when the player is plainly looking at a traffic car. Options are `radius` (default 25, at most 200) and `limit` (default 8, at most 32); a bare number is read as the radius. Nearest first. Each entry carries `engineEntity` (a fixed `0x` string, because a REDengine entity hash does not survive a Lua number), `record`, `recordId`, `adoptable`, `position`, `orientation`, `distance`, `speed`, `speedKph`, `health`, `engineOn`, `locked`, `destroyed`, `doors`, `windows`, `tires`, `brokenGlass`, `brokenLights` and `detachedParts` -- everything the server's `Open77.vehicles.adopt` reads, spelled the same, so an entry can be handed over untouched. `adoptable` is false, and `record` empty, when the car's TweakDBID is not in the shipped reverse table: a shipping build keeps no id-to-name table and `gamedataVehicle_Record` has no `Name` accessor, so the 1376 known 2.31 records are carried and anything else reports its raw id and nothing more. `world_query_layout_unknown` means the targeting part layout has not been proven yet this session -- a precondition shared with the interaction prompts, and reported rather than answered with an empty list, because "the query cannot run" and "there is no car here" need opposite fixes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table|number` | optional | — |

Returns: `array of descriptions, or nil`, `reason`

```lua
-- Car theft: propose the nearest street car to the server when the key is pressed
RegisterKeyMapping("steal_car", "Steal the nearest car", "G", function()
    local cars = Open77.vehicles.adoptable({ radius = 6, limit = 1 })
    if not cars or not cars[1] then return Open77.hud.notify("No car close enough") end
    TriggerServerEvent("garage:adopt", cars[1])
end)
```

Registered in `ResourceHost.cpp` (line 25101) as `LuaVehiclesAdoptable`.

## aimed

```lua
Open77.vehicles.aimed()
```

`GAME`: Requires a live game instance.

The replicated vehicle the local player is looking at.

Requires `vehicles.read`. One `gametargetingTargetingSystem.GetLookAtObject` probe, right now, mapped through the replica table -- not the permanent inspector ray, which answers nothing unless some resource turned it on. Returns the full vehicle snapshot. `unknown_entity` is the interesting failure and it is COMMON: the player really is looking at a car, and that car is vanilla traffic no server resource created. It does not mean 'nothing is there'.

Returns: `vehicle snapshot, or nil`, `reason: permission_denied:vehicles.read, no_target, unknown_entity, aim_probe_unavailable_on_this_host`

```lua
-- Show the id of the server vehicle the player is looking at
RegisterKeyMapping("inspect_car", "Inspect vehicle", "K", function()
    local car, reason = Open77.vehicles.aimed()
    if not car then return Open77.hud.notify(reason == "unknown_entity" and "That is vanilla traffic" or "Look at a vehicle") end
    Open77.hud.notify(("Vehicle #%d %s"):format(car.id, car.record))
end)
```

Registered in `ResourceHost.cpp` (line 25097) as `LuaVehicleAimed`.

## all

```lua
Open77.vehicles.all()
```

`GAME`: Requires a live game instance.

Lists every vehicle currently streamed to this client.

Requires `vehicles.read`. Returns an empty array when permission or the projection backend is unavailable. Every entry has the exact shape returned by `Open77.vehicles.get`; server vehicles outside the client's streaming interest set are intentionally absent.

Returns: `array of vehicle snapshots`

```lua
-- Draw a blip on every server vehicle streamed to this client
for _, car in ipairs(Open77.vehicles.all()) do
    Open77.blips.create({ position = car.position, label = car.record })
end
```

Registered in `ResourceHost.cpp` (line 25076) as `LuaVehiclesAll`.

## cancelSeatSwitch

```lua
Open77.vehicles.cancelSeatSwitch(transitionId)
```

`GAME`: Requires a live game instance.

Requests cancellation of an active local seat transition.

Requires vehicles.control. transitionId is the string returned by seatSwitchState or its Started event. A stale ID cannot cancel a later transition. Cancellation releases the destination and restores the source; true means submitted to the server.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `transitionId` | `string` | required | — |

Returns: `true when submitted, otherwise false`, `reason`

```lua
local s = Open77.vehicles.seatSwitchState()
if s and s.active then Open77.vehicles.cancelSeatSwitch(s.id) end
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#client-api)

Registered in `ResourceHost.cpp` (line 25244) as `LuaVehicleCancelSeatSwitch`.

## clearAllPerformance

```lua
Open77.vehicles.clearAllPerformance()
```

`GAME`: Requires a live game instance.

Drops every performance ceiling, instance and record class alike.

Requires `vehicles.performance`. Intended for a gamemode teardown so a stopped resource cannot leave cars governed.

Returns: `true`, `empty reason`

```lua
-- The race resource stops: no car stays governed
AddEventHandler("onClientResourceStop", function(name)
    if name == GetCurrentResourceName() then Open77.vehicles.clearAllPerformance() end
end)
```

Registered in `ResourceHost.cpp` (line 25091) as `LuaVehicleClearAllPerformance`.

## clearPerformance

```lua
Open77.vehicles.clearPerformance(vehicleId)
```

`GAME`: Requires a live game instance.

Removes one vehicle's performance ceiling.

Requires `vehicles.performance`. The vehicle falls back to its record class profile, then the default, then to untouched native behaviour.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer|string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The player left the slow zone: lift the cap on their car
AddEventHandler("slowzone:left", function()
    local seat = Open77.vehicles.getPlayerSeat()
    if seat then Open77.vehicles.clearPerformance(seat.vehicleId) end
end)
```

Registered in `ResourceHost.cpp` (line 25088) as `LuaVehicleClearPerformance`.

## clearPerformanceClass

```lua
Open77.vehicles.clearPerformanceClass(record)
```

`GAME`: Requires a live game instance.

Removes a TweakDB record's performance ceiling.

Requires `vehicles.performance`. Vehicles of that record fall back to the default profile, then to untouched native behaviour.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Rebalance: the Quadra is no longer capped
Open77.vehicles.clearPerformanceClass("Vehicle.v_sport2_quadra_turbo_r")
```

Registered in `ResourceHost.cpp` (line 25090) as `LuaVehicleClearPerformanceClass`.

## closest

```lua
Open77.vehicles.closest([radiusOrOptions])
```

`SHARED`: Available without a live game instance.

The nearest replicated vehicle.

Requires `vehicles.read`. A bare number is the radius, which is the spelling almost every caller wants; the table form takes the same options as `nearby` plus `radius`, and without one the search is unbounded. `no_vehicle_in_range` is a real answer, not an error. **Only server-spawned vehicles are in the registry.** A player standing next to a vanilla traffic car is standing next to something REDengine population spawned on their own client: it has no canonical id and this call cannot see it. Adopting vanilla traffic is a separate capability that does not exist yet. This is the call an RP resource reaches for first, so read that paragraph twice: `closest()` answering a car forty metres away while the player leans on a taxi is the expected behaviour, not a bug.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `radiusOrOptions` | `number|table { radius?, origin?, streamed?, occupied? }` | optional | — |

Returns: `entry, or nil`, `reason: permission_denied:vehicles.read, invalid_radius, no_vehicle_in_range, no_session, no_local_position`

```lua
-- Press E near a server vehicle to ask the server to unlock it
RegisterKeyMapping("unlock_car", "Unlock nearby car", "E", function()
    local car, reason = Open77.vehicles.closest(5.0)
    if not car then return end -- no_vehicle_in_range is a normal answer
    TriggerServerEvent("keys:unlock", car.id)
end)
```

Registered in `ResourceHost.cpp` (line 25096) as `LuaVehiclesClosest`.

## customizationAvailable

```lua
Open77.vehicles.customizationAvailable(id)
```

`GAME`: Requires a live game instance.

Reads what this vehicle's record supports and how far its look got on this client.

Requires `vehicles.read`. Client-side and read-only. Answers from the live TweakDB and from this client's copy of the server's visual value: `{ record, paint, rims, lights, twintone, profiles, appearance, streamed, destroyed, rev, appliedRev, neonLit, skipped }`. A capability the record did not answer is absent (nil). `skipped` lists `"<category>:<item>:<reason>"` strings for what this game could not draw: `attach:<key>:model_not_registered` when its parts pack is missing, `hide:*:held_parts_missing` while it keeps the factory parts those pieces replace, `tint:<level>:pending_mesh` while the glass is still loading. Read `skipped` only once `appliedRev` has caught up with `rev`. Returns nil and `unknown_vehicle`, `vehicles_unavailable` or `permission_denied:vehicles.read` on failure.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `value`, `reason (when the first is nil)`

```lua
-- Tell the player when their game cannot draw a part the server put on the car
local state, reason = Open77.vehicles.customizationAvailable(vehicleId)
if not state then print(reason); return end
if (state.appliedRev or 0) < (state.rev or 0) then return end -- not applied yet
for _, entry in ipairs(state.skipped or {}) do
    if entry:find(":model_not_registered", 1, true) then
        print("parts pack missing, reconnect to download it: " .. entry)
    end
end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `ResourceHost.cpp` (line 27735) as `LuaVehicleCustomizationAvailable`.

## describeAdoptable

```lua
Open77.vehicles.describeAdoptable(engineEntity)
```

`GAME`: Requires a live game instance.

The same description for one named engine-spawned chassis.

Requires `vehicles.read`. Takes the `engineEntity` string an `adoptable()` entry carries (a plain integer id also works). `already_managed` means the server already owns that car and it is not a candidate; `vehicle_not_found` means it is not within eighty metres of the player, which is as far as this read looks.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `engineEntity` | `string|integer` | required | — |

Returns: `description, or nil`, `reason`

```lua
-- Re-describe one chassis right before proposing it, so the position is fresh
local description, reason = Open77.vehicles.describeAdoptable(engineEntity)
if description then TriggerServerEvent("garage:adopt", description) end
```

Registered in `ResourceHost.cpp` (line 25102) as `LuaVehicleDescribeAdoptable`.

## fromEntity

```lua
Open77.vehicles.fromEntity(entityId)
```

`GAME`: Requires a live game instance.

The replicated vehicle wearing an entity id.

Requires `vehicles.read`. Accepts either id a vehicle carries: the Open77 `entity` (what `Open77.players.entity` and a raycast hand back) or the REDengine `engineEntity`. Returns the full snapshot, so `fromEntity(entity).id` is the round trip from a world query to a canonical vehicle id.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entityId` | `entity id` | required | — |

Returns: `vehicle snapshot, or nil`, `reason: permission_denied:vehicles.read, invalid_entity, unknown_entity`

```lua
-- Turn a raycast hit into a canonical vehicle id
local hit = Open77.world.raycast(from, to, { entities = true })
if hit and hit.entity and hit.entity.kind == "vehicle" then
    local car = Open77.vehicles.fromEntity(hit.entity.engineEntity)
    if car then print("looking at vehicle", car.id) end
end
```

Registered in `ResourceHost.cpp` (line 25098) as `LuaVehicleFromEntity`.

## get

```lua
Open77.vehicles.get(id)
```

`GAME`: Requires a live game instance.

Reads one streamed server-owned vehicle.

Client read-only API requiring `vehicles.read`. The snapshot contains durable state (`health`, flags, the derived `locked`, `undriveable`/`drivable` and `frozen` booleans -- the last two are server-authored bits no owner report can clear, see `wiki/vehicles.md` -- six door bits, four open-window bits, tyre/glass/light masks, detached panels and the 30-cell body grid), authority and occupants including durable seat-policy flags, an ephemeral generation-checked local entity handle, plus the replicated transform and the latest drivetrain and wheel telemetry. The transform is `position` (a plain `{ x, y, z }`, which is what an Open77 vector3 is), `orientation` as `{ x, y, z, w }`, and `heading`/`yaw` -- one value under both names, in degrees, extracted from the full quaternion so a car on a slope reports the heading it is actually facing, and matching the server vehicle read exactly. Both come from the latest replicated motion when one exists and from the canonical create otherwise, so a proximity query answers for a car whose REDengine projection has not attached yet. `speed` is metres per second, the unit every Open77 read uses, and `speedKph` is the same number converted once because performance ceilings and speed limits are authored in km/h. Open windows and broken glass are separate states. Creation, repair, authoritative seat assignment and arbitrary mutation are server-only; the complete server surface is documented in `wiki/vehicles.md`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `table { id, record, entity, engineEntity, revision, physicsOwner, authorityEpoch, flags, locked, health, position, orientation, heading, yaw, doors, windows, tires, bodyDamage, brokenGlass, brokenLights, detachedParts, damage, speed, speedKph, rpm, rpmMax, throttle, brake, steering, wheelRotation, suspensionLongitudinal, suspensionTransversal, burnout, gear, onGround, reversing, streamed, locallyOwned, occupants }, or nil`

```lua
-- A dashboard: speed and health of the car the player is driving
SetTick(function()
    local seat = Open77.vehicles.getPlayerSeat()
    if not seat then return end
    local car = Open77.vehicles.get(seat.vehicleId)
    if car then dashboard.set(car.speedKph, car.health) end
end)
```

Registered in `ResourceHost.cpp` (line 25063) as `LuaVehicleGet`.

## getActiveWeapons

```lua
Open77.vehicles.getActiveWeapons(vehicleId)
```

`GAME`: Requires a live game instance.

Lists the attached weapons selected by the driver.

Requires `vehicles.read`. Client-side, synchronous and read-only. Uses captured selection for the local driver and replicated selection for observers. Unknown selection returns nil, weapon_state_unavailable; a known empty selection returns an empty table.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `Array of weapon tables, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local weapons, reason = Open77.vehicles.getActiveWeapons(seat.vehicleId)
if not weapons then print(reason); return end
for _, weapon in ipairs(weapons) do
    print("Selected mount", weapon.index, weapon.type)
end
-- An empty array means known selection with no active attached weapons.
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25069) as `LuaVehicleGetActiveWeapons`.

## getPaint

```lua
Open77.vehicles.getPaint(id)
```

`GAME`: Requires a live game instance.

Reads replicated authoritative paint for one streamed vehicle.

Requires `vehicles.read`. Returns the separate `applied` flag plus primary and secondary keyed RGB tables. It returns `nil` while the vehicle is unknown or not streamed; the value is read-only on the client.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer|string` | required | — |

Returns: `table { applied, primary, secondary }, or nil`

```lua
local paint = Open77.vehicles.getPaint(vehicleId)
if paint and paint.applied then print(paint.primary.r, paint.primary.g, paint.primary.b) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `ResourceHost.cpp` (line 25077) as `LuaVehicleGetPaint`.

## getPlayerSeat

```lua
Open77.vehicles.getPlayerSeat([playerId])
```

`GAME`: Requires a live game instance.

Reads a replicated player's canonical vehicle-seat assignment.

Client read-only API requiring `vehicles.read`. Omitting `playerId` selects the local player. The result includes `playerId`, `vehicleId`, canonical `seat`, numeric `flags`, `entering`, `exiting`, `forcedEntry`, `exitLocked`, `forcedExit`, plus the current local projection's `streamed` state and ephemeral `entity` handle. It returns nil when no assignment is currently replicated.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | optional | — |

Returns: `table { playerId, vehicleId, seat (the canonical name only: seat_front_left, seat_front_right, seat_back_left or seat_back_right -- never driver or passenger, which are accepted aliases, not reported ones), flags (integer bitfield), entering, exiting, forcedEntry, exitLocked, forcedExit (booleans); the client copy adds animatedEntry and occupants }, or nil when the player is not assigned to a vehicle`

```lua
-- Is the local player driving right now?
local seat = Open77.vehicles.getPlayerSeat()
if seat and seat.seat == "seat_front_left" then
    Open77.hud.notify("You are the driver")
end
```

Registered in `ResourceHost.cpp` (line 25078) as `LuaVehicleGetPlayerSeat`.

## getSteeringInput

```lua
Open77.vehicles.getSteeringInput(vehicleId)
```

`GAME`: Requires a live game instance.

Reads normalized vehicle steering input.

Client-only; requires vehicles.read. Pass a network vehicle ID. Returns the same -1..1 steering input as vehicles.get(id).steering, not the interior wheel angle or rolling tire phase. Returns nil, reason for invalid/missing vehicles or permission denial. See the vehicle steering guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer|string` | required | — |

Returns: `number input`, `reason on failure`

```lua
local input, reason = Open77.vehicles.getSteeringInput(vehicleId)
if input ~= nil then print(input) end
```

Registered in `ResourceHost.cpp` (line 25974) as `LuaVehicleGetSteeringInput`.

## getSteeringWheelAngle

```lua
Open77.vehicles.getSteeringWheelAngle(vehicleId)
```

`GAME`: Requires a live game instance.

Reads the actual rendered steering-wheel angle in degrees.

Client-only; requires vehicles.read and a streamed vehicle. Reads native steering_wheel and Base slots using the loaded authored rig, removing chassis orientation, column tilt and mesh offsets. Supports locally driven and replicated cars with the required slots and steering_wheel_turn bone. Unsupported rigs, including bikes without a readable wheel, return nil instead of an inferred angle. No first-frame neutral calibration. This is the visible client pose, subject to normal presentation delay, not a server-authoritative angle. Errors: invalid_vehicle_id, vehicle_not_found, vehicle_not_streamed, steering_wheel_unsupported, steering_angle_unavailable, permission_denied:vehicles.read.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer|string` | required | — |

Returns: `number degrees`, `reason on failure`

```lua
local degrees, reason = Open77.vehicles.getSteeringWheelAngle(vehicleId)
if degrees ~= nil then
    print(("Steering wheel: %.1f degrees"):format(degrees))
end
```

Registered in `ResourceHost.cpp` (line 25975) as `LuaVehicleGetSteeringWheelAngle`.

## getVisual

```lua
Open77.vehicles.getVisual(id)
```

`GAME`: Requires a live game instance.

Reads the vehicle's visual value exactly as the server published it.

Requires `vehicles.read`. Client-side and read-only. Returns the `open77.visual` value (schema v1: `v`, `rev`, `profile`, `neon`, `hide`, `show`, `attach`, `plate`, `tint`, and `na`, `lc`, `st` for the neon animation, lamp colours and stickers), or nil when the car has none -- the factory look -- or is unknown. It is what the server sent, not what this game managed to draw: `customizationAvailable` tells that. Returns nil and `vehicles_unavailable` or `permission_denied:vehicles.read` on failure.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `value`, `reason (when the first is nil)`

```lua
local visual = Open77.vehicles.getVisual(vehicleId)
if not visual then print("factory look"); return end
print("rev", visual.rev, "plate", visual.plate or "-")
for _, part in ipairs(visual.attach or {}) do print(part.key, part.model) end
```

[Vehicle customization natives](../../01%20Documentation/vehicle-customization-natives.md)

Registered in `ResourceHost.cpp` (line 27734) as `LuaVehicleGetVisual`.

## getWeapon

```lua
Open77.vehicles.getWeapon(vehicleId, mountIndex)
```

`GAME`: Requires a live game instance.

Reads one weapon mount by its stable one-based index.

Requires `vehicles.read`. Client-side, synchronous and read-only. The index comes from weapon.index, not weaponId or slotId. An unattached declared mount is returned with attached=false; invalid indices are rejected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `mountIndex` | `integer` | required | — |

Returns: `Weapon table, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local weapons = Open77.vehicles.getWeapons(seat.vehicleId)
if not weapons or not weapons[1] then return end
local weapon, reason = Open77.vehicles.getWeapon(seat.vehicleId, weapons[1].index)
if not weapon then print(reason); return end
print(weapon.index, weapon.attached, weapon.active)
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25070) as `LuaVehicleGetWeapon`.

## getWeaponAim

```lua
Open77.vehicles.getWeaponAim(vehicleId)
```

`GAME`: Requires a live game instance.

Reads the Basilisk's captured or replicated turret inputs.

Requires `vehicles.read`. Client-side, synchronous and read-only. Returns yaw, pitch and source. These are graph angles in degrees relative to the chassis, not a world aim ray. Other models or unavailable state return nil, weapon_aim_unavailable.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `Aim table, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local aim, reason = Open77.vehicles.getWeaponAim(seat.vehicleId)
if not aim then print(reason); return end
print(aim.yaw, aim.pitch, aim.source) -- degrees relative to the chassis
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25075) as `LuaVehicleGetWeaponAim`.

## getWeaponAmmo

```lua
Open77.vehicles.getWeaponAmmo(vehicleId, mountIndex)
```

`GAME`: Requires a live game instance.

Reads available native ammunition counters for a mounted weapon.

Requires `vehicles.read`. Client-side, synchronous and read-only. Returns available, source, optional magazine/capacity/total/reserve. Ammunition is native-local-driver only; observer counters are not replicated and are omitted, never invented as zero. Counters do not establish whether a mounted weapon has infinite ammo or can fire.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `mountIndex` | `integer` | required | — |

Returns: `Ammo table, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local weapons = Open77.vehicles.getWeapons(seat.vehicleId)
if not weapons or not weapons[1] then return end
local ammo, reason = Open77.vehicles.getWeaponAmmo(seat.vehicleId, weapons[1].index)
if not ammo then print(reason); return end
if ammo.available and ammo.magazine ~= nil then
    print(ammo.magazine, ammo.capacity)
else
    print("--", ammo.source) -- e.g. not_replicated for a passenger
end
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25071) as `LuaVehicleGetWeaponAmmo`.

## getWeaponCount

```lua
Open77.vehicles.getWeaponCount(vehicleId)
```

`GAME`: Requires a live game instance.

Counts weapon objects attached to a streamed vehicle.

Requires `vehicles.read`. Client-side, synchronous and read-only. Returns actual attached count, not declaredCount or activeCount. Zero means a known streamed vehicle with no resolved weapon objects.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `integer, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local count, reason = Open77.vehicles.getWeaponCount(seat.vehicleId)
if count == nil then print(reason); return end
print("Attached weapons:", count) -- zero is valid
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25074) as `LuaVehicleGetWeaponCount`.

## getWeaponModel

```lua
Open77.vehicles.getWeaponModel(record)
```

`GAME`: Requires a live game instance.

Reads the weapon mounts declared by a vehicle model.

Requires `vehicles.read`. Client-side, synchronous and read-only. This is a TweakDB declaration, not proof that all weapons are attached in an appearance. Returns record, hasWeaponMounts, declaredCount and mounts.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `Model table, or nil, reason`

```lua
local model, reason = Open77.vehicles.getWeaponModel("Vehicle.v_militech_basilisk")
if not model then print(reason); return end
print(model.record, model.declaredCount)
for _, mount in ipairs(model.mounts) do
    print(mount.index, mount.record, mount.slot, mount.type)
end
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25064) as `LuaVehicleGetWeaponModel`.

## getWeapons

```lua
Open77.vehicles.getWeapons(vehicleId)
```

`GAME`: Requires a live game instance.

Lists the weapon objects actually attached to a streamed vehicle.

Requires `vehicles.read`. Client-side, synchronous and read-only. The returned array is dense but each weapon.index preserves its one-based position in the model mount list. Do not use the array position as the mount index.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `Array of weapon tables, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local weapons, reason = Open77.vehicles.getWeapons(seat.vehicleId)
if not weapons then print(reason); return end
for _, weapon in ipairs(weapons) do
    print(weapon.index, weapon.type, weapon.slot)
end
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25068) as `LuaVehicleGetWeapons`.

## getWeaponState

```lua
Open77.vehicles.getWeaponState(vehicleId)
```

`GAME`: Requires a live game instance.

Reads one coherent mounted-weapon snapshot.

Requires `vehicles.read`. Client-side, synchronous and read-only. Uses a canonical network vehicle ID. Includes declared mounts, attached/active counts, selection mask, local-driver flag and available native ammo/heat. Local reads are cached for at most 100 ms.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `Snapshot table, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if seat then
    local state, reason = Open77.vehicles.getWeaponState(seat.vehicleId)
    if state then print(state.attachedCount, state.activeCount) end
end
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25066) as `LuaVehicleGetWeaponState`.

## getWeaponType

```lua
Open77.vehicles.getWeaponType(vehicleId, mountIndex)
```

`GAME`: Requires a live game instance.

Reads a mounted weapon's stable category.

Requires `vehicles.read`. Client-side, synchronous and read-only. Categories: machine_gun, cannon, missile, countermeasure, unknown. Native itemType, when available, is a separate field in getWeapon.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `mountIndex` | `integer` | required | — |

Returns: `Category string, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local weapons = Open77.vehicles.getWeapons(seat.vehicleId)
if not weapons or not weapons[1] then return end
local category, reason = Open77.vehicles.getWeaponType(seat.vehicleId, weapons[1].index)
if not category then print(reason); return end
print(category) -- machine_gun, cannon, missile, countermeasure or unknown
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25072) as `LuaVehicleGetWeaponType`.

## honk

```lua
Open77.vehicles.honk(id, [durationMs])
```

`GAME`: Requires a live game instance.

Alias of `Open77.vehicles.triggerHorn`.

Requires `vehicles.control` and local physics ownership. Duration is bounded to 100..2000 ms and defaults to 250 ms.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer|string` | required | — |
| `durationMs` | `integer` | optional | `250` |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Honk the car the player is simulating
RegisterKeyMapping("horn", "Horn", "H", function()
    local seat = Open77.vehicles.getPlayerSeat()
    if seat then Open77.vehicles.honk(seat.vehicleId, 400) end
end)
```

Registered in `ResourceHost.cpp` (line 25082) as `LuaVehicleTriggerHorn`.

## isArmed

```lua
Open77.vehicles.isArmed(vehicleId)
```

`GAME`: Requires a live game instance.

Checks whether a streamed vehicle actually has attached weapons.

Requires `vehicles.read`. Client-side, synchronous and read-only. Counts resolved weapon objects, not the model's inherited mount declarations. An unstreamed vehicle returns nil, vehicle_not_streamed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |

Returns: `boolean, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local armed, reason = Open77.vehicles.isArmed(seat.vehicleId)
if armed == nil then print(reason); return end
print(armed and "Attached weapons found" or "No attached weapons")
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25067) as `LuaVehicleIsArmed`.

## isAutoSeatSwitchEnabled

```lua
Open77.vehicles.isAutoSeatSwitchEnabled()
```

`GAME`: Requires a live game instance.

Reads the effective local automatic driver-takeover policy.

Requires vehicles.read. Combines client resource vetoes and the authoritative server policy. A true result permits the normal vanilla takeover when its native conditions are met. Returns nil and a reason if no active session can be queried.

Returns: `boolean, or nil`, `reason`

```lua
local enabled, reason = Open77.vehicles.isAutoSeatSwitchEnabled()
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#client-api)

Registered in `ResourceHost.cpp` (line 25247) as `LuaVehicleAutoSeatSwitchEnabled`.

## isDoorOpen

```lua
Open77.vehicles.isDoorOpen(id, door)
```

`GAME`: Requires a live game instance.

Reads a canonical vehicle door, trunk, or hood state.

Client read-only API requiring `vehicles.read`. Accepts index 0..5, a standard camelCase/snake_case name, or a `Open77.vehicles.doors` constant. This reads the replicated reversible opening mask.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `door` | `integer|string` | required | — |

Returns: `boolean, or nil when denied, unavailable, or unknown`

```lua
-- Warn the driver when the trunk is open while moving
local seat = Open77.vehicles.getPlayerSeat()
if seat and Open77.vehicles.isDoorOpen(seat.vehicleId, "trunk") then
    Open77.hud.notify("Trunk is open")
end
```

Registered in `ResourceHost.cpp` (line 25083) as `LuaVehicleIsDoorOpen`.

## isLocked

```lua
Open77.vehicles.isLocked(id)
```

`GAME`: Requires a live game instance.

Reads a vehicle's replicated entry-lock state.

Requires `vehicles.read`. Returns the durable server-authoritative lock for a streamed vehicle. This is the vehicle entry lock, not the per-occupant exit lock and not the client-side grid control lock.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer|string` | required | — |

Returns: `boolean, or nil when the vehicle is unknown`

```lua
-- Show a lock icon on the nearest car
local car = Open77.vehicles.closest(5.0)
if car then
    Open77.hud.notify(Open77.vehicles.isLocked(car.id) and "Locked" or "Unlocked", { replace = true })
end
```

Registered in `ResourceHost.cpp` (line 25080) as `LuaVehicleIsLocked`.

## isPlayerExitLocked

```lua
Open77.vehicles.isPlayerExitLocked([playerId])
```

`GAME`: Requires a live game instance.

Reports the replicated exit-lock policy for a player.

Client read-only API requiring `vehicles.read`. Omitting `playerId` selects the local player. Returns nil when the player has no replicated seat assignment; otherwise returns the canonical `exitLocked` boolean.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | optional | — |

Returns: `boolean, or nil`

```lua
-- Tell the player why F does nothing
if Open77.vehicles.isPlayerExitLocked() then
    Open77.hud.notify("You cannot leave this vehicle")
end
```

Registered in `ResourceHost.cpp` (line 25079) as `LuaVehicleIsPlayerExitLocked`.

## isWeaponActive

```lua
Open77.vehicles.isWeaponActive(vehicleId, mountIndex)
```

`GAME`: Requires a live game instance.

Checks whether the driver selected a particular attached mount.

Requires `vehicles.read`. Client-side, synchronous and read-only. Selection is not a firing event or a guarantee of native mesh deployment. Unknown selection is an explicit error.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer` | required | — |
| `mountIndex` | `integer` | required | — |

Returns: `boolean, or nil, reason`

```lua
local seat = Open77.vehicles.getPlayerSeat()
if not seat then return end
local weapons = Open77.vehicles.getWeapons(seat.vehicleId)
if not weapons or not weapons[1] then return end
local active, reason = Open77.vehicles.isWeaponActive(seat.vehicleId, weapons[1].index)
if active == nil then print(reason); return end
print(active and "Selected" or "Not selected") -- not a firing flag
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25073) as `LuaVehicleIsWeaponActive`.

## isWindowOpen

```lua
Open77.vehicles.isWindowOpen(id, window)
```

`GAME`: Requires a live game instance.

Reads a canonical openable side-window state.

Client read-only API requiring `vehicles.read`. Windows use indexes 0..3 and names `frontLeft`, `frontRight`, `backLeft`, and `backRight`. This does not report shattered glass; inspect `brokenGlass` or `damage.glass` in the vehicle snapshot for that.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |
| `window` | `integer|string` | required | — |

Returns: `boolean, or nil when denied, unavailable, or unknown`

```lua
-- Rain sound is louder with the driver's window open
local seat = Open77.vehicles.getPlayerSeat()
if seat and Open77.vehicles.isWindowOpen(seat.vehicleId, "frontLeft") then
    SetSoundVolume(rainLoop, 1.0)
end
```

Registered in `ResourceHost.cpp` (line 25084) as `LuaVehicleIsWindowOpen`.

## modelHasWeaponMounts

```lua
Open77.vehicles.modelHasWeaponMounts(record)
```

`GAME`: Requires a live game instance.

Checks whether a vehicle model declares any weapon mounts.

Requires `vehicles.read`. Client-side, synchronous and read-only. A valid unarmed model returns false; an unknown model returns nil. Inherited mounts do not guarantee a usable armed vehicle.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `boolean, or nil, reason`

```lua
local declared, reason = Open77.vehicles.modelHasWeaponMounts("Vehicle.v_militech_basilisk")
if declared == nil then print(reason); return end
print(declared and "Model declares mounts" or "No declared mounts")
-- Declarations alone do not prove weapons are attached to a live vehicle.
```

[Armed vehicle guide](../../01%20Documentation/vehicle-weapons.md#function-reference)

Registered in `ResourceHost.cpp` (line 25065) as `LuaVehicleModelHasWeaponMounts`.

## nearby

```lua
Open77.vehicles.nearby(radius, [options])
```

`SHARED`: Available without a live game instance.

Every replicated vehicle within a radius, nearest first.

Requires `vehicles.read`. Centred on the local player unless `origin` is given. `streamed = true` keeps only the ones with a REDengine projection, `occupied` filters on whether anyone is aboard, `limit` caps the list. Ties are broken by vehicle id so the order does not flicker between frames. Each entry is `{ id, entity, engineEntity, record, distance, streamed, locked, occupants, position }`, and `position` is a plain `{ x, y, z }` -- which is what an Open77 vector3 is, so `#(a.position - b.position)` works on it without a conversion. **Only server-spawned vehicles are in the registry.** A player standing next to a vanilla traffic car is standing next to something REDengine population spawned on their own client: it has no canonical id and this call cannot see it. Adopting vanilla traffic is a separate capability that does not exist yet.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `radius` | `number` | required | — |
| `options` | `table { origin?, streamed?, occupied?, limit? }` | optional | — |

Returns: `array of entries, or nil`, `reason: permission_denied:vehicles.read, invalid_radius, invalid_origin, invalid_limit, no_session, no_local_position`

```lua
-- List the server vehicles within 30 m, nearest first
for _, car in ipairs(Open77.vehicles.nearby(30, { limit = 5 })) do
    print(("#%d %s at %.1f m, %d aboard"):format(car.id, car.record, car.distance, car.occupants))
end
```

Registered in `ResourceHost.cpp` (line 25095) as `LuaVehiclesNearby`.

## occupantInSeat

```lua
Open77.vehicles.occupantInSeat(id, seat)
```

`GAME`: Requires a live game instance.

Which player is in one seat of a replicated vehicle.

Requires `vehicles.read`. The `GetPedInVehicleSeat` of FiveM, answering an Open77 player id. A plain `nil` means the seat is free; a `nil` with a reason means the question could not be answered.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `seat` | `integer|string` | required | — |

Returns: `playerId, or nil`, `reason: permission_denied:vehicles.read, invalid_vehicle_id, invalid_seat, vehicle_not_found`

```lua
-- Who is sitting next to the driver?
local seat = Open77.vehicles.getPlayerSeat()
if seat then
    local passenger = Open77.vehicles.occupantInSeat(seat.vehicleId, "frontPassenger")
    if passenger then print("passenger is player", passenger) end
end
```

Registered in `ResourceHost.cpp` (line 25100) as `LuaVehicleOccupantInSeat`.

## ratedTopSpeed

```lua
Open77.vehicles.ratedTopSpeed(vehicleId)
```

`GAME`: Requires a live game instance.

Gearing rating of a vehicle's TweakDB record, in km/h.

Requires `vehicles.read`. Returns the speed at which the record's top gear reaches its rated maximum engine RPM, reached through `vehEngineData`. Use it to rank a roster and to pick a ceiling no car has to be slowed much to meet. It is **not** the measured terminal velocity, which torque and aerodynamic drag carry higher. Returns `nil` when the record cannot be resolved.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer|string` | required | — |

Returns: `number or nil`

```lua
-- Pick a fair race cap: 10 km/h under the slowest car in the roster
local cap = math.huge
for _, record in ipairs(roster) do
    cap = math.min(cap, Open77.vehicles.ratedTopSpeed(record) or cap)
end
Open77.vehicles.setPerformanceClass(roster[1], { topSpeedKph = cap - 10 })
```

Registered in `ResourceHost.cpp` (line 25094) as `LuaVehicleRatedTopSpeed`.

## releaseAdopted

```lua
Open77.vehicles.releaseAdopted(engineEntity)
```

`GAME`: Requires a live game instance.

Hands one engine-spawned chassis back to the population system.

Requires `vehicles.adopt`, which is its own permission because this deletes something from the local world rather than reading it. It is the second half of adoption and must run ONLY after the server has accepted the proposal: the canonical vehicle replaces this car, and removing it first deletes a traffic car for nothing. Removal goes through `gameIPopulationSystem::RemoveEntity`, the asynchronous teardown path the world sanitizer already runs every second in production -- never `entEntity.Dispose` or any of the lifecycle functions the debug bridge blocks, which tear an object down while save and AI systems still hold references. `not_offered` means this chassis was not returned by an `adoptable()` or `describeAdoptable()` call in the last minute: an engine entity id is just a number, so being offered is the capability, and without that rule a resource could name an NPC or a door. `managed_vehicle` refuses a car the server owns, `player_mounted` refuses the one the player is sitting in, and a chassis the engine already removed answers true, because the caller wanted it gone and it is.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `engineEntity` | `string|integer` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The server accepted the proposal: hand the vanilla chassis back
RegisterNetEvent("garage:adopted", function(engineEntity, vehicleId)
    Open77.vehicles.releaseAdopted(engineEntity)
    Open77.hud.notify(("Vehicle #%d is yours"):format(vehicleId))
end)
```

Registered in `ResourceHost.cpp` (line 25103) as `LuaVehicleReleaseAdopted`.

## seatFree

```lua
Open77.vehicles.seatFree(id, seat)
```

`SHARED`: Available without a live game instance.

Whether one seat of a replicated vehicle is free.

Requires `vehicles.read`. Read from the replicated occupancy ledger, so it agrees with the server's own answer rather than with the local projection. An occupant whose entry animation is still running counts as present: the seat is reserved and a second entry into it is refused.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `seat` | `integer|string` | required | — |

Returns: `boolean, or nil`, `reason: permission_denied:vehicles.read, invalid_vehicle_id, invalid_seat, vehicle_not_found`

```lua
-- Only offer the "enter as passenger" prompt when the seat is free
local car = Open77.vehicles.closest(4.0)
if car and Open77.vehicles.seatFree(car.id, "frontPassenger") then
    Open77.hud.notify("Press E to get in", { replace = true })
end
```

Registered in `ResourceHost.cpp` (line 25099) as `LuaVehicleSeatFree`.

## seatSwitchState

```lua
Open77.vehicles.seatSwitchState()
```

`GAME`: Requires a live game instance.

Reads the local canonical seat transition and automatic policy.

Requires vehicles.read. Returns active and autoSwitchEnabled, plus vehicleId/fromSeat while seated and id/toSeat while switching. IDs are strings. The canonical source stays occupied until native completion. Query on resource start to recover a transition already in progress; nil plus reason means the session or backend is unavailable.

Returns: `state table, or nil`, `reason`

```lua
local state, reason = Open77.vehicles.seatSwitchState()
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#client-api)

Registered in `ResourceHost.cpp` (line 25245) as `LuaVehicleSeatSwitchState`.

## setAutoSeatSwitchEnabled

```lua
Open77.vehicles.setAutoSeatSwitchEnabled(enabled)
```

`GAME`: Requires a live game instance.

Controls this client resource's automatic driver-takeover veto.

Requires vehicles.control. false blocks automatic front-passenger takeover of an empty driver seat. true releases only this resource's veto; another resource or the server can still disable takeover. Explicit switches remain available. The claim is released when its resource stops.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, otherwise false`, `reason`

```lua
Open77.vehicles.setAutoSeatSwitchEnabled(false)
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#client-api)

Registered in `ResourceHost.cpp` (line 25246) as `LuaVehicleSetAutoSeatSwitchEnabled`.

## setControlLock

```lua
Open77.vehicles.setControlLock(vehicleId, locked)
```

`GAME`: Requires a live game instance.

Locks or releases a stopped vehicle's driving inputs.

Requires `vehicles.performance`. While locked, the native drive update consumes zero acceleration, zero brake/reverse and no burnout. Releasing restores REDengine's current input snapshot on the next frame; the API never teleports or clamps velocity. Use it for authoritative starting grids after the vehicle is stationary. Like the performance governor, this is client-side gamemode enforcement, not an anti-cheat boundary.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer|string` | required | — |
| `locked` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Starting grid: freeze the inputs, then release them on GO
RegisterNetEvent("race:grid", function(vehicleId)
    Open77.vehicles.setControlLock(vehicleId, true)
end)
RegisterNetEvent("race:go", function(vehicleId)
    Open77.vehicles.setControlLock(vehicleId, false)
end)
```

Registered in `ResourceHost.cpp` (line 25092) as `LuaVehicleSetControlLock`.

## setFrozen

```lua
Open77.vehicles.setFrozen(vehicleId, frozen)
```

`GAME`: Requires a live game instance.

Freezes or wakes a vehicle's native chassis physics.

Requires `vehicles.performance`. Disables the verified REDengine vehicle physics group without changing the authority lease, player-controlled state or mounted workspot. Unlike `setControlLock`, this stops inertia, gravity and external impulses without teleporting the vehicle. Releasing restores the mask and calls the native physics wake-up path. Pair both calls for a starting grid: lock inputs then freeze; on GO thaw then unlock inputs. Unknown or not-yet-streamed vehicles fail so the resource can retry.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer|string` | required | — |
| `frozen` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Starting grid: lock the inputs and freeze the chassis until GO
RegisterNetEvent("race:grid", function(vehicleId)
    Open77.vehicles.setControlLock(vehicleId, true)
    Open77.vehicles.setFrozen(vehicleId, true)
end)
RegisterNetEvent("race:go", function(vehicleId)
    Open77.vehicles.setFrozen(vehicleId, false)
    Open77.vehicles.setControlLock(vehicleId, false)
end)
```

Registered in `ResourceHost.cpp` (line 25093) as `LuaVehicleSetFrozen`.

## setPerformance

```lua
Open77.vehicles.setPerformance(vehicleId, profile)
```

`GAME`: Requires a live game instance.

Caps one vehicle's top speed and pickup on this client.

Requires `vehicles.performance`. Cyberpunk 2077 2.31 has no speed setter, so the ceiling is applied by withdrawing driver throttle where the drive update reads it, tapered across a band so the top of the acceleration curve flattens instead of hitting a wall. The profile table takes `topSpeedKph` (0 leaves the top end alone), `taperKph` (default 12, values under 3 are refused because a narrower band makes the limiter hunt) and `accelerationScale` in (0, 1]. Precedence is instance, then record class, then default. **This is balance, not anti-cheat**: the client that simulates the vehicle applies it, and a modified client can decline to. Cleared automatically on despawn. This is the client-local form and it still works unchanged; the server-authoritative form is `Open77.vehicles.setPerformance` on the server, which takes the same three numbers, replicates them to every viewer and does not need the client to cooperate in deciding them. The two are separate policies over the same native governor, which resolves instance, then record class, then default -- so a client-local instance profile on the same vehicle takes precedence over the replicated one.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `vehicleId` | `integer|string` | required | — |
| `profile` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- A 60 km/h limit inside the city centre, applied to the car the player drives
AddEventHandler("citycentre:entered", function()
    local seat = Open77.vehicles.getPlayerSeat()
    if seat then Open77.vehicles.setPerformance(seat.vehicleId, { topSpeedKph = 60, taperKph = 10 }) end
end)
```

Registered in `ResourceHost.cpp` (line 25087) as `LuaVehicleSetPerformance`.

## setPerformanceClass

```lua
Open77.vehicles.setPerformanceClass(record, profile)
```

`GAME`: Requires a live game instance.

Caps every present and future vehicle spawned from one TweakDB record.

Requires `vehicles.performance`. This is the shape a roster wants: name the cars offered and the ceiling they share once, rather than tracking instances. Same profile table as `setPerformance`. A record class is overridden by an explicit per-instance profile.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |
| `profile` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Every Quadra in the race is capped at 180 km/h
Open77.vehicles.setPerformanceClass("Vehicle.v_sport2_quadra_turbo_r", {
    topSpeedKph = 180,
    accelerationScale = 0.8,
})
```

Registered in `ResourceHost.cpp` (line 25089) as `LuaVehicleSetPerformanceClass`.

## switchSeat

```lua
Open77.vehicles.switchSeat(seat)
```

`GAME`: Requires a live game instance.

Requests an animated move to another seat for the local player.

Requires vehicles.control and protocol 1.44. Submits a server-authorized in-cabin move. A true return means queued; use open77:vehicleSeatSwitchDecision and the Started/Completed/Failed events for its outcome. Both source and destination stay reserved until completion. Refuses occupied or unsupported seats, exit locks and overlapping requests.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `seat` | `string|integer` | required | — |

Returns: `true when submitted, otherwise false`, `reason`

```lua
local ok, reason = Open77.vehicles.switchSeat('rearRight')
```

[Changing seats guide](../../01%20Documentation/vehicle-seat-switching.md#client-api)

Registered in `ResourceHost.cpp` (line 25243) as `LuaVehicleSwitchSeat`.

## taskPlayerEnterVehicle

```lua
Open77.vehicles.taskPlayerEnterVehicle(playerId, vehicleId, seat)
```

`GAME`: Requires a live game instance.

Presents an authorized replicated occupant with the staged entry path.

Requires `vehicles.presentation` and an already-authorized occupancy tuple. Uses the guarded door/workspot presentation for a live remote entry and retains a bounded instant-warp fallback. This is presentation only and never grants a seat or changes physics authority.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `vehicleId` | `integer|string` | required | — |
| `seat` | `string|integer` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Play the door animation for a remote player whose seat the server already granted
RegisterNetEvent("garage:showEntry", function(playerId, vehicleId, seat)
    Open77.vehicles.taskPlayerEnterVehicle(playerId, vehicleId, seat)
end)
```

Registered in `ResourceHost.cpp` (line 25086) as `LuaVehicleTaskPlayerEnter`.

## triggerHorn

```lua
Open77.vehicles.triggerHorn(id, [durationMs])
```

`GAME`: Requires a live game instance.

Sounds the horn of a locally simulated vehicle.

Requires `vehicles.control`. The vehicle must be streamed and this client must be its current physics owner. Duration is bounded to 100..2000 ms. For an arbitrary vehicle, call the authoritative server API so it can route the request to the current owner.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer|string` | required | — |
| `durationMs` | `integer` | optional | `250` |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Double honk from the car this client simulates
CreateThread(function()
    Open77.vehicles.triggerHorn(vehicleId, 200)
    Wait(400)
    Open77.vehicles.triggerHorn(vehicleId, 200)
end)
```

Registered in `ResourceHost.cpp` (line 25081) as `LuaVehicleTriggerHorn`.

## warpPlayerIntoVehicle

```lua
Open77.vehicles.warpPlayerIntoVehicle(playerId, vehicleId, seat)
```

`GAME`: Requires a live game instance.

Presents a replicated player instantly in an authoritative vehicle seat.

Requires `vehicles.presentation`. This is the client meaning of the name: a guarded visual operation for an existing server-owned vehicle and occupancy assignment. It does not claim the seat or physics authority. The exact `(player, vehicle, seat)` tuple must already exist in the replicated ledger or the call fails with `occupancy_mismatch`. The server runtime exposes an identically named authoritative assignment API; the two Lua states are distinct.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `vehicleId` | `integer|string` | required | — |
| `seat` | `string|integer` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Instantly show a remote occupant in the seat the server assigned
RegisterNetEvent("garage:showWarp", function(playerId, vehicleId, seat)
    Open77.vehicles.warpPlayerIntoVehicle(playerId, vehicleId, seat)
end)
```

Registered in `ResourceHost.cpp` (line 25085) as `LuaVehicleWarpPlayer`.

## whenStreamed

```lua
Open77.vehicles.whenStreamed(id, [timeoutMs])
```

`GAME`: Requires a live game instance.

A promise that resolves once this client's projection of a vehicle is attached.

Requires `vehicles.read`. The replacement for FiveM's `RequestModel` / `HasModelLoaded` loops: streaming is implicit here, so the only honest wait is "does this client have the body yet". Resolves with the `Open77.vehicles.get(id)` snapshot the moment it reports `streamed == true` -- before the call returns when it already does -- and rejects with `timeout` after `timeoutMs` (default 15000, clamped to 1..120000). Listens for `open77:vehicleCreated` (the create arriving; the engine spawn follows, measured up to 3.7 s later) and polls the snapshot every 100 ms until it is attached. The one-shot handler is removed on resolve, on timeout and when the resource stops. `nil, reason` only for a bad id, a bad timeout or a missing permission; an id this client has never seen is waited for and times out, because "not here yet" and "not anywhere" look the same from a client.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `vehicle id` | required | — |
| `timeoutMs` | `number` | optional | — |

Returns: `Promise, or nil`, `reason: invalid_vehicle_id, invalid_timeout, permission_denied:vehicles.read, promise_unavailable`

```lua
-- Attach the taxi sign only once the car is really here
RegisterNetEvent("taxi:assigned", function(vehicleId)
    CreateThread(function()
        local pending, reason = Open77.vehicles.whenStreamed(vehicleId, 10000)
        if not pending then return print("cannot wait: " .. tostring(reason)) end
        local car, why = pending:await()
        if not car then return print("taxi never streamed in: " .. tostring(why)) end
        Open77.blips.create({ entity = car.entity, sprite = "taxi", label = "Your taxi" })
    end)
end)
```

Registered in `ResourceHost.cpp` (line 23323) as `LuaPrelude`.
