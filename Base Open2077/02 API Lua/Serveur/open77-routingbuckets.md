# Open77.routingBuckets (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-routingbuckets.md) · [Page web](https://open2077.net/docs/api/server/open77-routingbuckets)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

6 functions.

## getEntity

```lua
Open77.routingBuckets.getEntity(entityId)
```

`NETWORK`: Uses the network backend.

An entity's routing bucket.

No permission. `GetEntityRoutingBucket` is the same function. The default is `0`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entityId` | `entity id` | required | — |

Returns: `bucket (integer)`

```lua
-- Which instance is this vehicle in?
print("vehicle bucket", Open77.routingBuckets.getEntity(vehicleEntityId))
```

Registered in `LuaResourceRuntime.cs` (line 9845) as `LuaResourceRuntime.Bootstrap`.

## getPlayer

```lua
Open77.routingBuckets.getPlayer(playerId)
```

`NETWORK`: Uses the network backend.

A player's routing bucket.

No permission. `GetPlayerRoutingBucket` is the same function. The default is `0`. A bucket is a visibility scope: players and entities only see others in the same one.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |

Returns: `bucket (integer)`

```lua
-- Only players in the same bucket can trade
if Open77.routingBuckets.getPlayer(a) ~= Open77.routingBuckets.getPlayer(b) then
    return Open77.chat.send(a, "You are not in the same instance.")
end
```

Registered in `LuaResourceRuntime.cs` (line 9843) as `LuaResourceRuntime.Bootstrap`.

## setEntity

```lua
Open77.routingBuckets.setEntity(entityId, bucket)
```

`NETWORK`: Uses the network backend.

Moves an entity to another routing bucket.

No permission. `SetEntityRoutingBucket` is the same function. The per-kind helpers (`Open77.vehicles.update`, `Open77.npcs.setBucket`, `Open77.props.setBucket`) do the same thing with validation.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `entityId` | `entity id` | required | — |
| `bucket` | `integer` | required | — |

Returns: `boolean`

```lua
-- Move the entity into the arena instance
Open77.routingBuckets.setEntity(entityId, ARENA_BUCKET)
```

Registered in `LuaResourceRuntime.cs` (line 9846) as `LuaResourceRuntime.Bootstrap`.

## setLockdownMode

```lua
Open77.routingBuckets.setLockdownMode(bucket, mode)
```

`NETWORK`: Uses the network backend.

Sets a bucket's entity lockdown mode.

No permission. `SetRoutingBucketEntityLockdownMode` is the same function. Modes are `inactive`, `relaxed`, `strict` and `full`, tightening what clients in that bucket may create.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |
| `mode` | `string (inactive|relaxed|strict|full)` | required | — |

Returns: `boolean`

```lua
-- Nothing client-created may exist in the competitive arena
Open77.routingBuckets.setLockdownMode(ARENA_BUCKET, "strict")
```

Registered in `LuaResourceRuntime.cs` (line 9847) as `LuaResourceRuntime.Bootstrap`.

## setPlayer

```lua
Open77.routingBuckets.setPlayer(playerId, bucket)
```

`NETWORK`: Uses the network backend.

Moves a player to another routing bucket.

No permission. `SetPlayerRoutingBucket` is the same function. This moves their authoritative visibility scope, so it is how a lobby, an instanced arena or a private apartment is built.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `integer|string` | required | — |
| `bucket` | `integer` | required | — |

Returns: `boolean`

```lua
-- Each apartment is its own instance: bucket 1000 + house id
RegisterNetEvent("house:enter", function(houseId)
    Open77.routingBuckets.setPlayer(source, 1000 + houseId)
end)
```

Registered in `LuaResourceRuntime.cs` (line 9844) as `LuaResourceRuntime.Bootstrap`.

## setPopulationEnabled

```lua
Open77.routingBuckets.setPopulationEnabled(bucket, enabled)
```

`NETWORK`: Uses the network backend.

Turns ambient population on or off for one bucket.

No permission. `SetRoutingBucketPopulationEnabled` is the same function. Switching it off empties a bucket of vanilla pedestrians and traffic, which is what an arena or a staged scene usually wants.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `bucket` | `integer` | required | — |
| `enabled` | `boolean` | required | — |

Returns: `boolean`

```lua
-- No traffic or crowd in the arena instance
Open77.routingBuckets.setPopulationEnabled(ARENA_BUCKET, false)
```

Registered in `LuaResourceRuntime.cs` (line 9848) as `LuaResourceRuntime.Bootstrap`.
