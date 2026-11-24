# Open77.character (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-character.md) · [Page web](https://open2077.net/docs/api/client/open77-character)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

31 functions.

## aimedEntity

```lua
Open77.character.aimedEntity()
```

`GAME`: Requires a live game instance.

What the local player is aiming at, if it is a thing rather than scenery.

Requires **both** `player.aim.read` and `world.query`: where the player looks is about the player, what is there is a world entity read, and granting the first must not quietly hand out the second. Uses the engine's own look-at target, not a physics ray -- a physics ray answers with geometry and carries no entity reference. Returns `engineEntity`, `className`, `kind`, `family`, `position` and `distance`, plus `entity` and one of `playerId`/`vehicleId`/`npcId` when the thing is an Open77 body, or canonical decimal-string `propId` for a replicated prop. `family` is the raw class family and `kind` is that refined by ownership, exactly as `Open77.camera.aimRay({ entities = true })` refines it, so a vanilla crowd ped reads `populationNpc` in both. Three outcomes, kept apart on purpose: a table, a BARE `nil` for "pointing at scenery or nothing", and `nil, reason` when the question could not be asked.

Returns: `table, bare nil, or nil, reason`

```lua
local t = Open77.character.aimedEntity()
if t and t.kind == "player" then print("aiming at player", t.playerId) end
```

Registered in `ResourceHost.cpp` (line 24793) as `LuaCharacterAimedEntity`.

## aimState

```lua
Open77.character.aimState()
```

`GAME`: Requires a live game instance.

One coherent aim snapshot, with the staleness counter that makes it honest.

Requires `player.aim.read`. Returns `aiming`, `firing`, `weaponDrawn` and `stateSequence`. `stateSequence` rises on every observed CHANGE of the aim or reload bits; difference it across two reads to tell a live aim from a mirror that has not moved in minutes. It is evidence, never a gate -- nothing times an aim out, because a player may hold one as long as they like. `weaponDrawn` is a live check for a weapon object in the right hand: false proves empty hands, true does not say which weapon family. The read costs no raycast, advances no sequence and disturbs no ballistics.

Returns: `table, or nil, reason`

```lua
local a = Open77.character.aimState()
if a and a.aiming and a.firing then print("aimed fire") end
```

Registered in `ResourceHost.cpp` (line 24792) as `LuaCharacterAimState`.

## animation

```lua
Open77.character.animation([entity])
```

`GAME`: Requires a live game instance.

The animation currently playing on this character.

Reads the current animation snapshot for an entity, or for the local player when omitted. The result is observational and can change on the next frame; it does not grant animation ownership.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `clip name, or nil`

```lua
-- Debug overlay: show which clip the player's body is playing
SetTick(function()
    debugText.set("anim: " .. tostring(Open77.character.animation()))
end)
```

Registered in `ResourceHost.cpp` (line 24771) as `LuaCharacterAnimation`.

## boneAliases

```lua
Open77.character.boneAliases()
```

`GAME`: Requires a live game instance.

The alias table the slot resolver uses, as data.

No permission: it describes the API, not a body, the way `Open77.blips.sprites()` does. Returns `{ alias, slot }` pairs -- `head` to `Head`, `pelvis`/`hips` to `Hips`, `left_hand` to `LeftHand`/`left_hand`, `right_hand` to `RightHand`/`right_hand`. `chest`, `feet` and `hands` are deliberately absent: nothing measured on a 2.31 body publishes a chest or a foot slot, and a pair of hands is not one position.

Returns: `array of { alias, slot }`

```lua
-- Print the alias names bonePosition accepts
for _, pair in ipairs(Open77.character.boneAliases()) do
    print(pair.alias, "->", pair.slot)
end
```

Registered in `ResourceHost.cpp` (line 24800) as `LuaCharacterBoneAliases`.

## bonePosition

```lua
Open77.character.bonePosition([entity], name)
```

`GAME`: Requires a live game instance.

The world position of one of an entity's slots.

