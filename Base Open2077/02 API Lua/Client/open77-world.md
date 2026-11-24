# Open77.world (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-world.md) · [Page web](https://open2077.net/docs/api/client/open77-world)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

16 functions.

## clearDeviceInteraction

```lua
Open77.world.clearDeviceInteraction(target)
```

`GAME`: Requires a live game instance.

Drops this resource's device-interaction policy for one target.

Removes the calling resource's policy for that engine entity or class name, which is not the same as setDeviceInteractionEnabled(target, true): an explicit allow is still a policy this resource holds, while a cleared key is one it no longer has an opinion about. Another resource's refusal on the same target is untouched either way -- every active policy must allow for an interaction to happen. Clearing a key that was never set is not an error, so a resource that tidies up on every path can do so unconditionally. Requires the world.devices permission.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer engine entity id, or string class name / 0x id` | required | — |

Returns: `true, otherwise false`, `reason on failure`

```lua
-- The custom vending menu closes: give the vanilla prompt back
AddEventHandler("vending:closed", function()
    Open77.world.clearDeviceInteraction("VendingMachine")
end)
```

Registered in `ResourceHost.cpp` (line 25224) as `LuaWorldSetDeviceInteraction`.

## deviceInteractionEnabled

```lua
Open77.world.deviceInteractionEnabled(target)
```

`GAME`: Requires a live game instance.

Whether a vanilla device prompt is currently allowed, across every resource.

The EFFECTIVE verdict, combining every running resource's policies -- not the calling resource's own claim. A resource that released its policy still reads false while another resource holds one, exactly as Open77.input.isActionBlocked answers for every holder. The same target spelling as setDeviceInteractionEnabled; a decimal id and its 0x form resolve to the same device. Requires the world.devices permission, deliberately the same string as the write: a resource that may not change a policy has no business enumerating what other resources set.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer engine entity id, or string class name / 0x id` | required | — |

Returns: `boolean, or nil`, `reason on failure`

```lua
-- Before drawing our own prompt, check nobody else already refused the device
if Open77.world.deviceInteractionEnabled(machine.engineEntity) then
    Open77.hud.notify("Press E to hack the machine", { replace = true })
end
```

Registered in `ResourceHost.cpp` (line 25226) as `LuaWorldDeviceInteractionEnabled`.

## deviceInteractionPolicies

```lua
Open77.world.deviceInteractionPolicies()
```

`GAME`: Requires a live game instance.

Every standing device-interaction policy and who holds it.

One row per policy: owner (the resource that set it), enabled, and exactly one of engineEntity or className. Ordered by owner then by key. This is how an admin resource proves a claim really came back, the same role Open77.input.blocks() plays for input claims. Requires the world.devices permission.

Returns: `array of { owner, engineEntity | className, enabled }, or nil`, `reason on failure`

```lua
-- Debug: list who blocked which device
for _, policy in ipairs(Open77.world.deviceInteractionPolicies() or {}) do
    print(policy.owner, policy.className or policy.engineEntity, policy.enabled)
end
```

Registered in `ResourceHost.cpp` (line 25227) as `LuaWorldDeviceInteractionPolicies`.

## district

```lua
Open77.world.district([position])
```

`GAME`: Requires a live game instance.

The district the local player is standing in.

