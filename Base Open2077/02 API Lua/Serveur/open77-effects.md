# Open77.effects (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-effects.md) · [Page web](https://open2077.net/docs/api/server/open77-effects)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

17 functions.

## all

```lua
Open77.effects.all([bucket])
```

`NETWORK`: Uses the network backend.

Every looping effect, optionally filtered to one routing bucket.

Requires `world.effects`. Returns the whole registry, not just this resource's effects; the snapshot's `resource` field is provenance and cleanup scope, not an authorization boundary.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `array of effect snapshots`

```lua
-- Count the fires still burning in the arena instance
local burning = 0
for _, effect in ipairs(Open77.effects.all(ARENA_BUCKET)) do
    if effect.effect:sub(1, 5) == "fire." then burning = burning + 1 end
end
```

Registered in `LuaResourceRuntime.cs` (line 9776) as `LuaResourceRuntime.Bootstrap`.

## attach

```lua
Open77.effects.attach(target, effect, options)
```

`NETWORK`: Uses the network backend.

Create a replicated effect bound to a typed target and native slot.

Requires world.effects. target={kind="player"|"npc"|"vehicle"|"prop",id=decimalStringOrInteger}; effect is an installed preset. options requires slot; optional localAnchor=body|weaponRight, localSlot, ttlMs, streamingRadius=90, streamingHysteresis=20, localEvent, soundEvent, soundOnOwner=true. Uses exact target incarnation, streaming reattachment and resource-owned cleanup. localEvent selects an owner entity graph (not a sound); soundEvent is separate. Target disappearance/body/bucket change invalidates the effect. The ID proves registry admission, not visible/audible playback. Attachment TTL defaults60000 ms and is bounded to1..600000 ms; slot/localSlot are at most64 bytes. Limits are2048 effects globally and512 per resource. weaponRight/localEvent are player-only.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `table` | required | — |
| `effect` | `string` | required | — |
| `options` | `table` | required | — |

Returns: `decimal string effect ID`, `nil, reason on rejection`

```lua
local id, err = Open77.effects.attach({kind="player",id=tostring(playerId)},
 "electric.industrial_arm", {slot="RightHand",ttlMs=1500})
if not id then print(err); return end
-- Retain id and remove it when your action ends.
```

Registered in `LuaResourceRuntime.cs` (line 9731) as `LuaResourceRuntime.Bootstrap`.

## catalog

```lua
Open77.effects.catalog()
```

`NETWORK`: Uses the network backend.

Always an empty table on the server.

The dedicated server keeps no curated effect catalogue: the aliases live on the client, next to the records they resolve. It answers an empty list rather than `nil` so the shipped console command stays honest instead of failing on a nil call. Use the client `Open77.vfx` catalogue for alias names.

Returns: `empty table`

```lua
-- Always empty on the server; the alias list lives on the client (Open77.vfx.catalog)
print(#Open77.effects.catalog()) -- 0
```

Registered in `LuaResourceRuntime.cs` (line 9817) as `LuaResourceRuntime.Bootstrap`.

## clearScreen

```lua
Open77.effects.clearScreen(playerId)
```

`GAME`: Requires a live game instance.

Clears every server-driven screen effect on one player.

Requires `players.screenfx`. The same thing as `Open77.effects.screen(playerId, false)`, spelled out. Only the effects this relay owns are cleared: a client resource's own screen effects belong to that resource. See the Effects guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `number` | required | — |

Returns: `true, or false and a reason`

```lua
-- Sobering up: clear every screen effect this resource put on the player
RegisterNetEvent("items:coffee", function()
    Open77.effects.clearScreen(source)
end)
```

Registered in `LuaResourceRuntime.cs` (line 9815) as `LuaResourceRuntime.Bootstrap`.

## create

```lua
Open77.effects.create(def)
```

`NETWORK`: Uses the network backend.

Creates a looping world effect owned by this resource.

