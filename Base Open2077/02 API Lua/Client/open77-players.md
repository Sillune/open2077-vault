# Open77.players (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-players.md) · [Page web](https://open2077.net/docs/api/client/open77-players)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

52 functions.

## all

```lua
Open77.players.all([options])
```

`GAME`: Requires a live game instance.

Lists every player this client currently knows about, ascending by player id.

No permission. STREAMING-SCOPED, not the server roster: it holds the players this client has been told about and can draw, so a player the server counts as connected can legitimately be absent. Anything that must be true of the whole session asks the server. The local player is included, matching FiveM's GetActivePlayers. A player who is connected but has no body is still listed; pass `{ streamed = true }` to keep only the ones with a body, which is the NetworkIsPlayerActive filter.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table { streamed }` | optional | — |

Returns: `array of player ids`

```lua
-- How many players does this client currently see with a body?
local visible = Open77.players.all({ streamed = true })
Open77.hud.notify(("%d players around you"):format(#visible - 1)) -- minus yourself
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25159) as `LuaPlayersAll`.

## allHealthStates

```lua
Open77.players.allHealthStates()
```

`GAME`: Requires a live game instance.

Lists all replicated player health snapshots known to the client.

Requires `players.life.read`; returns an empty array when unavailable.

Returns: `array of health snapshots`

```lua
-- Show a low-health warning icon over every wounded player nearby
for _, health in ipairs(Open77.players.allHealthStates()) do
    if health.health < health.maxHealth * 0.25 then
        markWounded(health.playerId)
    end
end
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25152) as `LuaPlayerHealthAll`.

## allLifeStates

```lua
Open77.players.allLifeStates()
```

`GAME`: Requires a live game instance.

Lists all replicated canonical player life states known to the client.

Requires `players.life.read`; returns an empty array when unavailable.

Returns: `array of life snapshots`

