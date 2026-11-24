# Open77.vfx (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-vfx.md) · [Page web](https://open2077.net/docs/api/client/open77-vfx)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

12 functions.

## attach

```lua
Open77.vfx.attach(effectId, entity, slot, [anchor])
```

`GAME`: Requires a live game instance.

Attach an owned world effect to a local native body slot.

Requires world.effects. effectId and entity are local handles; slot is a nonempty native slot name up to128 bytes. anchor defaults to body; weaponRight uses the supported equipped weapon anchor. Missing body/slot/weapon is a refusal, not permission to bind an unrelated target. True is native attachment acceptance, not visual proof.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `effectId` | `string|integer` | required | — |
| `entity` | `string|integer` | required | — |
| `slot` | `string` | required | — |
| `anchor` | `string` | optional | — |

Returns: `boolean accepted`, `reason on failure`

```lua
local entity, err = Open77.vfx.resolveTarget({kind="player",id=tostring(playerId)})
if not entity then print(err); return end
local ok, reason = Open77.vfx.attach(effectId, entity, "RightHand", "body")
if not ok then print(reason) end
```

Registered in `ResourceHost.cpp` (line 25536) as `LuaVfxAttach`.

## catalog

```lua
Open77.vfx.catalog()
```

`GAME`: Requires a live game instance.

Returns the curated stable world-VFX aliases and cooked paths.

Catalog access does not start an effect. Raw cooked paths remain build-dependent even when present.

Returns: `table mapping alias to effect path, or nil`, `reason`

```lua
-- Print every alias this client can play
for alias, path in pairs(Open77.vfx.catalog() or {}) do print(alias, path) end
```

Registered in `ResourceHost.cpp` (line 25547) as `LuaVfxCatalog`.

## clear

```lua
Open77.vfx.clear()
```

`GAME`: Requires a live game instance.

Stops every VFX owned by the calling resource.

Requires `world.effects`; SFX and other resource owners are unaffected.

Returns: `true on success, otherwise false`, `reason`

```lua
-- The fireworks show is over
AddEventHandler("fireworks:ended", function()
    Open77.vfx.clear()
end)
```

Registered in `ResourceHost.cpp` (line 25542) as `LuaEffectClear`.

## list

```lua
Open77.vfx.list()
```

`GAME`: Requires a live game instance.

Lists active VFX owned by the calling resource.

Requires `world.effects`.

Returns: `array of { id, kind, name, entity, remaining }, or nil`, `reason`

```lua
-- Debug: which effects does this resource still own?
for _, fx in ipairs(Open77.vfx.list() or {}) do print(fx.id, fx.name, fx.remaining) end
```

Registered in `ResourceHost.cpp` (line 25545) as `LuaEffectList`.

## play

```lua
Open77.vfx.play(effect, options)
```

`GAME`: Requires a live game instance.

Starts a resource-owned world VFX at a fixed transform.

Requires `world.effects`. `options` requires `position` and accepts `orientation`, `duration`, and `ignoreTimeDilation`. The effect is released automatically with its owner.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `effect` | `string` | required | — |
| `options` | `table` | required | — |

Returns: `effect handle string, or nil`, `reason`

```lua
-- Local smoke at the campfire for 30 seconds
Open77.vfx.play("smoke.exterior", { position = { x = -1460.2, y = 99.9, z = 14.8 }, duration = 30 })
```

Registered in `ResourceHost.cpp` (line 25534) as `LuaVfxPlay`.

## playEntity

```lua
Open77.vfx.playEntity(effect, [options])
```

`GAME`: Requires a live game instance.

Starts a resource-owned authored VFX on an entity.

Requires `world.effects`. Options accept `entity`, `instance`, `persistOnDetach`, `breakAllLoops`, `breakAllOnDestroy`, and `duration`; omitting `entity` targets the local player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `effect` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `effect handle string, or nil`, `reason`

```lua
-- Sparks on the local player's body when a hack lands
RegisterNetEvent("hack:hit", function()
    Open77.vfx.playEntity("sparks.burst.small", { duration = 2 })
end)
```

Registered in `ResourceHost.cpp` (line 25539) as `LuaVfxPlayEntity`.

## resolveTarget

```lua
Open77.vfx.resolveTarget(target)
```

`GAME`: Requires a live game instance.

Resolve a typed network target to its currently streamed local handle.

