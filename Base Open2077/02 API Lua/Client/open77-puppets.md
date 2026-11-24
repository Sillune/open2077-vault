# Open77.puppets (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-puppets.md) · [Page web](https://open2077.net/docs/api/client/open77-puppets)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## createLocal

```lua
Open77.puppets.createLocal(options)
```

`GAME`: Requires a live game instance.

Create a local character-selection mannequin.

Requires puppets.present and a loaded world. options contains body={family,groups}, position={x,y,z}, optional equipment and yaw in degrees (default 0). Keep face morphs in the captured body. The encoded options must fit in 8 KiB. Creation returns a resource-owned preview handle immediately; poll getLocal for readiness. Limits: 32 per resource, 128 per client, 30-second preparation timeout. Previews are local, idle and removed on resource stop/reload or world teardown. See [Character previews](https://open2077.net/docs/api/client/character-previews.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `integer preview handle, or nil`, `reason on failure`

```lua
local preview, reason = Open77.puppets.createLocal({
    body = savedLook.body, equipment = savedLook.equipment,
    position = { x = -699, y = 2164, z = 53.1 }, yaw = 90,
})
```

[Character previews & player-shaped NPCs](../../01%20Documentation/character-previews.md)

Registered in `ResourceHost.cpp` (line 17283) as `LuaPuppetsCreateLocal`.

## getLocal

```lua
Open77.puppets.getLocal(handle)
```

`GAME`: Requires a live game instance.

Read a local mannequin preparation state.

Requires puppets.present. Accepts a preview handle owned by this resource generation, not a player ID or server NPC ID. Returns id, entity, ready, status and reason. status is spawning, preparing, ready or failed. Use entity for camera/scene integration only after ready is true. See [Character previews](https://open2077.net/docs/api/client/character-previews.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `integer` | required | — |

Returns: `table, or nil`, `reason on failure`

```lua
local state, reason = Open77.puppets.getLocal(preview)
if state and state.ready then print(state.entity) end
```

[Character previews & player-shaped NPCs](../../01%20Documentation/character-previews.md)

Registered in `ResourceHost.cpp` (line 17320) as `LuaPuppetsGetLocal`.

## removeLocal

```lua
Open77.puppets.removeLocal(handle)
```

`GAME`: Requires a live game instance.

Remove a local mannequin or cancel its pending spawn.

Requires puppets.present and a handle owned by this resource generation. Remove the old preview before changing a selection, outfit or placement. Resource stop/reload and world teardown also clean up previews. Unknown and foreign handles are refused. See [Character previews](https://open2077.net/docs/api/client/character-previews.md).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `handle` | `integer` | required | — |

Returns: `boolean removed`, `reason on failure`

```lua
local removed, reason = Open77.puppets.removeLocal(preview)
```

[Character previews & player-shaped NPCs](../../01%20Documentation/character-previews.md)

Registered in `ResourceHost.cpp` (line 17340) as `LuaPuppetsRemoveLocal`.

## setBody

```lua
Open77.puppets.setBody(playerId, body)
```

`GAME`: Requires a live game instance.

Project a captured body onto a network player proxy.

Requires puppets.present. playerId is a network player ID; body={family,groups}, or false to clear the remote body. Uses asynchronous native dressing; true means acceptance, not a fully rendered proxy. Use the appearance service to distribute authoritative records and preserve wardrobe ownership.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `body` | `table|false` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
local ok, err = Open77.puppets.setBody(playerId, capturedBody)
if not ok then print(err) end
```

Registered in `ResourceHost.cpp` (line 24827) as `LuaPuppetsSetBody`.

## setSlot

```lua
Open77.puppets.setSlot(playerId, slot, [record])
```

`GAME`: Requires a live game instance.

Project one remote equipment-slot override.

Requires puppets.present. playerId is a network player ID. record is a TweakDB record string (up to160 bytes), false for explicitly empty, or nil for the native default. Do not confuse nil and false when restoring a clothing snapshot. Native dressing is asynchronous; this is not durable inventory ownership.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `slot` | `string` | required | — |
| `record` | `string|false|nil` | optional | — |

Returns: `boolean accepted`, `reason on failure`

```lua
local ok, err = Open77.puppets.setSlot(playerId, "OuterChest", false)
if not ok then print(err) end
```

Registered in `ResourceHost.cpp` (line 24828) as `LuaPuppetsSetSlot`.

## setWardrobe

```lua
Open77.puppets.setWardrobe(playerId, snapshot)
```

`GAME`: Requires a live game instance.

Project a remote wardrobe snapshot.

Requires puppets.present. snapshot={active=integer,outfits={[outfitId]={[slot]=recordOrFalse}}}. false means an explicitly empty slot. Network player IDs and native wardrobe IDs are different namespaces. This is asynchronous presentation; the server wardrobe service remains the durable source of truth.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer` | required | — |
| `snapshot` | `table` | required | — |

Returns: `boolean accepted`, `reason on failure`

```lua
local ok, err = Open77.puppets.setWardrobe(playerId, wardrobeSnapshot)
if not ok then print(err) end
```

Registered in `ResourceHost.cpp` (line 24829) as `LuaPuppetsSetWardrobe`.