Requires `world.effects`. The field is `effect`, not `name`, and the lifetime is `ttlMs`: a looping VFX has no duration because the registry retires it, not the engine. Definitions accept `effect`, `position`, `orientation`, `bucket`, `visible`, `streamingRadius` (10 to 2000, default 90), `streamingHysteresis` (default 20) and `ttlMs`. The registry holds 2,048 looping effects, of which one resource may own 512.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `def` | `table { effect, position, orientation?, bucket?, visible?, streamingRadius?, streamingHysteresis?, ttlMs? }` | required | — |

Returns: `effect id (decimal string), or nil`, `reason`

```lua
-- A burning barrel that stays lit for ten minutes
local fire, reason = Open77.effects.create({
    effect = "fire.small",
    position = { x = -1460.2, y = 99.9, z = 14.8 },
    ttlMs = 10 * 60 * 1000,
})
if not fire then print("no fire: " .. reason) end
```

Registered in `LuaResourceRuntime.cs` (line 9753) as `LuaResourceRuntime.Bootstrap`.

## explosion

```lua
Open77.effects.explosion(position, [options])
```

`NETWORK`: Uses the network backend.

Detonates a blast at a point: VFX, area damage, and optionally the cars in it.

Native-parity E6, FiveM's `AddExplosion`. It composes three authorities that already exist and adds none: the picture is the same one-shot as `Open77.effects.play`, the player damage goes through the stats authority exactly as `Open77.players.damage` does (so god mode, the life-phase interlocks, the `explosion` damage multiplier and kill attribution all apply), and vehicles go through `Open77.vehicles.explode` unchanged -- occupants stay occupants, and a wreck already blown up is quietly skipped. **Three grants, because the halves differ in reach.** `world.effects` is always required and on its own buys a blast that is only a picture. `world.explosions` is required the moment `damage` is positive: unlike `players.stats.apply`, which hurts a player the resource has already named, this one hurts everyone in a circle, including players it never enumerated. `vehicles = true` requires `world.vehicles`, and without that opt-in no car is touched. Damage falls off linearly to zero at the rim. `force` scales the ragdoll impulse on a **lethal** hit only -- there is no server-side push for a survivor. A player whose last snapshot is stale is skipped rather than damaged at a position the server is unsure of, and everything is scoped to one routing bucket. Publishes `onExplosion` host-wide, with counts rather than names. See the [effects guide](https://open2077.net/docs/api/server/effects.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y, z }` | required | — |
| `options` | `table { radius?, damage?, force?, vfx?, sound?, bucket?, vehicles?, attacker? }` | optional | — |

Returns: `table { players = { ids }, vehicles = { ids } }, or nil`, `reason: permission_denied:world.effects, permission_denied:world.explosions, permission_denied:world.vehicles, invalid_position, invalid_radius, invalid_damage, invalid_force, invalid_bucket, invalid_effect, invalid_sound, invalid_player_id, damage_unavailable, vehicles_unavailable, world_unavailable`

```lua
-- /boom: an explosion at the caller's feet that hurts players and wrecks cars
RegisterCommand("boom", function(source)
    local me = Open77.players.position(source)
    if not me then return end
    local blast = Open77.effects.explosion(me, { radius = 8, damage = 60, vehicles = true, attacker = source })
    if blast then print(#blast.players, "players hit,", #blast.vehicles, "vehicles wrecked") end
end, true)
```

Registered in `LuaResourceRuntime.Explosions.cs` (line 258) as `LuaResourceRuntime.Bootstrap`.

## fire

```lua
Open77.effects.fire(position, [options])
```

`NETWORK`: Uses the network backend.

Lights a script fire at a point: looping VFX plus a damage tick for whoever stands in it.