Requires `world.transform`. FiveM's `GetWorldPositionOfEntityBone`. `name` is a slot name the body publishes or one of the aliases `boneAliases()` lists; matching ignores case and punctuation. Anything the body does not carry is `unknown_bone`, by name, rather than a position taken from somewhere else. Read through the native `entSlotComponent.GetSlotTransform`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity id, {playerId|vehicleId|npcId|propId}, or {engineEntity, radius?}` | optional | `the local player` |
| `name` | `string` | required | — |

Returns: `table { x, y, z }, or nil`, `reason`

```lua
-- Draw a marker over a nearby player's head
local entity = Open77.players.entity(playerId)
local head = entity and Open77.character.bonePosition(entity, "head")
if head then
    Open77.markers.create({ position = { x = head.x, y = head.y, z = head.z + 0.3 }, radius = 0.2 })
end
```

Registered in `ResourceHost.cpp` (line 24799) as `LuaCharacterBonePosition`.

## bones

```lua
Open77.character.bones([entity])
```

`GAME`: Requires a live game instance.

Lists the slot names an entity actually publishes.

Requires `world.transform`. Cyberpunk has no `GetEntityBoneIndexByName`: what an entity publishes is `entSlotComponent.slots[].slotName`, authored per entity. So the question is inverted -- this call reports what this particular body carries, instead of handing out a vocabulary and hoping. An EMPTY array is a real answer, not a failure: most entities carry no slot component. Works on vehicles too, whose occupant slots derive from the same component.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity id, {playerId|vehicleId|npcId|propId}, or {engineEntity, radius?}` | optional | `the local player` |

Returns: `array of strings, or nil`, `reason`

```lua
-- Which slots does this body publish?
for _, name in ipairs(Open77.character.bones() or {}) do
    print(name)
end
```

Registered in `ResourceHost.cpp` (line 24798) as `LuaCharacterBones`.

## forward

```lua
Open77.character.forward([entity])
```

`GAME`: Requires a live game instance.

The unit vector an entity is facing along.

Requires `world.transform`. FiveM's `GetEntityForwardVector`. The same `entEntity.GetWorldForward` read `Open77.character.state().forward` answers with, so the two cannot disagree.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity id, {playerId|vehicleId|npcId|propId}, or {engineEntity, radius?}` | optional | `the local player` |

Returns: `table { x, y, z }, or nil`, `reason`

```lua
-- A placement ghost two metres in front of the player
local me = Open77.character.state().position
local fwd = Open77.character.forward()
local spot = vector3(me.x + fwd.x * 2, me.y + fwd.y * 2, me.z)
Open77.props.create({ model = "crate.small", position = spot, collision = false })
```

Registered in `ResourceHost.cpp` (line 24795) as `LuaCharacterForward`.

## frame

```lua
Open77.character.frame([entity])
```

`GAME`: Requires a live game instance.

Reads an entity's position and its own three axes, in world space.