```lua
-- Count how many players around are down
local down = 0
for _, life in ipairs(Open77.players.allLifeStates()) do
    if life.phase ~= "alive" then down = down + 1 end
end
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25150) as `LuaPlayerLifeAll`.

## allowAim

```lua
Open77.players.allowAim(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- No aiming inside the safe zone; released again when the player leaves it
AddEventHandler("safezone:entered", function() Open77.players.allowAim(false) end)
AddEventHandler("safezone:left", function() Open77.players.allowAim(true) end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12381) as `LuaPlayerControl`.

## allowCrouch

```lua
Open77.players.allowCrouch(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- A cutscene where the player must stay standing
Open77.players.allowCrouch(false)
SetTimeout(10000, function() Open77.players.allowCrouch(true) end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12383) as `LuaPlayerControl`.

## allowDodge

```lua
Open77.players.allowDodge(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- The "no dodge" round of the fighting tournament
Open77.players.allowDodge(false)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12384) as `LuaPlayerControl`.

## allowEncumbrance

```lua
Open77.players.allowEncumbrance(allow)
```

`GAME`: Requires a live game instance.

Enables or suppresses native overweight penalties.

Requires `players.controls` and a strict boolean. `false` acquires resource-owned suppression of vanilla overweight warnings and movement penalties, removing an existing encumbered effect. `true` releases only this resource's claim; `resetControls`, resource stop and player/session teardown also release it. Inventory contents, measured weight and carry capacity are unchanged; server inventory rules remain separate. Update `open77_equipment` to 0.2.1 for suppression by default, or set its `Open77EquipmentConfig.nativeEncumbrance = true` to release that package's claim. See [native weight penalties](../../01%20Documentation/player-utilities.md#native-weight-penalties).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `allow` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
assert(Open77.players.allowEncumbrance(false))
-- Release this resource's suppression when appropriate:
assert(Open77.players.allowEncumbrance(true))
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 13415) as `LuaPlayerControl`.

## allowInteraction

```lua
Open77.players.allowInteraction(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- Freeze the world's prompts while a dialogue is on screen
Open77.players.allowInteraction(false)
AddEventHandler("dialogue:closed", function() Open77.players.allowInteraction(true) end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12385) as `LuaPlayerControl`.

## allowJump

```lua
Open77.players.allowJump(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- No bunny-hopping across the racing track
AddEventHandler("race:started", function() Open77.players.allowJump(false) end)
AddEventHandler("race:finished", function() Open77.players.allowJump(true) end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12381) as `LuaPlayerControl`.

## allowRunning

```lua
Open77.players.allowRunning(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- A tired player can only walk for 20 seconds
Open77.players.allowRunning(false)
SetTimeout(20000, function() Open77.players.allowRunning(true) end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12382) as `LuaPlayerControl`.

## allowShoot

```lua
Open77.players.allowShoot(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- Weapons stay out but cannot fire in the lobby
Open77.players.allowShoot(false)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12382) as `LuaPlayerControl`.

## allowWeapons

```lua
Open77.players.allowWeapons(enabled)
```

`GAME`: Requires a live game instance.

Allow or block a local-player action.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- The hospital is a no-weapon zone
AddEventHandler("hospital:entered", function() Open77.players.allowWeapons(false) end)
AddEventHandler("hospital:left", function() Open77.players.allowWeapons(true) end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12385) as `LuaPlayerControl`.

## closest

```lua
Open77.players.closest([options])
```

`GAME`: Requires a live game instance.

The nearest player with a body.

No permission. `Open77.players.nearby` reduced to its first entry, with the same options plus an optional `radius` (unbounded without one). Returns `nil, "no_player_in_range"` when nobody qualifies, which is deliberately distinct from `nearby`'s empty table.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table { radius, origin, includeSelf }` | optional | — |

Returns: `{ playerId, entity, distance, position, name, isLocal }`, `or nil, reason`

```lua
-- Press E to trade with the nearest player within 3 m
RegisterKeyMapping("trade", "Trade", "E", function()
    local other = Open77.players.closest({ radius = 3.0 })
    if not other then return Open77.hud.notify("Nobody close enough") end
    TriggerServerEvent("trade:request", other.playerId)
end)
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25165) as `LuaPlayersClosest`.

## entity

```lua
Open77.players.entity(playerId)
```

`GAME`: Requires a live game instance.

The entity id of one player's body.

No permission. Feed the result to any Open77 entity API, `Open77.character.state(entity)` first among them. The local player's body is the reserved entity id `0` that every entity API reads as "me", which is a value, not a failure. Reasons: `player_not_streamed` (known, no body right now), `unknown_player`, `invalid_player`. Cache the player id, never the entity id: a proxy body is destroyed and recreated on death, re-dress and world transition.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `entity id`, `or nil, reason`

```lua
-- Read another player's body state through their entity id
local entity = Open77.players.entity(playerId)
if entity then
    local state = Open77.character.state(entity)
    print(state.position.x, state.position.y, state.position.z)
end
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25162) as `LuaPlayersEntity`.

## freezePosition

```lua
Open77.players.freezePosition(enabled)
```

`GAME`: Requires a live game instance.

Request or release a native local-player freeze.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- Hold the player still while the shop menu is open
Open77.players.freezePosition(true)
AddEventHandler("shop:closed", function() Open77.players.freezePosition(false) end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12380) as `LuaPlayerControl`.

## freezeRotation

```lua
Open77.players.freezeRotation(enabled)
```

`GAME`: Requires a live game instance.

Request or release a native local-player freeze.

Requires players.controls and a strict boolean. Freeze true acquires a block; Allow false acquires a block. The inverse releases only this resource VM's block, never another owner or vanilla restriction. Automatically released on resource stop, death and session teardown. Does not freeze vehicles, moving platforms or server teleports. See player-utilities.md for each native restriction and limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `true, or false, reason`

```lua
-- Lock the camera direction for a fixed-angle photo
Open77.players.freezeRotation(true)
SetTimeout(5000, function() Open77.players.freezeRotation(false) end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12380) as `LuaPlayerControl`.

## fromEntity

```lua
Open77.players.fromEntity(entityId)
```

`GAME`: Requires a live game instance.

Which player wears a given body.

No permission. The inverse of `Open77.players.entity`, for turning a raycast or a proximity hit back into a player id. Accepts the entity as an integer or as the decimal string form. Entity `0` resolves to the local player. Returns `nil, "unknown_entity"` for an id that is not a live player body right now, which is what a cached id becomes after its body is retired.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entityId` | `integer|string` | required | — |

Returns: `player id`, `or nil, reason`

```lua
-- The raycast hit a body: which player is it?
local hit = Open77.world.raycast(from, to, { entities = true })
if hit and hit.entity then
    local playerId = Open77.players.fromEntity(hit.entity.engineEntity)
    if playerId then Open77.hud.notify("You are looking at player " .. playerId) end
end
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25163) as `LuaPlayersFromEntity`.

## getControlMask

```lua
Open77.players.getControlMask()
```

`GAME`: Requires a live game instance.

Read the union of Open77 player-control block bits.

Requires players.controls. Does not remove other resources' blocks or expose vanilla restriction state. See player-utilities.md.

Returns: `integer mask, or nil, reason`

```lua
-- Debug: which controls has this resource blocked?
print(("control mask: 0x%X"):format(Open77.players.getControlMask() or 0))
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12392) as `LuaPlayerControlsState`.