Native-parity E7, FiveM's `StartScriptFire`. **The fire is one looping effect, and the `fireId` it answers IS that effect's id** -- same registry, same ownership, revisioning and streaming, so a player who walks away and comes back still finds it burning, `Open77.effects.get(fireId)` describes the flames and `onEffectCreated` announces them. Two ids for one object would have been two things that could disagree about whether it is still there. The registry is therefore the authority on EXISTENCE and the owning resource only on damage: `Open77.effects.remove(fireId)` puts the fire out with the reason `effect_removed` instead of leaving an invisible one burning, while expiry works the other way round -- the resource's own deadline retires the entry, which is why the effect carries no TTL of its own. **Damage is flat inside the radius, deliberately unlike an explosion's linear falloff**: a fire is a volume you are inside or outside of, and a falloff would make the rim nearly free. One damage interval is one second, so `damagePerSecond` is the literal unit; a billed interval is capped at five seconds so a server hitch cannot settle a minute of burning at once; and a player whose last snapshot is stale is skipped rather than burned at a position the server is unsure of. Damage goes through the same scripted-damage funnel as `Open77.players.damage` with the attack kind `environment` (2.31 has no fire kind), so god mode, the life-phase interlocks, the damage multiplier and kill attribution all apply. **The same three grants as an explosion and no new manifest string**: `world.effects` buys flames that are only a picture, `world.explosions` is required the moment `damagePerSecond` is positive, and `vehicles = true` requires `world.vehicles`. The capability describes the reach -- area damage to players the resource never enumerated -- and a fire reaches exactly as far as a blast repeated once a second. With `vehicles = true` a car burns at the same rate against its 0-1 pool scaled through one stated rule (a fire that kills a healthy player in ten seconds wrecks a healthy car in ten seconds) and one that reaches zero is exploded through `Open77.vehicles.explode`, C12's call and not a second way to blow up a car. At most 64 fires per resource, much lower than the effect registry's 512 because each one costs a per-second proximity sweep. Publishes `onFire` host-wide (and `fireEvent` under the FiveM name), with a count rather than names. See the [effects guide](https://open2077.net/docs/api/server/effects.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `position` | `table { x, y, z }` | required | — |
| `options` | `table { radius?, damagePerSecond?, durationMs?, vfx?, bucket?, vehicles?, attacker? }` | optional | — |

Returns: `fire id (decimal string), which is also the looping effect's id, or nil`, `reason: permission_denied:world.effects, permission_denied:world.explosions, permission_denied:world.vehicles, invalid_position, invalid_radius, invalid_damage, invalid_duration, invalid_bucket, invalid_effect, invalid_player_id, fire_limit, quota_exceeded, damage_unavailable, vehicles_unavailable, world_unavailable, resource_stopping`

```lua
-- Arson RP: a car park burns for a minute and hurts anyone who lingers.
local fireId, reason = Open77.effects.fire({ x = -1448.2, y = 96.1, z = 17.5 }, {
    radius          = 5.0,
    damagePerSecond = 8.0,
    durationMs      = 60000,
    vfx             = "fire.large",
    attacker        = source,
})
if not fireId then print("no fire: " .. reason) end
```

Registered in `LuaResourceRuntime.Fires.cs` (line 521) as `LuaResourceRuntime.Bootstrap`.

## fires

```lua
Open77.effects.fires([bucket])
```

`NETWORK`: Uses the network backend.

Lists the fires this resource still has burning, ascending by id.

Native-parity E7. Requires `world.effects`. Answers **this resource's** fires and nobody else's: they are its own to put out, and the host-wide `onFire` is how the rest of the server learns about somebody else's. Each row carries `id` (which is also the looping effect's id), `effect`, `position`, `radius`, `damagePerSecond`, `bucket`, `vehicles`, `attacker`, `hurt` and `remainingMs`. `hurt` is a COUNT of distinct players the fire has burned, never a roster -- the same rule the event follows, because who was standing in a fire is a proximity read the caller can do for itself if it is entitled to. `remainingMs` is absent for a fire lit with `durationMs = 0`, so an eternal fire is distinguishable from one about to go out rather than both reading as zero. An optional bucket narrows the list.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `array of { id, effect, position, radius, damagePerSecond, bucket, vehicles, attacker, hurt, remainingMs? }`

```lua
-- A dispatch board: what is alight, and how long it has left.
for _, burning in ipairs(Open77.effects.fires()) do
    print(("fire %s r=%.1f dps=%.1f hurt=%d left=%s"):format(
        burning.id, burning.radius, burning.damagePerSecond, burning.hurt,
        burning.remainingMs and tostring(burning.remainingMs) or "until removed"))
end
```

