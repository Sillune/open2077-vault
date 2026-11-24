# Open77.doors (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-doors.md) · [Page web](https://open2077.net/docs/api/client/open77-doors)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

25 functions.

## aimed

```lua
Open77.doors.aimed()
```

`GAME`: Requires a live game instance.

The streamed door currently under the crosshair.

Requires `world.doors`. The returned `id` is an opaque 64-bit REDengine identity encoded as a string; never convert it to a Lua number. A successful discovery also makes that door addressable by `state` and the command functions while it remains streamed.

Returns: `door snapshot, or nil`, `reason when the first is nil`

```lua
local door, reason = Open77.doors.aimed()
if door then
    print(door.id, door.open, door.locked, door.distance)
end
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25018) as `LuaDoorAimed`.

## aimedAny

```lua
Open77.doors.aimedAny()
```

`GAME`: Requires a live game instance.

Reads the entity currently under the crosshair even when it is not a controllable Door.

Requires `world.doors`. Unlike `aimed`, this controller-agnostic probe accepts static and scenery door meshes. It always provides identity and placement when targeting succeeds; open, locked, and sealed state is populated only when the entity exposes a door controller.

Returns: `door-like entity snapshot, or false`, `reason on failure`

```lua
-- Sink the scenery door the player is looking at, even without a controller
RegisterKeyMapping("door_probe", "Probe door", "K", function()
    local door = Open77.doors.aimedAny()
    if door then print(door.class, door.name, door.open) end
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25019) as `LuaDoorAimedAny`.

## available

```lua
Open77.doors.available()
```

`GAME`: Requires a live game instance.

Whether the client door backend is available.

This capability check does not discover or stream a door. Door operations require the `world.doors` permission.

Returns: `boolean`

```lua
-- Skip the door features entirely on a host without the backend
if not Open77.doors.available() then
    print("door backend unavailable, /lockpick disabled")
    return
end
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25009) as `LuaDoorsAvailable`.

## clearInteractionPolicy

```lua
Open77.doors.clearInteractionPolicy(door)
```

`GAME`: Requires a live game instance.

Removes this resource's interaction policy for a door.

Requires `world.doors`. Other resources' policies are unaffected.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |

Returns: `true on success, otherwise false`, `reason for refusal`

```lua
-- The heist is over: the vault door goes back to its vanilla behaviour
AddEventHandler("heist:ended", function()
    Open77.doors.clearInteractionPolicy(vaultDoorId)
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25049) as `LuaDoorPolicy`.

## close

```lua
Open77.doors.close(door, [force])
```

`GAME`: Requires a live game instance.

Closes a door.

Alias of `setOpen(door, false, force)`. Requires `world.doors`; application is asynchronous.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- Close the garage door behind the car
Open77.doors.close(garageDoorId)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25035) as `LuaDoorCommand`.

## closest

```lua
Open77.doors.closest([radius])
```

`GAME`: Requires a live game instance.

The closest streamed door around the local player.

Requires `world.doors`. Uses the same 0.5 to 100 metre player-centered query as `near`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `radius` | `number` | optional | `20` |

Returns: `door snapshot, or nil`, `reason when the first is nil`

```lua
-- Press E to toggle the nearest door within 3 m
RegisterKeyMapping("door_toggle", "Open or close door", "E", function()
    local door = Open77.doors.closest(3.0)
    if not door then return end
    Open77.doors.setOpen(door.id, not door.open)
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25022) as `LuaDoorClosest`.

## grantKey

```lua
Open77.doors.grantKey(door, holder)
```

`GAME`: Requires a live game instance.

Grants an entity an opening token for a door.

Requires `world.doors`. Uses the door controller's native token mechanism rather than a parallel Open77 key list. This grants authorization but does not lock or deny an otherwise usable door; access enforcement belongs to `setInteractionAllowed`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `holder` | `opaque entity id` | required | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- Give the local player's body a token for the apartment door
Open77.doors.grantKey(apartmentDoorId, Open77.players.entity(Open77.players.localId()))
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25044) as `LuaDoorCommand`.

## hasKey

```lua
Open77.doors.hasKey(door, holder)
```

`GAME`: Requires a live game instance.

Whether an entity holds an opening token for a door.

Requires `world.doors`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `holder` | `opaque entity id` | required | — |

Returns: `boolean, or nil`, `reason when the first is nil`

```lua
-- Only holders of a token get the "unlock" prompt
local me = Open77.players.entity(Open77.players.localId())
if Open77.doors.hasKey(doorId, me) then
    Open77.hud.notify("Press E to unlock", { replace = true })