Reads the engine's own district for the player: `Watson`, `Dogtown`, `Arroyo`, and a finer `subDistrict` such as `ClairesGarage` where one is authored. The names are the `gamedataDistrict` enum's, so they are stable identifiers rather than display text; `localized` carries the record's own display name. `position` is optional and is accepted only when it is the local player's own position (within 25 m); any other position is refused with `position_not_local`. That is not a limitation of Open77: Cyberpunk resolves a district by trigger volume, the volume fires when the PLAYER crosses it, and the geometry is baked sector data with no getter, so nothing in the engine maps an arbitrary coordinate to a district and a guessed answer would be worse than a refusal. There is deliberately no `Open77.world.street`: the engine has no street layer, only street names painted onto sign meshes as UI text, so no coordinate has a street. To learn another player's district, listen for `open77:districtChanged` on their client and send it to the server. Requires `world.query`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y, z } (the local player's own)` | optional | — |

Returns: `table { district, subDistrict?, localized? } -- subDistrict is absent, not empty, when there is none, or nil`, `reason: permission_denied:world.query, invalid_position, position_not_local, district_unavailable, world_unavailable_on_this_host`

```lua
-- Show the district name when the player crosses into a new one
local last
SetTick(function()
    local place = Open77.world.district()
    if place and place.district ~= last then
        last = place.district
        Open77.hud.notify("Now entering " .. (place.localized or place.district))
    end
end)
```

Registered in `ResourceHost.cpp` (line 25215) as `LuaWorldDistrict`.

## entityGeometry

```lua
Open77.world.entityGeometry(entity)
```

`GAME`: Requires a live game instance.

Position, facing, orientation and loaded mesh bounds of one entity, for geometry built on top of it.

The engine-side read PolyZone's entity zones are built on. Takes an engine entity id (a number, the `engineEntity` a `nearby` or raycast row carries) or a typed reference `{ kind = "player" | "vehicle" | "npc" | "entity", id = <Open77 id> }`, and answers `{ attached, entity, position, forward, orientation = {x, y, z, w}, bounds?, head?, damage? }`. `bounds` is `{ min, max, source = "loaded_mesh_bounds" }` in world space and is only present while the mesh is streamed in; `head` (puppets) and `damage` (`{ health, revision, attacker?, weapon?, isMelee }`) are likewise omitted rather than fabricated when the engine has nothing to say. Requires `world.query`.

Fails with `invalid_entity_kind`, `invalid_entity_id`, `entity_geometry_unavailable`, or the entity resolver's own reason when the id names nothing attached.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `number | table` | required | — |

Returns: `table { attached, entity, position, forward, orientation, bounds?, head?, damage? }, or nil`, `reason on failure`

```lua
-- Is the local player inside the footprint of the nearest car?
local hits = Open77.world.nearby(15, "other")
for _, hit in ipairs(hits or {}) do
    if hit.family == "vehicle" then
        local geometry = Open77.world.entityGeometry(hit.engineEntity)
        if geometry and geometry.bounds then
            local x, y = Open77.character.position()
            local b = geometry.bounds
            if x >= b.min.x and x <= b.max.x and y >= b.min.y and y <= b.max.y then
                Open77.hud.notify("Step away from the " .. hit.className)
            end
        end
        break
    end
end
```

Registered in `ResourceHost.cpp` (line 25209) as `LuaEntityGeometry`.

## getTimeScale

```lua
Open77.world.getTimeScale()
```

`GAME`: Requires a live game instance.

The engine's effective time scale, its simulation clock, and every live claim.

`scale` is the engine's own effective dilation (`GetActiveTimeDilation`), which is what the player is experiencing; it can differ from every claim while a vanilla producer (a deflect, a menu) holds one of its own or while a claim is easing. `simTime` is the engine's game clock in game seconds (`GetGameTimeStamp`): it runs at the day-length multiplier (8x real on the vanilla day, measured 7.97) and dilates with `scale`, so a slow-motion is measured rather than trusted as the ratio of two rates -- its rate against `GetGameTimer()` (the steady clock, which never slows) during the hold over its rate at real time. The rate against the wall clock alone is the day length, not the dilation. `claims` lists every live claim as `{ owner, scale, applied, remainingMs?, phase }`, `phase` being `easingIn`, `holding` or `easingOut` and `remainingMs` absent for an open-ended hold; a server-replicated beat appears under the reserved owner `open77:server`. Ungated.

Returns: `table { scale, engineActive, simTime, claims }`, `nil, reason when no game backend is attached`

```lua
-- Measure a slow-motion instead of trusting it: the rate of the game clock
-- during the hold over its rate at real time (the clock itself runs at the
-- day length, 8x real on the vanilla day).
local function clockRate()
    local view = Open77.world.getTimeScale()
    local sim0, wall0 = view.simTime, GetGameTimer()
    Wait(1000)
    view = Open77.world.getTimeScale()
    return (view.simTime - sim0) / ((GetGameTimer() - wall0) / 1000)