Requires `world.transform`. Returns `{ position, forward, right, up }`, every axis normalised, in one engine pass. The axes are read from `entEntity.GetWorldForward/GetWorldRight/GetWorldUp`, including pitch and roll. Accepts a receiver-local entity id, a canonical `{playerId}`, `{vehicleId}`, `{npcId}` or `{propId}` selector, or `{engineEntity, radius?}`. A prop id is canonical unsigned-64 decimal text. An engine id may be a lossless decimal/hex string or a Lua integer bit pattern. A canonical id on a decorated ray record takes precedence over `engineEntity`; an unavailable canonical parent never falls through to another identity.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity id, {playerId|vehicleId|npcId|propId}, or {engineEntity, radius?}` | optional | `the local player` |

Returns: `table, or nil`, `reason`

```lua
-- Spawn a marker one metre to the player's right
local f = Open77.character.frame()
Open77.markers.create({
    position = { x = f.position.x + f.right.x, y = f.position.y + f.right.y, z = f.position.z },
    radius = 0.3,
})
```

Registered in `ResourceHost.cpp` (line 24794) as `LuaCharacterFrame`.

## groundSpeed

```lua
Open77.character.groundSpeed([entity])
```

`GAME`: Requires a live game instance.

Speed projected on the ground, without the vertical component.

Returns horizontal movement speed for the selected character, excluding vertical velocity. Entity ids refer to the current client stream and must not be persisted as server identity.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `number`

```lua
-- A speedometer on foot, ignoring jumps and falls
SetTick(function()
    hud.speed = ("%.1f m/s"):format(Open77.character.groundSpeed())
end)
```

Registered in `ResourceHost.cpp` (line 24773) as `LuaCharacterNumber<&SelectGroundSpeed>`.

## health

```lua
Open77.character.health([entity])
```

`GAME`: Requires a live game instance.

Current health.

Reads the engine-side health visible for the selected character on this client. For replicated player gameplay rules use the server-authoritative player health APIs; this helper is a presentation snapshot, not authority.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `number`

```lua
-- A low-health screen tint, from the presentation snapshot
SetTick(function()
    vignette.visible = Open77.character.health() < 30
end)
```

Registered in `ResourceHost.cpp` (line 24774) as `LuaCharacterNumber<&SelectHealth>`.

## isAiming

```lua
Open77.character.isAiming()
```

`GAME`: Requires a live game instance.

Whether the local player is aiming down sights.

Requires `player.aim.read`. Read from the player's own weapon animation graph, not from `Open77.perspective.state().aiming` -- that field has only two writers, a trigger pull and the third-person self-view's per-frame gate, and the second is skipped entirely in first person, so in FPP it reports whatever the last shot said for as long as the player does not shoot again. **The underlying bit is a latched mirror, not a poll**: it keeps its value between two pushes from the graph, so `true` alone cannot distinguish "aiming now" from "the last thing the graph ever said was aim". Use `Open77.character.aimState().stateSequence` when that distinction matters. `nil, "aim_state_unavailable"` when nothing has been observed yet.

Returns: `boolean, or nil, reason`

```lua
-- Show the hit marker layer only while aiming down sights
if Open77.character.isAiming() then showHitMarkers() end
```

Registered in `ResourceHost.cpp` (line 24790) as `LuaCharacterAimFlag<&GameBackend::AimState::aiming>`.

## isAlive

```lua
Open77.character.isAlive([entity])
```

`GAME`: Requires a live game instance.

Whether the character is alive.

Reports the current engine alive/dead flag for the selected character. It is useful for presentation and diagnostics but may lag the authoritative server health transition by a replication frame.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- Do not draw the HUD over the death screen
SetTick(function()
    hud.visible = Open77.character.isAlive()
end)
```

Registered in `ResourceHost.cpp` (line 24776) as `LuaCharacterFlag<&SelectAlive>`.

## isArmed

```lua
Open77.character.isArmed([entity])
```

`GAME`: Requires a live game instance.

Whether a weapon is drawn.

Reports whether the selected character is currently considered armed by the game presentation state. It does not enumerate inventory or prove that a weapon action is server-authorized.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- The shop refuses to open while a weapon is drawn
RegisterKeyMapping("shop_open", "Open shop", "E", function()
    if Open77.character.isArmed() then return Open77.hud.notify("Holster your weapon first") end
    openShop()
end)
```

Registered in `ResourceHost.cpp` (line 24781) as `LuaCharacterFlag<&SelectArmed>`.

## isCrouched

```lua
Open77.character.isCrouched([entity])
```

`GAME`: Requires a live game instance.

Whether the character is crouching.

Reads the character's current crouch presentation flag. Omit the entity to inspect the local player; remote values exist only while that proxy is streamed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- Sneaking makes the stealth meter fill slower
SetTick(function()
    detection = detection + (Open77.character.isCrouched() and 0.2 or 1.0)
end)
```

Registered in `ResourceHost.cpp` (line 24778) as `LuaCharacterFlag<&SelectCrouched>`.

## isDriver

```lua
Open77.character.isDriver([entity])
```

`GAME`: Requires a live game instance.

Whether they occupy the driver's seat.

Returns whether the selected character occupies the canonical driver seat of a vehicle. Use `seat()` when the exact seat and vehicle entity are also needed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- Only the driver hears the radio station switch prompt
if Open77.character.isDriver() then Open77.hud.notify("Press R to change station") end
```

Registered in `ResourceHost.cpp` (line 24783) as `LuaCharacterFlag<&SelectDriverSeat>`.

## isEmoting

```lua
Open77.character.isEmoting([entity])
```

`GAME`: Requires a live game instance.

Whether they hold a workspot (an emote is running).

Reports whether Open77 currently presents the character in an emote/workspot state. This is a transient local snapshot and is cleared with the owning presentation lifecycle.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- Do not start a second emote while one is running
RegisterKeyMapping("emote_wave", "Wave", "F5", function()
    if Open77.character.isEmoting() then return end
    TriggerServerEvent("emotes:play", "wave")
end)
```