end
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25025) as `LuaDoorHasKey`.

## holders

```lua
Open77.doors.holders(door)
```

`GAME`: Requires a live game instance.

The entities holding an opening token for a door.

Requires `world.doors`. Holder IDs are opaque strings for the same precision reason as door IDs. Tokens are vanilla authorization data, not a global lock rule: an unlocked interactive door remains usable with no token. Combine `hasKey` with `setInteractionAllowed` when implementing an ox_doorlock-style policy.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |

Returns: `array of opaque entity ids, or nil`, `reason when the first is nil`

```lua
-- Debug: who holds a token for this door?
for _, holder in ipairs(Open77.doors.holders(doorId) or {}) do
    print("token holder", holder)
end
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25024) as `LuaDoorHolders`.

## lock

```lua
Open77.doors.lock(door, [force])
```

`GAME`: Requires a live game instance.

Locks a door.

Alias of `setLocked(door, true, force)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- Lock the shop door when closing time comes
RegisterNetEvent("shop:closed", function()
    Open77.doors.lock(shopDoorId)
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25037) as `LuaDoorCommand`.

## near

```lua
Open77.doors.near([radius])
```

`GAME`: Requires a live game instance.

All streamed doors around the local player.

Requires `world.doors`. The radius is centered on the player and must be between 0.5 and 100 metres. Results are sorted nearest first and register each door for later ID-based operations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `radius` | `number` | optional | `20` |

Returns: `array of door snapshots, or nil`, `reason when the first is nil`

```lua
-- List every door within 10 m, nearest first
for _, door in ipairs(Open77.doors.near(10) or {}) do
    print(door.name, door.distance, door.locked and "locked" or "unlocked")
end
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25021) as `LuaDoorNear`.

## open

```lua
Open77.doors.open(door, [force])
```

`GAME`: Requires a live game instance.

Opens a door.

Alias of `setOpen(door, true, force)`. Requires `world.doors`; application is asynchronous.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- The server approved the lockpick: open the door
RegisterNetEvent("lockpick:success", function(doorId)
    Open77.doors.open(doorId, true)
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25034) as `LuaDoorCommand`.

## reset

```lua
Open77.doors.reset(door)
```

`GAME`: Requires a live game instance.

Resets a door to its configured default state.

Requires `world.doors`; queues the vanilla `ResetDoorState` event.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- Put the door back to its authored default when the mission ends
Open77.doors.reset(missionDoorId)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25043) as `LuaDoorCommand`.

## revokeKey

```lua
Open77.doors.revokeKey(door, holder)
```

`GAME`: Requires a live game instance.

Revokes an entity's opening token for a door.

Requires `world.doors`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `holder` | `opaque entity id` | required | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- The tenant moved out: take the token back
Open77.doors.revokeKey(apartmentDoorId, tenantEntity)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25045) as `LuaDoorCommand`.

## seal

```lua
Open77.doors.seal(door, [force])
```

`GAME`: Requires a live game instance.

Seals a door.

Alias of `setSealed(door, true, force)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- Seal the vault after the alarm
Open77.doors.seal(vaultDoorId, true)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25040) as `LuaDoorCommand`.

## setAutomaticClose

```lua
Open77.doors.setAutomaticClose(door, enabled)
```

`GAME`: Requires a live game instance.

Enables or disables a door's automatic closing behavior.

Requires `world.doors`; queues the vanilla `SetCloseItself` event.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- The garage door stays open until closed by a script
Open77.doors.setAutomaticClose(garageDoorId, false)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25042) as `LuaDoorCommand`.

## setInteractionAllowed

```lua
Open77.doors.setInteractionAllowed(door, allowed)
```

`GAME`: Requires a live game instance.

Pre-declares whether vanilla interaction may open a door.

Requires `world.doors`. The decision is synchronous when the game interacts with the door. Policies are owned by the resource, all active policies must allow, and they are removed automatically on stop/reload. `open77:doorInteract` is an observational event with `(doorId, activatorId, "true"|"false")`; it is too late to change that interaction.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `allowed` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `reason for refusal`