end

CreateThread(function()
    Open77.world.setTimeScale(0.5, { durationMs = 2000 })
    local held = clockRate()
    Wait(1500)
    local real = clockRate()
    print(("the hold ran at %.2f of real time"):format(held / real)) -- about 0.50
end)
```

Registered in `ResourceHost.cpp` (line 25231) as `LuaWorldGetTimeScale`.

## groundZ

```lua
Open77.world.groundZ(x, y, [fromZ])
```

`GAME`: Requires a live game instance.

Height of the static ground below a point, by a downward ray.

Casts a `Static`-only ray straight down from `(x, y, fromZ)` over one kilometre and returns the Z of the first surface. Static only on purpose: a parked car or a body is not the ground. The default start height clears every rooftop in Night City. Returns nil when nothing was hit (open water, outside the world). Requires `world.query`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `x` | `number` | required | — |
| `y` | `number` | required | — |
| `fromZ` | `number (default 400)` | optional | — |

Returns: `z (number), or nil when nothing is below`, `reason on failure`

```lua
-- Place a marker on the ground under a map-picked point
local z = Open77.world.groundZ(-1442.2, 127.4)
if z then
    Open77.markers.create({ position = { x = -1442.2, y = 127.4, z = z }, radius = 1.0 })
end
```

Registered in `ResourceHost.cpp` (line 25207) as `LuaWorldGroundZ`.

## isBurning

```lua
Open77.world.isBurning([target])
```

`GAME`: Requires a live game instance.

Reads an entity's native Burning status effect.

Requires `world.query`. Reads the resolved entity's native Burning status effect. Omit target or pass 0 for the local player; accepts a receiver-local entity integer, `{ playerId }`, `{ npcId }`, `{ vehicleId }`, `{ propId }`, or `{ engineEntity, radius? }`. Canonical IDs take priority; an unavailable canonical target never falls back to a different entity. Preserve engine IDs as strings. `false` means no native Burning status; `nil, reason` means unavailable. Cosmetic flames, wreck visuals and nearby fires are not Burning status. No implicit replication or server authority. See [burning status](../../01%20Documentation/world-queries.md#burning-status).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer|table` | optional | — |

Returns: `boolean, or nil`, `reason`

```lua
local burning, reason = Open77.world.isBurning({ playerId = otherPlayerId })
if burning ~= nil then print(burning) end
```

Registered in `ResourceHost.cpp` (line 5949) as `LuaWorldIsBurning`.

## nearby

```lua
Open77.world.nearby(radius, [filter])
```

`GAME`: Requires a live game instance.

Every object within a radius of the local player, nearest first.

Exposes `Api::WorldQuery::Around`, the engine's own targeting search. It is centred on the local player because the search takes a source object, not a point: to look around somewhere else, stand there. `filter` narrows the object classes: a name, a list of names or a raw mask. Names: `player`, `puppet`, `sensor`, `device`, `other` (classes), `friendly`, `hostile`, `neutral`, `alive`, `dead`, `turnedOn`, `turnedOff`, `quickHackable` (attitude and state bits, which widen a search rather than narrow it). Requires `world.query`.

Every row answers two separate questions. **What is it** -- `kind`, from the same classifier the raycast row uses, refined by ownership: an Open77-owned body is `npc`, a vanilla crowd body is `populationNpc`, an Open77 car is `vehicle` and a traffic car is `trafficVehicle`. `family` is the unrefined answer (`npc`, `vehicle`, `device`, `door`, `player`, `weapon`, `item`, `object`, `unknown`) for a caller that means "is this a ped at all".

**Whose is it** -- exactly one of `playerId`, `vehicleId` or `npcId` when Open77 owns the entity, and none of the three when it does not. A row with no identity is local scenery: nothing replicates it, every client sees a different one, and it can be gone next frame. `Open77.character.state({ engineEntity = row.engineEntity })` reads such a body, read-only.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `radius` | `number (metres, 0 < r <= 1000)` | required | — |
| `filter` | `string | table of strings | number mask` | optional | — |