## getDriveByState

```lua
Open77.players.getDriveByState([playerId])
```

`GAME`: Requires a live game instance.

Reads local or remote vehicle drive-by state.

Client; requires players.read. Omit playerId or pass integer 0 for the local player. Returns enabled, available, active, phase, sequence, duration and elapsed; non-none phases also include canonical vehicle and seat. Phases are none, entering, active, exiting. active includes both transitions. duration/elapsed are seconds. Remote data must be fresh with an attached presented occupant body; unavailable data has phase none. Remote enabled reflects server policy, not another client's local resource restrictions. Returns nil, reason for unavailable players or invalid arguments. Use matching Unstable 2.31.21-unstable+op77.118 client and server (protocol 1.43), including the animation assets. Passenger support was introduced in .117; car and motorcycle drivers require .118. See [Vehicle drive-by](../../01%20Documentation/drive-by.md). Adds mode (passenger, car_driver, bike_driver) and driverYaw/driverPitch/driverRoll in degrees relative to the seated pose. Drivers use front_left and phase active with zero duration/elapsed. Only interpret mode while active; inactive data defaults to passenger with zero angles.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | `0` |

Returns: `state table or nil`, `reason on failure`

```lua
CreateThread(function()
    local state, reason = Open77.players.getDriveByState()
    if state and state.available then print(state.phase, state.vehicle, state.seat)
    elseif not state then print(reason) end
end)
```

[Passenger drive-by guide](../../01%20Documentation/drive-by.md)

Registered in `ResourceHost.cpp` (line 12379) as `LuaReadDriveBy`.

## getHealthState

```lua
Open77.players.getHealthState([playerId])
```

`GAME`: Requires a live game instance.

Reads one replicated player health snapshot.

Requires `players.life.read`. Omit `playerId` for the local player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `table { playerId, revision, health, maxHealth, armor, godMode, lastAttacker, lastDamage, lastAttackKind, lastBodyPart, lastHitDirection }, or nil`

```lua
-- A simple health bar for the local player
SetTick(function()
    local hp = Open77.players.getHealthState()
    if hp then healthBar.set(hp.health / hp.maxHealth, hp.armor) end
end)
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25151) as `LuaPlayerHealthState`.

## getHoloCallEyes

```lua
Open77.players.getHoloCallEyes([playerId])
```

`GAME`: Requires a live game instance.

Read the server-replicated blue eye-glow intent in the current routing bucket.

Requires players.holocall.read. Omit playerId for the local network player. Returns nil, state_unavailable before the initial server snapshot. False means absent from this client's active bucket snapshot; it is not a cross-bucket global query or proof that native pixels are rendered. There is intentionally no client setter. See holocall-eyes.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean enabled, or nil, reason`

