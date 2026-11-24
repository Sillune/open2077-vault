# Open77.npcs (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-npcs.md) · [Page web](https://open2077.net/docs/api/client/open77-npcs)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

6 functions.

## all

```lua
Open77.npcs.all([options])
```

`GAME`: Requires a live game instance.

Lists the NPCs this client can see, Open77-owned and optionally vanilla.

With no argument this is the server-owned set in this client's interest set, unchanged: every row has an `id` the server agrees with, and `source` is `"open77"`. Canonical creation, tasks, health and removal are server-only; see [NPCs](https://open2077.net/docs/api/client/npcs.md).

`includeForeign = true` additionally lists the **vanilla population peds** standing around the local player. Those rows are deliberately a different shape -- `source = "population"`, `kind = "populationNpc"`, no `id`, no `health`, no `bucket` -- because nothing replicates them. They are here so a targeting resource can see the whole world, not so anything can be commanded. The extra reach costs a world query and its own permission, `npcs.foreign`, and is refused by name without it.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table { includeForeign?, radius? (metres, default 50) }` | optional | — |

Returns: `array of NPC snapshots (owned) and population rows, or nil`, `reason on failure`

```lua
for _, row in ipairs(Open77.npcs.all({ includeForeign = true, radius = 25 })) do
    if row.source == "open77" then print("ours", row.id)
    else print("scenery", row.engineEntity) end
end
```

[NPC lifecycle & read-only client guide](../../01%20Documentation/npcs.md)

Registered in `ResourceHost.cpp` (line 25124) as `LuaNpcsAll`.

## currentTask

```lua
Open77.npcs.currentTask(id)
```

`GAME`: Requires a live game instance.

Returns the canonical active task ID for an NPC.

Returns the last Open77 NPC task snapshot currently presented for a network NPC. It requires `npcs.read`; task state is ephemeral and can disappear when the NPC leaves the streaming set.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `task id, or nil`

```lua
-- Debug: which task is this NPC presenting?
print("task", Open77.npcs.currentTask(npcId))
```

[NPC lifecycle & read-only client guide](../../01%20Documentation/npcs.md)

Registered in `ResourceHost.cpp` (line 25127) as `LuaNpcCurrentTask`.

## entity

```lua
Open77.npcs.entity(id)
```

`GAME`: Requires a live game instance.

Returns the ephemeral local Open77 entity handle for an NPC projection.

Returns nil before readiness and after stream-out. Never cache this handle across lifecycle changes.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `entity, or nil`

```lua
-- Read a server NPC's body through its entity handle
local entity = Open77.npcs.entity(npcId)
if entity then print(Open77.character.state(entity).position.x) end
```

[NPC lifecycle & read-only client guide](../../01%20Documentation/npcs.md)

Registered in `ResourceHost.cpp` (line 25126) as `LuaNpcEntity`.

## get

```lua
Open77.npcs.get(id)
```

`GAME`: Requires a live game instance.

Reads one server-owned NPC currently known by this client.

Read-only and guarded by npcs.read. The snapshot includes the exact Character record; template is the legacy alias or 'record' for direct spawns. The local entity handle is generation-checked and becomes invalid after stream-out. Observe onNpcReady/onNpcSpawnFailed for asynchronous projection readiness; see wiki/npcs.md. Stable 156 snapshots include armor. Server create accepts armor (default 0), update preserves it when omitted; finite 0..100000 points absorb canonical damage before health. Matching protocol 1.50 is required. See [NPC armor](../../01%20Documentation/npcs.md#armor).

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `NPC snapshot, or nil`

```lua
-- Show the NPC's record name over its head
local npc = Open77.npcs.get(npcId)
if npc then Open77.hud.notify(npc.record) end
```

[NPC lifecycle & read-only client guide](../../01%20Documentation/npcs.md)

Registered in `ResourceHost.cpp` (line 25123) as `LuaNpcGet`.

## isStreamedIn

```lua
Open77.npcs.isStreamedIn(id)
```

`GAME`: Requires a live game instance.

Whether an NPC projection is attached and ready locally.

Reports whether a network NPC currently has a live client proxy in this player's streaming scope. It does not request streaming or prove that the server NPC record no longer exists.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `integer` | required | — |

Returns: `boolean`

```lua
-- Only draw the marker when the NPC has a body on this client
if Open77.npcs.isStreamedIn(npcId) then drawMarker(npcId) end
```

[NPC lifecycle & read-only client guide](../../01%20Documentation/npcs.md)

Registered in `ResourceHost.cpp` (line 25125) as `LuaNpcIsStreamedIn`.

## whenReady

```lua
Open77.npcs.whenReady(id, [timeoutMs])
```

`GAME`: Requires a live game instance.

A promise that resolves once this client's projection of an NPC is attached and behaving.

Requires `npcs.read`. The NPC twin of `Open77.vehicles.whenStreamed`: resolves with `{ id, entity }` once `Open77.npcs.isStreamedIn(id)` answers true -- the same instant `onNpcReady` fires, which is the projection attached AND its behaviour applied, not merely the create received -- and rejects with `timeout` after `timeoutMs` (default 15000, clamped to 1..120000). Listens for `onNpcReady` and polls every 100 ms; the one-shot handler is removed on resolve, on timeout and with the resource. `nil, reason` only for a bad id, a bad timeout or a missing permission; an unknown id is waited for and times out.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `NPC id` | required | — |
| `timeoutMs` | `number` | optional | — |

Returns: `Promise, or nil`, `reason: invalid_npc_id, invalid_timeout, permission_denied:npcs.read, promise_unavailable`

```lua
-- Play the greeting emote only when the vendor has a body on this client
RegisterNetEvent("shop:vendorSpawned", function(npcId)
    Open77.npcs.whenReady(npcId, 8000):next(function(body)
        Open77.animations.play(body.entity, "wave")
    end, function(reason)
        print("vendor never appeared: " .. reason)
    end)
end)
```

[NPC lifecycle & read-only client guide](../../01%20Documentation/npcs.md)

Registered in `ResourceHost.cpp` (line 23338) as `LuaPrelude`.