Registered in `LuaResourceRuntime.Fires.cs` (line 551) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.effects.get(id)
```

`NETWORK`: Uses the network backend.

Reads one looping effect's canonical snapshot.

Requires `world.effects`. Reads are not restricted to the owning resource, so a spectator or admin tool can inspect an effect it did not create. Answers `nil` for an unknown id.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `effect snapshot, or nil`

```lua
-- Is the campfire still burning?
local fire = Open77.effects.get(campfireId)
if not fire then relight() end
```

Registered in `LuaResourceRuntime.cs` (line 9775) as `LuaResourceRuntime.Bootstrap`.

## play

```lua
Open77.effects.play(name, opts)
```

`NETWORK`: Uses the network backend.

Plays a one-shot world effect at a point.

Requires `world.effects`. Broadcast to the players in range and never stored, so there is no handle and nothing to remove. Options are `position` (required, or the options table itself), `orientation` (quaternion, default identity), `bucket`, `range` (1 to 500, default 150) and `sound`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `opts` | `table { position, orientation?, bucket?, range?, sound? }` | required | — |

Returns: `true, or nil`, `reason`

```lua
-- Sparks where the player's lockpick failed
RegisterNetEvent("lockpick:failed", function()
    local me = Open77.players.position(source)
    if me then Open77.effects.play("sparks.burst.small", { position = me, range = 50 }) end
end)
```

Registered in `LuaResourceRuntime.cs` (line 9738) as `LuaResourceRuntime.Bootstrap`.

## playOn

```lua
Open77.effects.playOn(target, name, [options])
```

`NETWORK`: Uses the network backend.

Play an ephemeral native effect on a typed target.

Requires world.effects. A bare target ID explicitly means player; typed targets avoid collisions between registries. name is a native entity effect event. options supports duration(default0),loop(defaultfalse),slot. Snapshot lifetime guards reject replaced targets; this one-shot is not replayed after streaming. Use attach for a durable owned effect.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `table|string|integer` | required | — |
| `name` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `true`, `nil, reason on rejection`

```lua
local ok, err = Open77.effects.playOn({kind="player",id=tostring(playerId)}, nativeEvent, {duration=1})
if not ok then print(err) end
```

Registered in `LuaResourceRuntime.cs` (line 9777) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.effects.remove(id)
```

`NETWORK`: Uses the network backend.

Removes a looping effect this resource owns.