```lua
-- Draw the "on a call" icon over players whose eyes glow
local glowing = Open77.players.getHoloCallEyes(playerId)
if glowing then showCallIcon(playerId) end
```

[Blue holocall eyes guide](../../01%20Documentation/holocall-eyes.md)

Registered in `ResourceHost.cpp` (line 25146) as `LuaPlayerHoloCallEyes`.

## getLifeState

```lua
Open77.players.getLifeState([playerId])
```

`GAME`: Requires a live game instance.

Reads the canonical replicated life state of one player.

Requires `players.life.read`. Omit `playerId` for the local session player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `life snapshot, or nil`

```lua
-- Show the death screen only when the server says we are dead
local life = Open77.players.getLifeState()
if life and life.phase ~= "alive" then showDeathScreen() end
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25147) as `LuaPlayerLifeState`.

## getLocalDeathContext

```lua
Open77.players.getLocalDeathContext()
```

`GAME`: Requires a live game instance.

Reads the local player's replicated death context.

Requires `players.life.read`. The snapshot includes phase, cause, killer, weapon, position, yaw, impulse, revision and grace duration.

Returns: `life snapshot, or nil`

```lua
-- Tell the player who killed them
RegisterNetEvent("hud:died", function()
    local death = Open77.players.getLocalDeathContext()
    if death and death.killer then
        Open77.hud.notify("Killed by player " .. tostring(death.killer))
    end
end)
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25148) as `LuaLocalDeathContext`.

## getModel

```lua
Open77.players.getModel([playerId])
```

`GAME`: Requires a live game instance.

Read this client's NPC presentation state for a player.

Requires players.model.read. Omit playerId or pass nil for the local player; an explicit ID must be a positive integer. Returns nil without an error for the original character, or a table with player, record, appearance, revision, entity, ready, status and optional failure. The entity handle is local and disposable, never a network player ID. Status is preparing, active, parked, not_streamed or failed. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `table? state`, `string? reason`

```lua
local model, reason = Open77.players.getModel()
if model then print(model.record, model.ready, model.status) end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `ResourceHost.cpp` (line 25140) as `LuaPlayerModel`.

## getVehicleSeat

```lua
Open77.players.getVehicleSeat([playerId])
```

`GAME`: Requires a live game instance.

Reads a replicated player's canonical vehicle-seat assignment.

Alias of `Open77.vehicles.getPlayerSeat`, exposed under the player namespace. Requires `vehicles.read`; omitting `playerId` selects the local player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | optional | — |

Returns: `table { playerId, vehicleId, seat (the canonical name only: seat_front_left, seat_front_right, seat_back_left or seat_back_right -- never driver or passenger, which are accepted aliases, not reported ones), flags (integer bitfield), entering, exiting, forcedEntry, exitLocked, forcedExit (booleans); the client copy adds animatedEntry and occupants }, or nil when the player is not assigned to a vehicle`

```lua
-- Is another player in a vehicle right now?
local seat = Open77.players.getVehicleSeat(playerId)
if seat then print("player", playerId, "is in vehicle", seat.vehicleId, "seat", seat.seat) end
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25153) as `LuaVehicleGetPlayerSeat`.

## isAiming

```lua
Open77.players.isAiming([playerId])
```

`GAME`: Requires a live game instance.

Graph-produced aiming state, including toggle aim.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- A stealth mission: warn when the player raises their weapon near the guards
SetTick(function()
    if Open77.players.isAiming() and nearGuards then
        Open77.hud.notify("Lower your weapon", { replace = true })
    end
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12366) as `LuaPlayerPredicate`.

## isCrouching

```lua
Open77.players.isCrouching([playerId])
```

`GAME`: Requires a live game instance.