Registered in `ResourceHost.cpp` (line 24784) as `LuaCharacterFlag<&SelectInWorkspot>`.

## isFiring

```lua
Open77.character.isFiring()
```

`GAME`: Requires a live game instance.

Whether the local player is holding the trigger.

Requires `player.aim.read`. The FiveM `IsPedShooting` equivalent, read from `gameweaponAnimFeature_WeaponData` on the player's own graph -- it is the trigger being HELD, not a shot event, so it stays true across a burst and is true even when the shot is refused. Same call and same snapshot as `isAiming()`; call `aimState()` once if you want both without two reads. Nothing here reads a key or a bind: a player who fires from a script or a macro raises the same flag, because the same thing happened to the same graph.

Returns: `boolean, or nil, reason`

```lua
-- Screen shake while the trigger is held
SetTick(function()
    if Open77.character.isFiring() then Open77.camera.shake(nil, "hand", 0.5, 100) end
end)
```

Registered in `ResourceHost.cpp` (line 24791) as `LuaCharacterAimFlag<&GameBackend::AimState::firing>`.

## isFrozen

```lua
Open77.character.isFrozen()
```

`GAME`: Requires a live game instance.

Whether the local body is being held still.

Client-only; requires `player.freeze`. True while the server's freeze bit is set OR any client resource holds a local claim -- including one that is not yours. It is **not** true merely because you are dead: `Open77.character.isAlive` answers that question, and answers it better.

Returns: `boolean, or nil`, `reason for the refusal`

```lua
-- Explain why the body will not move
if Open77.character.isFrozen() then
    Open77.hud.notify("You are being held in place")
end
```

Registered in `ResourceHost.cpp` (line 24786) as `LuaCharacterIsFrozen`.

## isGrounded

```lua
Open77.character.isGrounded([entity])
```

`GAME`: Requires a live game instance.

Whether the character is touching the ground.

Reads the engine grounded flag for the selected character. The value is sampled from the current client frame and should not be used alone for authoritative movement validation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- Count the time spent in the air for a stunt score
SetTick(function()
    if not Open77.character.isGrounded() then airtime = airtime + 1 end
end)
```

Registered in `ResourceHost.cpp` (line 24777) as `LuaCharacterFlag<&SelectGrounded>`.

## isInVehicle

```lua
Open77.character.isInVehicle([entity])
```

`GAME`: Requires a live game instance.

Whether the character is mounted in a vehicle.

Reports whether the character is mounted in a vehicle according to the current client presentation. Call `seat()` to retrieve the mounted slot and vehicle entity id.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- Show the vehicle HUD only while mounted
SetTick(function()
    vehicleHud.visible = Open77.character.isInVehicle()
end)
```

Registered in `ResourceHost.cpp` (line 24782) as `LuaCharacterFlag<&SelectInVehicle>`.

## isSliding

```lua
Open77.character.isSliding([entity])
```

`GAME`: Requires a live game instance.

Whether the character is sliding.