Returns: `array of table { engineEntity, entity identity (playerId | vehicleId | npcId), kind, family, className, position { x, y, z }, distance }`, `reason on failure`

```lua
-- Count hostile NPCs within 30 m for a "danger" indicator
local hostiles = Open77.world.nearby(30, { "puppet", "hostile" }) or {}
hud.danger = #hostiles > 0
```

Registered in `ResourceHost.cpp` (line 25208) as `LuaWorldNearby`.

## nearest

```lua
Open77.world.nearest(radius, [filter])
```

`GAME`: Requires a live game instance.

The closest object within a radius of the local player, or nil.

Same search, same filter vocabulary and the same row shape as `Open77.world.nearby` -- including `kind`, `family` and the single owner identity -- returning only the first hit. Requires `world.query`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `radius` | `number (metres, 0 < r <= 1000)` | required | — |
| `filter` | `string | table of strings | number mask` | optional | — |

Returns: `table { engineEntity, entity identity (playerId | vehicleId | npcId), kind, family, className, position { x, y, z }, distance }, or nil`, `reason on failure`

```lua
-- Press E on the nearest device (ATM, vending machine, terminal) within 3 m
RegisterKeyMapping("use_device", "Use device", "E", function()
    local device = Open77.world.nearest(3, "device")
    if device then TriggerServerEvent("devices:use", device.className, device.engineEntity) end
end)
```

Registered in `ResourceHost.cpp` (line 25211) as `LuaWorldNearest`.

## population

```lua
Open77.world.population()
```

`GAME`: Requires a live game instance.

The ambient population policy the server set for this client's routing bucket.

Read-only, and there is no client setter on purpose: density is a server-load decision that has to be the same for every client in a bucket, so there is nothing here to race the server with. `Open77.world.setPopulation` on the server is the write side.

`crowd` is a genuine fraction of the vanilla pedestrian density and is applied as one. `traffic` is reported as sent but the engine offers no vehicle density modifier on 2.31, so any value above zero means vanilla traffic -- `trafficGranularity` says `binary` and `crowdGranularity` says `continuous` rather than letting a caller assume otherwise. `applied` is what the engine did with the policy (`Ok`, or a refusal name): a policy can arrive and be refused, and a caller comparing its own numbers against the street needs to see that. Requires `world.query`.

`sanitizer` is what this client's own clean-up did with the policy, cumulative since plugin load: `kept` is the unowned bodies the identity sanitizer classified and left alone because the policy allows their kind, `removedNpcs` / `removedVehicles` the ones it took out, `sweeps` how many passes ran. A resource that turned traffic on and sees `removedVehicles` climb between two reads is watching its cars vanish (release 65's defect); `kept` climbing is the street being left alone. Compare two reads rather than trusting one.

Fails with `no_population_policy` until a server has sent one. That is a different answer from empty streets and must not be read as one.

Returns: `table { bucket, crowd, traffic, police, revision, applied, crowdGranularity, trafficGranularity, sanitizer = { kept, removedNpcs, removedVehicles, sweeps } }, or nil`, `reason on failure`

```lua
-- Tell the player when they are in an empty instance
local policy = Open77.world.population()
if policy and policy.crowd == 0 and policy.traffic == 0 then
    Open77.hud.notify("Private instance: no crowd, no traffic")
end
```

Registered in `ResourceHost.cpp` (line 25210) as `LuaWorldPopulation`.

## raycast

```lua
Open77.world.raycast(from, to, [options])
```

`GAME`: Requires a live game instance.

Traces a physics ray between two world points and returns the nearest blocking surface.