Crouched locomotion state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Sneaking bonus: quieter footsteps while crouched
if Open77.players.isCrouching() then noiseLevel = 0.3 end
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12368) as `LuaPlayerPredicate`.

## isDead

```lua
Open77.players.isDead([playerId])
```

`GAME`: Requires a live game instance.

Whether a player is canonically dead or awaiting revive/respawn.

Requires `players.life.read`. Omit `playerId` for the local player. Unknown or unavailable states return false.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean`

```lua
-- Hide the HUD while the local player is dead
SetTick(function()
    Open77.hud.setVisible("all", not Open77.players.isDead())
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 25149) as `LuaPlayerIsDead`.

## isDiving

```lua
Open77.players.isDiving([playerId])
```

`GAME`: Requires a live game instance.

Native underwater swimming state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Warn about the oxygen the moment the player goes under
SetTick(function()
    if Open77.players.isDiving() then Open77.hud.notify("Watch your oxygen", { replace = true }) end
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12365) as `LuaPlayerPredicate`.

## isDriveByEnabled

```lua
Open77.players.isDriveByEnabled([playerId])
```

`GAME`: Requires a live game instance.

Reads the effective vehicle drive-by permission.

Client; requires players.read. Omit playerId or use 0 for the local player. Reads the enabled field of getDriveByState: local resource restrictions combine with server policy; remote reads report only server policy. True permits native vehicle combat but does not mean the player is currently in it. Returns nil, reason for an unavailable player or invalid arguments. Use matching Unstable 2.31.21-unstable+op77.118 client and server (protocol 1.43), including the animation assets. Passenger support was introduced in .117; car and motorcycle drivers require .118. See [Vehicle drive-by](../../01%20Documentation/drive-by.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | `0` |

Returns: `boolean enabled or nil`, `reason on failure`

```lua
CreateThread(function()
    local enabled, reason = Open77.players.isDriveByEnabled()
    if enabled == nil then print(reason) else print("Drive-by allowed:", enabled) end
end)
```

[Passenger drive-by guide](../../01%20Documentation/drive-by.md)

Registered in `ResourceHost.cpp` (line 12379) as `LuaReadDriveBy`.

## isDriver

```lua
Open77.players.isDriver([playerId])
```

`GAME`: Requires a live game instance.

Mounted in the driver seat.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- The radio key only works for the driver
RegisterKeyMapping("car_radio", "Car radio", "R", function()
    if Open77.players.isDriver() then toggleRadio() end
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12371) as `LuaPlayerPredicate`.

## isFalling

```lua
Open77.players.isFalling([playerId])
```

`GAME`: Requires a live game instance.

Falling phase.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Deploy the parachute automatically on a long fall
SetTick(function()
    if Open77.players.isFalling() and not chuteOpen then openChute() end
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12370) as `LuaPlayerPredicate`.

## isGrounded

```lua
Open77.players.isGrounded([playerId])
```

`GAME`: Requires a live game instance.

Locomotion ground contact.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- The dash key only works from the ground
RegisterKeyMapping("dash", "Dash", "SHIFT", function()
    if Open77.players.isGrounded() then dash() end
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12367) as `LuaPlayerPredicate`.

## isInDriveBy

```lua
Open77.players.isInDriveBy([playerId])
```

`GAME`: Requires a live game instance.

Reports whether a player is entering, using or leaving window combat.

Client; requires players.read. Omit playerId or use 0 for the local player. True for entering, active or exiting; false for none or stale action data. Use getDriveByState().phase == "active" when only the combat loop is relevant. Returns nil, reason for an unavailable player or invalid arguments. Use matching Unstable 2.31.21-unstable+op77.118 client and server (protocol 1.43), including the animation assets. Passenger support was introduced in .117; car and motorcycle drivers require .118. See [Vehicle drive-by](../../01%20Documentation/drive-by.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | `0` |

Returns: `boolean active or nil`, `reason on failure`

```lua
CreateThread(function()
    local active, reason = Open77.players.isInDriveBy()
    if active == nil then print(reason) else print("Window action:", active) end
end)
```

[Passenger drive-by guide](../../01%20Documentation/drive-by.md)

Registered in `ResourceHost.cpp` (line 12379) as `LuaReadDriveBy`.

## isInVehicle

```lua
Open77.players.isInVehicle([playerId])
```

`GAME`: Requires a live game instance.

Native vehicle mounting state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Show the speedometer only while in a vehicle
SetTick(function()
    speedometer.visible = Open77.players.isInVehicle() == true
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12371) as `LuaPlayerPredicate`.

## isJumping

```lua
Open77.players.isJumping([playerId])
```

`GAME`: Requires a live game instance.

Jump or double-jump phase.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Count jumps for a parkour score
local lastJumping = false
SetTick(function()
    local jumping = Open77.players.isJumping()
    if jumping and not lastJumping then jumps = jumps + 1 end
    lastJumping = jumping == true
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12370) as `LuaPlayerPredicate`.

## isLocalPuppetVisible

```lua
Open77.players.isLocalPuppetVisible()
```

`GAME`: Requires a live game instance.

Reads the combined local-avatar visibility request.

Client-only; requires players.local.visibility and takes no arguments. False means at least one resource holds a hide. True means no resource requests hiding; it does not force third person or prove a body is rendered. Returns the current combined policy immediately, while rendering follows on the game tick. Returns nil, reason when unavailable or permission is denied; unexpected arguments return unexpected_arguments. See [Local player visibility](../../01%20Documentation/local-puppet-visibility.md).

Returns: `boolean visible or nil`, `reason on failure`

```lua
CreateThread(function()
    local visible, reason = Open77.players.isLocalPuppetVisible()
    if visible == nil then print(reason) else print("Local avatar visible:", visible) end
end)
```

[Local player visibility & Prop Hunt](../../01%20Documentation/local-puppet-visibility.md)

Registered in `ResourceHost.cpp` (line 12394) as `LuaIsLocalPuppetVisible`.

## isModelReady

```lua
Open77.players.isModelReady([playerId])
```

`GAME`: Requires a live game instance.

Check whether the model is ready on this client.

Requires players.model.read. Omitted/nil playerId selects the local player; explicit IDs must be positive integers. False means no active ready model, including the original character; nil, reason indicates failure. This is local projection readiness, not an acknowledgement from every observer. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean? ready`, `string? reason`

```lua
local ready, reason = Open77.players.isModelReady()
if ready then print('Local presentation ready') end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `ResourceHost.cpp` (line 25142) as `LuaPlayerModel`.

## isModelValid

```lua
Open77.players.isModelValid(record)
```

`GAME`: Requires a live game instance.

Validate a Character record in the client's installed game data.

Requires players.model.read. Does not spawn or animate the model. True confirms an existing Character record, not compatibility of its rig, locomotion, seats or authored appearances. Invalid inputs and absent records return false, reason; permission failures return nil, reason. Cosmetic NPC presentation only: the real player retains identity, controls, health, equipment and saved appearance. Character.* records resolve against each client's installed game data, not a virtual allowlist. See player-models.md for ownership, events, cleanup and the known special-rig/vehicle limitations.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `record` | `string` | required | — |

Returns: `boolean? valid`, `string? reason`

```lua
local ok, reason = Open77.players.isModelValid('Character.Rogue')
if not ok then print(reason) end
```

[Player morphs and model lifecycle guide](../../01%20Documentation/player-models.md)

Registered in `ResourceHost.cpp` (line 25144) as `LuaPlayerModelValid`.

## isPassenger

```lua
Open77.players.isPassenger([playerId])
```

`GAME`: Requires a live game instance.

Mounted in a non-driver seat.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- A passenger may use the phone while the car moves; the driver may not
if Open77.players.isPassenger() then openPhone() end
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12372) as `LuaPlayerPredicate`.

## isReloading

```lua
Open77.players.isReloading([playerId])
```

`GAME`: Requires a live game instance.

Reloading state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Hide the crosshair while reloading
SetTick(function()
    Open77.hud.setVisible("crosshair", Open77.players.isReloading() ~= true)
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12367) as `LuaPlayerPredicate`.

## isShooting

```lua
Open77.players.isShooting([playerId])
```

`GAME`: Requires a live game instance.

Graph trigger state, not confirmation of a projectile or damage.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Is the remote player in front of us firing?
if Open77.players.isShooting(otherPlayerId) then
    print("player", otherPlayerId, "is firing")
end
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12366) as `LuaPlayerPredicate`.