Reads the current sliding locomotion flag for the selected character. It is an observational state and does not start or stop a slide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- Slide under the bar to win the parkour challenge
if Open77.character.isSliding() and nearBar then TriggerServerEvent("parkour:slid") end
```

Registered in `ResourceHost.cpp` (line 24779) as `LuaCharacterFlag<&SelectSliding>`.

## isVaulting

```lua
Open77.character.isVaulting([entity])
```

`GAME`: Requires a live game instance.

Whether the character is vaulting.

Reads the current vaulting locomotion flag for the selected character. Remote values are available only while the corresponding proxy is streamed.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `boolean`

```lua
-- Award a point per vault
local wasVaulting = false
SetTick(function()
    local vaulting = Open77.character.isVaulting()
    if vaulting and not wasVaulting then points = points + 1 end
    wasVaulting = vaulting
end)
```

Registered in `ResourceHost.cpp` (line 24780) as `LuaCharacterFlag<&SelectVaulting>`.

## offsetToWorld

```lua
Open77.character.offsetToWorld([entity], offset)
```

`GAME`: Requires a live game instance.

Turns a point in an entity's own frame into world coordinates.

Requires `world.transform`. FiveM's `GetOffsetFromEntityInWorldCoords`. The offset is in the entity's frame: `+y` ahead, `+x` to its right, `+z` up, which is REDengine's convention. Accepts the same target selectors as `frame`. The vector may be passed alone -- `offsetToWorld(vector3(0, 2, 0))` means two metres in front of the local player -- but an identity selector without a vector is refused. Pitch and roll are in the answer for a tilted entity, because they are in the engine's axes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity id, {playerId|vehicleId|npcId|propId}, or {engineEntity, radius?}` | optional | `the local player` |
| `offset` | `table { x, y, z }` | required | — |

Returns: `table { x, y, z }, or nil`, `reason`

```lua
local ahead = Open77.character.offsetToWorld(vector3(0, 2, 0))
Open77.markers.create({ position = ahead, radius = 0.5 })
```

Registered in `ResourceHost.cpp` (line 24796) as `LuaCharacterOffsetToWorld`.

## position

```lua
Open77.character.position([entity])
```

`GAME`: Requires a live game instance.

The character's position.

Returns the character's current world-space coordinates from the game-thread snapshot. Omit the entity for the local player; do not persist a client entity id as a network identity.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `x, y, z`

```lua
-- Show the player's coordinates on the HUD
SetTick(function()
    local x, y, z = Open77.character.position()
    hud.coords = ("%.1f  %.1f  %.1f"):format(x, y, z)
end)
```

Registered in `ResourceHost.cpp` (line 24768) as `LuaCharacterPosition`.

## seat

```lua
Open77.character.seat([entity])
```

`GAME`: Requires a live game instance.

The seat occupied in a vehicle, if any.

Returns the canonical seat name and the mounted vehicle's client entity id when the character is in a vehicle. A character that is not mounted returns `nil`; both values are snapshots and the entity id is valid only in the current client stream.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `seat name and vehicle entity id, or nil`, `reason when the character could not be read`

```lua
-- Which seat is the player in?
local seat, vehicleEntity = Open77.character.seat()
if seat then print("sitting in", seat, "of entity", vehicleEntity) end
```

Registered in `ResourceHost.cpp` (line 24769) as `LuaCharacterSeat`.

## setFrozen

```lua
Open77.character.setFrozen(frozen)
```

`GAME`: Requires a live game instance.

Holds the local player's body still, without a server round trip.

Client-only; requires `player.freeze`. **Presentation-local**: it holds YOUR body on YOUR machine, for a menu, a progress bar or a short scripted beat where a server round trip would be felt. The server's `Open77.players.setFrozen` outranks it outright -- while the server holds you, releasing your own claim changes nothing and the second return value stays `true`.

The hold is the engine's own `GameplayRestriction.NoMovement`, the same record vanilla death applies. It stops every locomotion input -- walk, run, sprint, jump, dodge, slide, vault -- and deliberately stops nothing else: camera, chat, voice, menus, interaction prompts and damage all keep working, and you can still be killed where you stand.

The claim is keyed by your resource, so another resource's claim survives your release. Yours is dropped for you when your resource stops or reloads, when you die, and when the session ends -- a freeze whose owner is gone is the one failure this must not have. The second return value is whether the body is still held afterwards by anybody at all.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `frozen` | `boolean` | required | — |

Returns: `true on success, otherwise false`, `effectiveFrozen, or the reason for the refusal`

```lua
-- Hold the body locally while the shop menu is open; the server's hold outranks ours
local ok, stillHeld = Open77.character.setFrozen(true)
AddEventHandler("shop:closed", function() Open77.character.setFrozen(false) end)
```

Registered in `ResourceHost.cpp` (line 24785) as `LuaCharacterSetFrozen`.

## speed

```lua
Open77.character.speed([entity])
```

`GAME`: Requires a live game instance.

Instantaneous speed, all components.