```lua
Open77.doors.setInteractionAllowed(doorId, playerHasAccess)
AddEventHandler("open77:doorInteract", function(door, activator, allowed)
    print(door, activator, allowed)
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25047) as `LuaDoorPolicy`.

## setLocked

```lua
Open77.doors.setLocked(door, locked, [force])
```

`GAME`: Requires a live game instance.

Sets whether a door is locked.

Requires `world.doors`. `force` uses the vanilla quest-authority action; application is asynchronous.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `locked` | `boolean` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- Lock or unlock the door from a server decision
RegisterNetEvent("doors:setLocked", function(doorId, locked)
    Open77.doors.setLocked(doorId, locked)
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25036) as `LuaDoorCommand`.

## setOpen

```lua
Open77.doors.setOpen(door, open, [force])
```

`GAME`: Requires a live game instance.

Sets whether a door is open.

Requires `world.doors`. Success means the REDscript action was accepted; application is asynchronous and may take up to 200 ms. `force` uses the vanilla quest-authority action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `open` | `boolean` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- Toggle a door and wait for the asynchronous application
CreateThread(function()
    Open77.doors.setOpen(doorId, true)
    Wait(250)
    local door = Open77.doors.state(doorId)
    print("open now:", door and door.open)
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25033) as `LuaDoorCommand`.

## setSealed

```lua
Open77.doors.setSealed(door, sealed, [force])
```

`GAME`: Requires a live game instance.

Sets whether a door is sealed.

Requires `world.doors`. Sealing is distinct from locking. `force` uses the vanilla quest-authority action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `sealed` | `boolean` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- Unseal the bunker once the code is entered
Open77.doors.setSealed(bunkerDoorId, false, true)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25039) as `LuaDoorCommand`.

## setState

```lua
Open77.doors.setState(door, state, [force])
```

`GAME`: Requires a live game instance.

Sets a door state by name.

Accepted states: `open`, `closed`, `locked`, `unlocked`, `sealed`, `unsealed`. Requires `world.doors`; `force` selects the quest-authority action.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `state` | `string` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- One call with the state as a word from the server
RegisterNetEvent("doors:setState", function(doorId, state)
    Open77.doors.setState(doorId, state) -- "open", "closed", "locked", ...
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25046) as `LuaDoorSetState`.

## sink

```lua
Open77.doors.sink(id, [depth])
```

`GAME`: Requires a live game instance.

Moves a static or scenery door below the floor.

Requires `world.doors`. This is a visual removal fallback for geometry that has no DoorControllerPS and therefore cannot be opened normally. Depth must be greater than zero and no more than 100 metres; the default is 3 metres.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `depth` | `number` | optional | `3.0` |

Returns: `true on success, otherwise false`, `reason`

```lua
-- A static door with no controller blocks the mission route: drop it into the floor
local door = Open77.doors.aimedAny()
if door and not door.controller then Open77.doors.sink(door.id, 3.0) end
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25020) as `LuaDoorSink`.

## state

```lua
Open77.doors.state(door)
```

`GAME`: Requires a live game instance.

A live snapshot of a streamed door.

Requires `world.doors`. Reads the live `DoorControllerPS`, never a fabricated default from the save record. Returns `door_not_streamed` after the entity unloads. Snapshot fields include `id`, `class`, `name`, `position`, `distance`, `open`, `closed`, `locked`, `sealed`, `busy`, `playerAuthorised`, `automaticClose`, toggle capabilities, shutter/lift flags, type/sides and opening speed/time.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |

Returns: `door snapshot, or nil`, `reason when the first is nil`

```lua
-- Show the lock state of the nearest door on the HUD
local door = Open77.doors.closest(3.0)
if door then
    local live = Open77.doors.state(door.id)
    if live then Open77.hud.notify(live.locked and "Locked" or "Unlocked", { replace = true }) end
end
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25023) as `LuaDoorState`.

## unlock

```lua
Open77.doors.unlock(door, [force])
```

`GAME`: Requires a live game instance.

Unlocks a door.

Alias of `setLocked(door, false, force)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- The player bought the key: unlock the door
RegisterNetEvent("doors:keyBought", function(doorId)
    Open77.doors.unlock(doorId)
end)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25038) as `LuaDoorCommand`.

## unseal

```lua
Open77.doors.unseal(door, [force])
```

`GAME`: Requires a live game instance.

Unseals a door.

Alias of `setSealed(door, false, force)`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `door` | `opaque door id` | required | — |
| `force` | `boolean` | optional | — |

Returns: `true when accepted, otherwise false`, `reason for refusal`

```lua
-- The alarm is reset: unseal the vault
Open77.doors.unseal(vaultDoorId)
```

[Networked world doors guide](../../01%20Documentation/doors.md)

Registered in `ResourceHost.cpp` (line 25041) as `LuaDoorCommand`.