## isSliding

```lua
Open77.players.isSliding([playerId])
```

`GAME`: Requires a live game instance.

Sliding locomotion state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Sliding through the finish line counts double
if Open77.players.isSliding() then score = score + 2 end
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12368) as `LuaPlayerPredicate`.

## isSprinting

```lua
Open77.players.isSprinting([playerId])
```

`GAME`: Requires a live game instance.

Sprint state rather than a velocity threshold.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Stamina drink wears off faster while sprinting
SetTick(function()
    if Open77.players.isSprinting() then buffTimer = buffTimer - 2 end
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12369) as `LuaPlayerPredicate`.

## isSwimming

```lua
Open77.players.isSwimming([playerId])
```

`GAME`: Requires a live game instance.

Native surface-swimming or diving state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Drop the held item when the player enters the water
SetTick(function()
    if Open77.players.isSwimming() and holdingBox then dropBox() end
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12365) as `LuaPlayerPredicate`.

## isVaulting

```lua
Open77.players.isVaulting([playerId])
```

`GAME`: Requires a live game instance.

Vault traversal state.

Requires players.read. Omit playerId to query the local network player. Remote players must be streamed and presentation-ready; swimming/diving are local-only and return nil, state_not_replicated remotely. Unavailable is not false. Read-only, without advancing replication counters. See player-utilities.md.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | optional | — |