Returns total character movement speed, including vertical motion, from the latest local snapshot. Use `groundSpeed()` when only horizontal locomotion is relevant.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `number`

```lua
-- A crash detector: a sudden drop in speed
local last = 0
SetTick(function()
    local now = Open77.character.speed()
    if last - now > 15 then Open77.hud.notify("Ouch!") end
    last = now
end)
```

Registered in `ResourceHost.cpp` (line 24772) as `LuaCharacterNumber<&SelectSpeed>`.

## state

```lua
Open77.character.state([entity])
```

`GAME`: Requires a live game instance.

One complete snapshot of a character.

A single engine pass fills the whole record even when the caller only wanted the speed. That is deliberate: every field then describes the **same instant**, so a caller cannot see a body grounded and airborne in the same breath.

With no argument, answers about the local player. An integer is an **Open77 entity id**.

A table `{ engineEntity = n, radius? }` reads a body Open77 does **not** own -- a vanilla population ped, a traffic car -- addressed by its engine id, because it has no Open77 id to address it with. Those answers carry `source = "observed"` and `owned = false`, `id` is 0, and only what can be read straight off the object is filled in: placement, orientation and health. Locomotion, weapon and vehicle state come from Open77's own replication and are left unset rather than guessed at. The body is located among what is streamed around the local player, so `entity_not_observed` means "not in view from here", which for a vanilla ped is an ordinary outcome and not an error. There is no write counterpart: every mutator is keyed by an Open77 id and answers `npc_not_found` for one of these.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity id, or table { engineEntity, radius? }` | optional | — |

Returns: `table { isPlayer, attached, id, engineId, position, orientation, forward, velocity, speed, groundSpeed, yaw, alive, health, grounded, crouched, sliding, vaulting, air, fall, landing, weapon…, inVehicle, seat, driver }`, `reason, when the first is nil`

```lua
local s = Open77.character.state()
if s.inVehicle and s.driver then
    print("driving at " .. math.floor(s.speed) .. " u/s")
end
```

Registered in `ResourceHost.cpp` (line 24767) as `LuaCharacterState`.

## weapon

```lua
Open77.character.weapon([entity])
```

`GAME`: Requires a live game instance.

The equipped weapon and its state.

Returns the drawn weapon's recognized item id and category for the selected character. It returns `nil` when no recognized weapon is currently presented and does not expose or mutate inventory.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `table { itemId, category }, or nil`, `reason when the character could not be read`

```lua
-- Show the weapon category in a custom HUD
local weapon = Open77.character.weapon()
hud.weapon = weapon and weapon.category or "unarmed"
```

Registered in `ResourceHost.cpp` (line 24770) as `LuaCharacterWeapon`.

## worldToOffset

```lua
Open77.character.worldToOffset([entity], point)
```

`GAME`: Requires a live game instance.

Turns a world point into an entity's own frame.

Requires `world.transform`. The exact inverse of `offsetToWorld`: the basis is orthonormal, so its transpose is its inverse. `y > 0` means the point is in front of the entity.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity id, {playerId|vehicleId|npcId|propId}, or {engineEntity, radius?}` | optional | `the local player` |
| `point` | `table { x, y, z }` | required | — |

Returns: `table { x, y, z }, or nil`, `reason`

```lua
-- Is the marker in front of the player or behind?
local rel = Open77.character.worldToOffset(vector3(-1442.2, 127.4, 18.0))
if rel and rel.y > 0 then Open77.hud.notify("The target is ahead of you") end
```

Registered in `ResourceHost.cpp` (line 24797) as `LuaCharacterWorldToOffset`.

## yaw

```lua
Open77.character.yaw([entity])
```

`GAME`: Requires a live game instance.

Horizontal orientation, in degrees.

Returns the selected character's current world yaw in degrees. The value is a presentation snapshot and should be normalized by the caller when computing angular deltas.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entity` | `entity` | optional | — |

Returns: `number`

```lua
-- A compass reading
SetTick(function()
    hud.heading = ("%d deg"):format(Open77.character.yaw() % 360)
end)
```

Registered in `ResourceHost.cpp` (line 24775) as `LuaCharacterNumber<&SelectYaw>`.