Requires world.effects. Accepts {kind="player"|"npc"|"vehicle"|"prop",id=decimalStringOrInteger}. Returns a decimal string local registry handle (local player resolves to1), or nil, reason such as target_not_streamed. Re-resolve after stream/body changes; never send this local handle over the network.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `target` | `table` | required | — |

Returns: `decimal string local handle`, `nil, reason on rejection`

```lua
local handle, err = Open77.vfx.resolveTarget({kind="player",id=tostring(playerId)})
if handle then print(handle) else print(err) end
```

Registered in `ResourceHost.cpp` (line 25537) as `LuaVfxResolveTarget`.

## screen

```lua
Open77.vfx.screen(name, options)
```

`GAME`: Requires a live game instance.

Plays one of Cyberpunk's full-screen post-process effects on the local view.

Client-only; requires `vfx.screen`, which is deliberately NOT `world.effects` -- covering somebody's eyes is a different power from putting smoke in a street. `name` must be an alias from `Open77.vfx.screenCatalog()`; anything else, including the engine's own effect name, is refused with `unsupported_screen_effect` and never forwarded, because an unknown authored effect name is a SILENT no-op in the engine. `options.strength` (0..1, default 1.0) picks among the tiers the game authored for the `drunk`, `drugged` and `health` families and is ignored by single-tier aliases: 2.31 has no continuous amount channel. `options.duration` is seconds, 0..600; 0 holds until stopped. The handle is an ordinary effect handle -- `Open77.vfx.stop`, `list` and `clear` all work on it. See the Effects guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `name` | `string` | required | — |
| `options` | `table` | required | — |

Returns: `effect handle string, or nil`, `reason`

```lua
-- The player drank too much: medium drunk wobble for 30 seconds
Open77.vfx.screen("drunk", { strength = 0.5, duration = 30 })
```

Registered in `ResourceHost.cpp` (line 25550) as `LuaVfxScreen`.

## screenCatalog

```lua
Open77.vfx.screenCatalog()
```

`GAME`: Requires a live game instance.

The curated screen-effect aliases and the engine effect name behind each.

Client-only and ungated, like `Open77.vfx.catalog`: knowing the vocabulary is not the power, playing it is. Returns `{ [alias] = engineEffectName }`. Tiered families appear as their tiers (`drunk.light`, `drunk.medium`, `drunk.heavy`), never as the bare family name -- the family is an input convenience for `strength`, the tiers are what exists. See the Effects guide.

Returns: `table of alias to engine effect name`

```lua
-- Check an alias before using it
local aliases = Open77.vfx.screenCatalog()
if aliases["drunk.heavy"] then print("heavy drunk available") end
```

Registered in `ResourceHost.cpp` (line 25551) as `LuaVfxScreenCatalog`.

## setParameter

```lua
Open77.vfx.setParameter(effectId, name, value)
```

`GAME`: Requires a live game instance.

Set a scalar parameter on an owned native world effect.

Requires world.effects. Parameter name is nonempty, at most128 bytes and contains no NUL; value must be finite. The native graph must actually expose the requested parameter. Acceptance does not guarantee visible change and does not modify authoritative gameplay.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `effectId` | `string|integer` | required | — |
| `name` | `string` | required | — |
| `value` | `number` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
-- Only use a parameter authored by your chosen effect graph.
local ok, err = Open77.vfx.setParameter(effectId, parameterName, 0.5)
if not ok then print(err) end
```

Registered in `ResourceHost.cpp` (line 25538) as `LuaVfxSetParameter`.

## stop

```lua
Open77.vfx.stop(id)
```

`GAME`: Requires a live game instance.

Stops one VFX handle owned by the calling resource.

Requires `world.effects`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Stop the smoke when the fire is put out
local smoke = Open77.vfx.play("smoke.exterior", { position = campfire })
AddEventHandler("fire:extinguished", function() Open77.vfx.stop(smoke) end)
```

Registered in `ResourceHost.cpp` (line 25540) as `LuaEffectStop`.

## update

```lua
Open77.vfx.update(effectId, options)
```

`GAME`: Requires a live game instance.

Move an owned native world effect.

Requires world.effects. options.position is a finite{x,y,z}; optional orientation{x,y,z,w} must be a nonzero quaternion and is normalized. Updates an existing owned world effect, not the registry target or TTL; use server attached effects for replicated lifetime.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `effectId` | `string|integer` | required | — |
| `options` | `table` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
local ok, err = Open77.vfx.update(effectId, {position={x=0,y=0,z=1}})
if not ok then print(err) end
```

Registered in `ResourceHost.cpp` (line 25535) as `LuaVfxUpdate`.