Returns: `boolean, or nil, reason`

```lua
-- Play a grunt sound on a vault
local wasVaulting = false
SetTick(function()
    local vaulting = Open77.players.isVaulting()
    if vaulting and not wasVaulting then Play2DSound("audio/grunt.wav") end
    wasVaulting = vaulting == true
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12369) as `LuaPlayerPredicate`.

## local

```lua
Open77.players.local()
```

`GAME`: Requires a live game instance.

The local player's id; keyword-named alias of `localId`.

No permission. Identical function value to `Open77.players.localId`. `local` is reserved in Lua, so this name only parses through the bracket form `Open77.players["local"]()`; prefer `localId`.

Returns: `player id`, `or nil, reason`

```lua
-- Reachable only through the bracket form; prefer localId
local me = Open77.players["local"]()
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25161) as `LuaPlayersLocal`.

## localId

```lua
Open77.players.localId()
```

`GAME`: Requires a live game instance.

The local player's id.

No permission. Returns `nil, "no_session"` while not connected. Registered under `local` as well, for the FiveM-parity spelling, but `local` is a Lua keyword: `Open77.players.local()` is a syntax error and the name is only reachable as `Open77.players["local"]()`. Same function value.

Returns: `player id`, `or nil, reason`

```lua
-- Skip yourself when listing nearby players
local me = Open77.players.localId()
for _, entry in ipairs(Open77.players.nearby(20)) do
    if entry.playerId ~= me then print(entry.name, entry.distance) end
end
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25160) as `LuaPlayersLocal`.

## nearby

```lua
Open77.players.nearby(radius, [options])
```

`GAME`: Requires a live game instance.

Players with a body within `radius`, nearest first.

No permission. Centred on the local player unless `origin` is given; ties broken by player id so the order does not flicker. Each entry is `{ playerId, entity, distance, position, name, isLocal }`, where `position` is a plain x/y/z table and therefore already a vector3. Players without a body are skipped, having no position; use `all()` to see that they exist. Empty table when nobody qualifies. Reasons: `invalid_radius`, `invalid_origin`, `invalid_limit`, `no_session`, `no_local_position` (connected with no body, so no centre - pass `origin` to ask anyway).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `radius` | `number` | required | — |
| `options` | `table { origin, includeSelf, limit }` | optional | — |

Returns: `array of { playerId, entity, distance, position, name, isLocal }`, `or nil, reason`

```lua
-- Everyone within 20 m, nearest first
for _, entry in ipairs(Open77.players.nearby(20)) do
    print(("%s is %.1f m away"):format(entry.name, entry.distance))