Traces the segment from `from` to `to`. Requires `world.query`. Synchronous native raycast on the client game thread. Defaults to Static and Dynamic groups; the nearer hit wins. Stable 156 adds `groups`, 1..16 case-sensitive native names such as Vehicle, Static and Terrain, checked against the engine registry. It replaces static/dynamic flags; combining them fails with `collision_groups_conflict`. Unknown names return `invalid_collision_group`; malformed lists return `invalid_collision_groups`. With `entities = true`, entity identity comes from the hit proxy's actual native owner, never a nearest-object guess. Geometry without an owner has no entity; unavailable owner lookup returns `ray_entity_lookup_unavailable`. The optional entity contains engineEntity, className, kind, position, distance and a canonical playerId/vehicleId/npcId/propId when owned. `ignore` is unsupported and returns `ray_ignore_unavailable`. See [world queries](../../01%20Documentation/world-queries.md#choosing-collision-groups).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `from` | `table { x, y, z }` | required | — |
| `to` | `table { x, y, z }` | required | — |
| `options` | `table { static?, dynamic?, groups?, entities? }` | optional | — |

Returns: `table { hit, position?, normal?, material?, distance?, entity? }, or nil`, `reason`

```lua
-- Is there a wall between the player and the target point?
local from = Open77.character.state().position
local hit = Open77.world.raycast(from, target)
if hit and hit.hit and hit.distance < #(vector3(target) - vector3(from)) then
    print("blocked by", hit.material, "at", hit.distance, "m")
end
```

Registered in `ResourceHost.cpp` (line 25206) as `LuaWorldRaycast`.

## safeAreasEnabled

```lua
Open77.world.safeAreasEnabled()
```

`GAME`: Requires a live game instance.

Whether the vanilla safe-area marking is currently in effect.

Reads the effective safe-area state across every resource that claimed it through `Open77.world.setSafeAreas`: true when the staged interiors behave as vanilla (walk-only, weapons holstered), false while at least one running resource holds the marking off. Requires the `world.safeareas` permission. Returns nil plus a reason when the permission is missing or no game backend is attached.

Returns: `boolean, or nil`, `reason on failure`

```lua
-- Warn the player before they draw a weapon in a bar
if Open77.world.safeAreasEnabled() then
    Open77.hud.notify("Weapons are holstered in this area")
end
```

Registered in `ResourceHost.cpp` (line 25204) as `LuaWorldSafeAreasEnabled`.

## setDeviceInteractionEnabled

```lua
Open77.world.setDeviceInteractionEnabled(target, enabled)
```

`GAME`: Requires a live game instance.

Turns a vanilla device prompt off, so a resource can put its own there.

I3. Cyberpunk has no per-device interactive flag a mod can write, so this wraps the one scripted choke point every device controller passes through: ScriptableDeviceComponentPS::DetermineInteractionState, which is declared on gameDeviceComponentPS, overridden exactly once on the scriptable device PS, and otherwise only on ScriptedPuppetPS, which is not a device. GetActions is overridden by some sixty controller subclasses and is therefore useless as a single gate. When a policy refuses, the wrapper publishes an EMPTY choice set through the class's own PushChoicesToInteractionComponent -- the same call vanilla makes for a device with no valid actions -- so the engine clears the prompt on the layer it chose itself. A second wrapper on InteractiveDevice::OnInteractionUsed refuses the use as well, so a policy that says off means off even if a scene or a subclass reaches the action another way. target is either an engine entity id -- exactly what Open77.world.nearby reports as engineEntity, decimal or 0x -- or a device class name, matched case-insensitively against BOTH names the engine has for a device: the object class (VendingMachine) and the controller persistent state (VendingMachineControllerPS), because the caller reads the first and the hook runs on the second. A class name accepts letters, digits and underscore only: a typo must not become a wildcard that disables half the city. The claim is per resource and additive in the refusing direction -- every active policy must allow an interaction for it to happen -- and everything a resource holds is released when it stops, reloads or crashes. Deliberately NOT under world.query, which is the read gate on nearby: this is a write that changes what the player can do with the world. It does not touch quickhacks, does not power a device down, and is presentation policy on one client rather than authority. Refusals: permission_denied:world.devices, device_backend_unavailable, invalid_device_target, invalid_device_state (a missing or non-boolean second argument is refused rather than read as 'disable'), device_policy_limit.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `integer engine entity id, or string class name / 0x id` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true, otherwise false`, `reason on failure`