Requires `world.effects`. Another resource's effect answers `owned_by_another_resource`; a resource may read every effect but mutate only its own.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The firefighters put the fire out
Open77.effects.remove(fireId)
```

Registered in `LuaResourceRuntime.cs` (line 9774) as `LuaResourceRuntime.Bootstrap`.

## removeFire

```lua
Open77.effects.removeFire(fireId)
```

`NETWORK`: Uses the network backend.

Puts one of this resource's fires out: the flames and the burning stop together.

Native-parity E7, FiveM's `RemoveScriptFire`. Requires `world.effects`. Removes the looping effect the fire is made of and stops the damage tick in the same statement, then publishes `onFire` with the state `stopped` and the reason `removed`. A fire that has already expired or been removed answers `not_found`, exactly as removing a looping effect twice does, and an id that belongs to another resource answers `owned_by_another_resource` -- the same distinction the effect registry draws, because "you put it out already" and "that is not yours" are different bugs in the caller. `Open77.effects.remove(fireId)` reaches the same end by a different door: the next tick finds the entry gone and stops the burn with the reason `effect_removed`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `fireId` | `string (decimal) or integer` | required | — |

Returns: `true, or nil`, `reason: permission_denied:world.effects, not_found, owned_by_another_resource, world_unavailable`

```lua
local fireId = Open77.effects.fire(position, { radius = 4.0, damagePerSecond = 5.0, durationMs = 0 })
-- durationMs 0 burns until somebody puts it out; the fire brigade does.
RegisterCommand("extinguish", function()
    local ok, reason = Open77.effects.removeFire(fireId)
    if not ok then print("still burning: " .. tostring(reason)) end
end, true)
```

Registered in `LuaResourceRuntime.Fires.cs` (line 545) as `LuaResourceRuntime.Bootstrap`.

## screen

```lua
Open77.effects.screen(playerId, name, options)
```

`GAME`: Requires a live game instance.

Plays a full-screen post-process effect on one player's own view.

Requires `players.screenfx`. Deliberately NOT replicated -- a drunk screen belongs to the person who drank -- which is why it takes a playerId rather than a position, and why it reaches that client whether or not it runs any resource. `name` false or nil clears every effect this relay owns on that client. `options.strength` (0..1) picks among the authored tiers of the `drunk`, `drugged` and `health` families; `options.duration` is seconds, 0..600, 0 holding until cleared. The alias is checked against a mirror of the client table and refused as `unsupported_screen_effect`; the client's table stays the authority. See the Effects guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `number` | required | — |
| `name` | `string` | required | — |
| `options` | `table` | required | — |

Returns: `true, or false and a reason`

```lua
-- Drinking makes the screen wobble for 30 seconds
RegisterNetEvent("items:drink", function()
    Open77.effects.screen(source, "drunk", { strength = 0.5, duration = 30 })
end)
```

Registered in `LuaResourceRuntime.cs` (line 9808) as `LuaResourceRuntime.Bootstrap`.

## screenCatalog

```lua
Open77.effects.screenCatalog()
```

`GAME`: Requires a live game instance.

The screen-effect aliases this server will accept, ascending.

A mirror of the client's curated table, so a typo is refused by the call that made it rather than by a log line on one player's machine. Includes the tiered family names (`drunk`, `drugged`, `health`) as well as their tiers. See the Effects guide.

Returns: `array of alias strings`

```lua
-- Validate an item's screen effect at startup
local known = {}
for _, alias in ipairs(Open77.effects.screenCatalog()) do known[alias] = true end
assert(known["drunk"], "this server does not support the drunk effect")
```

Registered in `LuaResourceRuntime.cs` (line 9816) as `LuaResourceRuntime.Bootstrap`.

## sound

```lua
Open77.effects.sound(target, event, [options])
```

`NETWORK`: Uses the network backend.

Play spatial sound on a typed target with optional action deduplication.

Requires world.effects. A bare target means player. options={unique=false,duration=5,actionId?,excludePlayers?}; duration is seconds and bounded by the backend. actionId is an opaque resource-scoped identity: repeated delivery of one contact does not replay sound. excludePlayers omits selected listeners, useful when native contact already supplies their audio. Sound targets carry current lifetime identity; no stale-body playback or durable late-stream replay is implied. duration is0.05..60 seconds; actionId is at most192 bytes and deduplication retains it for60 seconds without TTL renewal. excludePlayers accepts at most32 positive IDs; action identities are limited to8192 globally/4096 per resource.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `table|string|integer` | required | — |
| `event` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `true`, `nil, reason on rejection`

```lua
local ok, err = Open77.effects.sound({kind="player",id=tostring(victimId)}, "hit_face", {
 duration=1,actionId=acceptedActionId,excludePlayers={attackerId}
})
if not ok then print(err) end
```

Registered in `LuaResourceRuntime.cs` (line 9786) as `LuaResourceRuntime.Bootstrap`.

## update

```lua
Open77.effects.update(id, patch)
```

`NETWORK`: Uses the network backend.

Patches a looping effect this resource owns.

Requires `world.effects`. Sparse: only the fields present are taken. The effect name is deliberately not patchable -- changing it is a remove and a create, because the running VFX instance is built from it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `decimal string` | required | — |
| `patch` | `table { position?, orientation?, bucket?, visible?, streamingRadius?, streamingHysteresis?, ttlMs? }` | required | — |

Returns: `true, or false`, `reason`

```lua
-- The fire moves with the burning car
Open77.effects.update(fireId, { position = Open77.vehicles.getPosition(carId) })
```

Registered in `LuaResourceRuntime.cs` (line 9766) as `LuaResourceRuntime.Bootstrap`.
