# Open77.sfx (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-sfx.md) · [Page web](https://open2077.net/docs/api/client/open77-sfx)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## clear

```lua
Open77.sfx.clear()
```

`GAME`: Requires a live game instance.

Stops every SFX owned by the calling resource.

Requires `world.effects`; VFX and other resource owners are unaffected.

Returns: `true on success, otherwise false`, `reason`

```lua
-- Stop every sound this resource plays when the mini-game ends
AddEventHandler("minigame:ended", function() Open77.sfx.clear() end)
```

Registered in `ResourceHost.cpp` (line 25560) as `LuaEffectClear`.

## list

```lua
Open77.sfx.list()
```

`GAME`: Requires a live game instance.

Lists active SFX owned by the calling resource.

Requires `world.effects`.

Returns: `array of { id, kind, name, entity, remaining }, or nil`, `reason`

```lua
-- Debug: which sounds are still playing?
for _, sfx in ipairs(Open77.sfx.list() or {}) do print(sfx.id, sfx.name, sfx.remaining) end
```

Registered in `ResourceHost.cpp` (line 25563) as `LuaEffectList`.

## play

```lua
Open77.sfx.play(event, [options])
```

`GAME`: Requires a live game instance.

Starts resource-owned spatial audio on an entity.

Requires `world.effects`. Options accept `entity`, `emitter`, `tag`, `seekTime`, `duration`, and `unique`; omitting `entity` uses the local player.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |
| `options` | `table` | optional | — |

Returns: `effect handle string, or nil`, `reason`

```lua
-- A Wwise event on the local player when the hack lands
RegisterNetEvent("hack:hit", function()
    Open77.sfx.play("ui_hacking_access_denied", { duration = 2 })
end)
```

Registered in `ResourceHost.cpp` (line 25555) as `LuaSfxPlay`.

## play2d

```lua
Open77.sfx.play2d(event)
```

`GAME`: Requires a live game instance.

Plays a non-spatialised (2D) Wwise event -- the frontend door.

Client-only; requires `world.effects`. `Open77.sfx.play` posts the sound at an entity, so it always comes from somewhere in the world; a user-interface click comes from nowhere. This posts through `gameGameAudioSystem::Play` with no entity and no emitter, the same call vanilla scripts make for menu sounds. Returns `true` rather than a handle: a frontend one-shot has no emitter to address a stop to, so it is untracked, unstoppable and consumes no effect quota. There is no `volume` and no `pitch` option here or on `play` -- REDengine's audio events carry neither field; the only float in the surface is a bank-authored Wwise RTPC, not a gain. Use `open77_sound` when you need to control loudness or pitch.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `event` | `string` | required | — |

Returns: `true on success, otherwise nil`, `failure reason`

```lua
-- A menu click that comes from nowhere
AddEventHandler("menu:selected", function()
    Open77.sfx.play2d("ui_menu_onpress")
end)
```

Registered in `ResourceHost.cpp` (line 25556) as `LuaSfxPlay2d`.

## playVoice

```lua
Open77.sfx.playVoice(voice, options)
```

`GAME`: Requires a live game instance.

Plays one voice-over line from an entity's own voiceset -- a bark, not a sound-bank event.

Client-only; requires `world.effects`. `voice` is a `voContext` name, the word vanilla passes to `GameObject.PlayVoiceOver` (`greeting`, `fear_beg`, `start_combat`), resolved by the engine against the entity's voiceset; this queues the engine's own `SoundPlayVo` (`entGameplayVOEvent`) on the body. It is the executor behind the server's `Open77.npcs.speak`: the bundled `open77_effects` resource calls it when the one-shot arrives, on every client that has the body streamed, because a voice-over is audible only from a puppet instance. Used directly it is a purely local line. `ignoreFrustum` (default true) speaks off-screen; `ignoreDistance` (default false) skips the engine's distance cull. Returns `true` rather than a handle -- a line cannot be stopped by handle and consumes no quota. Refusals: `invalid_voice`, `options_must_be_a_table`, `invalid_entity`, `entity_unavailable`, `voice_disabled` (the NPC's replicated behaviour policy has its voice off), plus the facade's `world_unavailable`, `native_unavailable`, `invocation_failed`. A name the voiceset does not carry is dropped by the engine without a word.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `voice` | `string` | required | — |
| `options` | `table { entity, ignoreFrustum?, ignoreDistance? }` | required | — |

Returns: `true on success, otherwise nil`, `failure reason`

```lua
-- A local bark from the NPC under the reticle.
local target = Open77.inspector.target()
if target and target.kind == "npc" then
    Open77.sfx.playVoice("greeting", { entity = target.entity })
end
```

Registered in `ResourceHost.cpp` (line 25557) as `LuaSfxPlayVoice`.

## stop

```lua
Open77.sfx.stop(id)
```

`GAME`: Requires a live game instance.

Stops one SFX handle owned by the calling resource.

Requires `world.effects`. REDengine stops audio by event name, so identical simultaneous events on one entity may share stop behavior.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Stop the alarm loop on the car
local alarm = Open77.sfx.play("v_car_alarm", { entity = carEntity })
AddEventHandler("alarm:off", function() Open77.sfx.stop(alarm) end)
```

Registered in `ResourceHost.cpp` (line 25558) as `LuaEffectStop`.
