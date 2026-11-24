# Open77.loot (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-loot.md) · [Page web](https://open2077.net/docs/api/server/open77-loot)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## all

```lua
Open77.loot.all([bucket])
```

`NETWORK`: Uses the network backend.

Every loot drop, optionally filtered to one routing bucket.

Requires `world.loot`. Returns the whole registry, whichever resource created each drop.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | optional | — |

Returns: `array of drop snapshots`

```lua
-- Count the drops still lying in the arena
print(#Open77.loot.all(ARENA_BUCKET), "drops on the ground")
```

Registered in `LuaResourceRuntime.cs` (line 8997) as `LuaResourceRuntime.Bootstrap`.

## create

```lua
Open77.loot.create(def)
```

`NETWORK`: Uses the network backend.

Creates an authoritative ground loot drop.

Requires `world.loot`. The definition accepts `item`, `quantity` (default 1), `position`, `bucket`, `radius` (default 2), `label`, `visualItem`/`model` and `ttlMs`. Drops are replicated and owned by the creating resource, so they survive a player walking away and are cleaned up when the resource stops. See [loot](https://open2077.net/docs/api/server/loot.md) for pickup validation and client projection.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `def` | `table { item, quantity?, position, bucket?, radius?, label?, visualItem?, ttlMs? }` | required | — |

Returns: `loot id, or nil`, `reason`

```lua
-- A dead player drops 200 eddies where they fell, gone after five minutes
AddEventHandler("onPlayerLifeStateChanged", function(playerId, revision, phase)
    if phase ~= "dead" then return end
    local pos = Open77.players.position(playerId)
    if pos then
        Open77.loot.create({ item = "Items.money", quantity = 200, position = pos, label = "Eddies", ttlMs = 300000 })
    end
end)
```

Registered in `LuaResourceRuntime.cs` (line 8979) as `LuaResourceRuntime.Bootstrap`.

## get

```lua
Open77.loot.get(id)
```

`NETWORK`: Uses the network backend.

Reads one loot drop's snapshot.

Requires `world.loot`. Answers `nil` for an unknown or already collected id.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `loot id` | required | — |

Returns: `drop snapshot, or nil`

```lua
-- Is the drop still there before paying out?
local drop = Open77.loot.get(lootId)
if not drop then print("already picked up") end
```

Registered in `LuaResourceRuntime.cs` (line 8996) as `LuaResourceRuntime.Bootstrap`.

## remove

```lua
Open77.loot.remove(id)
```

`NETWORK`: Uses the network backend.

Removes a loot drop.

Requires `world.loot`. Use it when a mode ends a round rather than waiting for `ttlMs`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `loot id` | required | — |

Returns: `boolean`

```lua
-- Round over: clear the leftover drops
for _, drop in ipairs(Open77.loot.all(ARENA_BUCKET)) do Open77.loot.remove(drop.id) end
```

Registered in `LuaResourceRuntime.cs` (line 8995) as `LuaResourceRuntime.Bootstrap`.

## update

```lua
Open77.loot.update(id, def)
```

`NETWORK`: Uses the network backend.

Patches an existing loot drop.

Requires `world.loot`. Reads the current drop first and keeps every field the patch omits, so moving a drop does not silently reset its quantity or label. An unknown id answers `false`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `loot id` | required | — |
| `def` | `table, same fields as create` | required | — |

Returns: `boolean`

```lua
-- The wind blows the crate two metres; quantity and label stay
Open77.loot.update(lootId, { position = { x = -1444.2, y = 127.4, z = 18.0 } })
```

Registered in `LuaResourceRuntime.cs` (line 8985) as `LuaResourceRuntime.Bootstrap`.