end
```

[Players around you (client)](../../01%20Documentation/client-players.md)

Registered in `ResourceHost.cpp` (line 25164) as `LuaPlayersNearby`.

## resetControls

```lua
Open77.players.resetControls()
```

`GAME`: Requires a live game instance.

Release every player-control block owned by the calling resource VM.

Requires players.controls. Does not remove other resources' blocks or expose vanilla restriction state. See player-utilities.md.

Returns: `true, or false, reason`

```lua
-- The mini-game ends: release every block this resource holds
AddEventHandler("minigame:ended", function()
    Open77.players.resetControls()
end)
```

[Player checks & controls guide](../../01%20Documentation/player-utilities.md)

Registered in `ResourceHost.cpp` (line 12391) as `LuaPlayerControlsReset`.

## setLocalDriveByEnabled

```lua
Open77.players.setLocalDriveByEnabled(enabled)
```

`GAME`: Requires a live game instance.

Allows or denies native vehicle drive-by for the local player.

Client-only; requires players.driveby. Call with exactly one boolean from a running resource callback/thread. False holds this resource's restriction; true releases only that restriction. Another resource or server denial still wins. Stopping or reloading the resource releases its hold; disconnect resets local holds. Disabling an active passenger requests native exit; it does not equip a weapon or force a pose. Errors include expected_boolean, resource_not_running and owner_limit. Use matching Unstable 2.31.21-unstable+op77.118 client and server (protocol 1.43), including the animation assets. Passenger support was introduced in .117; car and motorcycle drivers require .118. See [Vehicle drive-by](../../01%20Documentation/drive-by.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `enabled` | `boolean` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
RegisterNetEvent("safezone:changed", function(inside)
    local ok, reason = Open77.players.setLocalDriveByEnabled(not inside)
    if not ok then print(reason) end
end)
```

[Passenger drive-by guide](../../01%20Documentation/drive-by.md)

Registered in `ResourceHost.cpp` (line 12375) as `LuaSetLocalDriveByEnabled`.

## setLocalPuppetVisible

```lua
Open77.players.setLocalPuppetVisible(visible)
```

`GAME`: Requires a live game instance.

Hides or restores the player's local avatar, including third person.

Client-only; requires players.local.visibility. Accepts exactly one boolean from a running resource callback/thread. False holds the native body, held weapon and third-person double hidden while retaining the camera and movement. True releases only this resource's hold; any other hide still wins. Rendering follows on the game tick. Resource stop/reload releases its hold; death, player replacement and disconnect clear lifecycle requests. Does not affect collision, separate props, gameplay effects or what other players see. Use server players.setVisible independently for observer visibility. Errors include expected_boolean, resource_not_running, player_not_ready and visibility_owner_quota. Requires a compatible client; check function availability on older builds. See [Local player visibility](../../01%20Documentation/local-puppet-visibility.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `visible` | `boolean` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
RegisterNetEvent("prophunt:localDisguise", function(disguised)
    local setVisible = Open77.players.setLocalPuppetVisible
    if not setVisible then return end
    local ok, reason = setVisible(not disguised)
    if not ok then print(reason) end
end)
```

[Local player visibility & Prop Hunt](../../01%20Documentation/local-puppet-visibility.md)

Registered in `ResourceHost.cpp` (line 12393) as `LuaSetLocalPuppetVisible`.