```lua
-- Replace every ATM's vanilla prompt with the server's banking menu
Open77.world.setDeviceInteractionEnabled("ATM", false)
```

Registered in `ResourceHost.cpp` (line 25221) as `LuaWorldSetDeviceInteraction`.

## setSafeAreas

```lua
Open77.world.setSafeAreas(enabled)
```

`GAME`: Requires a live game instance.

Turns the vanilla safe-area marking of staged interiors on or off for this session.

Vanilla marks a handful of interiors (the Afterlife, Lizzie's, Clouds) as staged, walk-only, weapons-holstered space. `setSafeAreas(false)` takes that marking off for the duration of the session, which is what makes a lobby staged in the Afterlife playable rather than a walking simulator. The claim is cooperative and owner-scoped: safe areas return once every claiming resource has released (or stopped), never because one resource asked. Requires the `world.safeareas` permission. Returns true plus the effective state across every owner, so a caller can see that another resource still holds safe areas off; false plus a reason on failure (permission_denied:world.safeareas, safe_area_backend_unavailable, invalid_safe_area_argument).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, otherwise false`, `effective state across owners, or reason on failure`

```lua
-- The deathmatch lobby is staged in the Afterlife: make it playable
AddEventHandler("onClientResourceStart", function(name)
    if name == GetCurrentResourceName() then Open77.world.setSafeAreas(false) end
end)
```

Registered in `ResourceHost.cpp` (line 25203) as `LuaWorldSetSafeAreas`.

## setTimeScale

```lua
Open77.world.setTimeScale(scale, [options])
```

`GAME`: Requires a live game instance.

Slows the local clock for a scripted beat: one claim per resource, eased, timed, released on stop.

The FiveM `SetTimeScale` shape, bound onto the engine's own `gameTimeSystem::SetTimeDilation` under a reason named after this resource (`open77:<resource>`), on the global layer only -- so the local body slows with the world, which is what a time scale means. `scale` is 0.05..1; 1 releases this resource's claim. Zero is refused: it freezes input along with the world. `options.durationMs` holds up to an hour (0, the default, holds until released or the resource stops); `options.easeMs` ramps linearly, up to ten seconds, from wherever the clock stands to `scale` and back to 1 at the end, in your milliseconds rather than an engine curve's fixed length.

**A claim, not a raw native.** A multiplayer client runs a guard that removes every dilation it can name and, after three seconds of one it cannot, exempts the player from it. A claim registered here tells that guard to stand down for exactly as long as the beat is live. Two resources holding claims are two entries the engine composes by its own rule (the slower wins); nothing here arbitrates between them. Every exit -- deadline, `setTimeScale(1)`, stop, reload, session end, plugin unload -- unsets the entry.

**Not a combat mechanic.** The server and every client outside the beat run at real time; a shot at a slowed client is adjudicated in real time. The server half, `Open77.world.setTimeScale(bucket, ...)`, replicates one beat to a whole bucket; the reflex overdrive deliberately touches no clock. Requires `world.timescale`. Reasons: `permission_denied:world.timescale`, `invalid_scale`, `invalid_options`, `invalid_timescale_option:<name>`, `timescale_unavailable_on_this_host`, `native_unavailable`, `player_unavailable` (the world has no local player yet -- a resource's first tick on a fresh client; the dilation layer does not exist before the player does, and writing it then took a client down).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `scale` | `number (0.05..1; 1 releases)` | required | — |
| `options` | `table { durationMs? 0..3600000, easeMs? 0..10000 }` | optional | — |

Returns: `true`, `false, reason`

```lua
-- Half speed for two seconds, easing in and out over a quarter of a second;
-- real time comes back on its own at the deadline.
local ok, reason = Open77.world.setTimeScale(0.5, { durationMs = 2000, easeMs = 250 })
if not ok then print("no slow motion: " .. tostring(reason)) end
```

Registered in `ResourceHost.cpp` (line 25230) as `LuaWorldSetTimeScale`.
