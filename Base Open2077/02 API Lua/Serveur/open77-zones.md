# Open77.zones (server runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/server/open77-zones.md) · [Page web](https://open2077.net/docs/api/server/open77-zones)

Runs on the dedicated server and is authoritative: state written here is the truth every client is told about.

5 functions.

## bounds

```lua
Open77.zones.bounds(zone)
```

`SHARED`: Available without a live game instance.

The zone's bounding volume: a planar centre and radius plus an optional height band.

Available in both runtimes, from the shared prelude scripting/lua/open77_zones.lua. Exactly the shape of the one squared comparison a poll loop wants per zone per tick, and of the radius a server roster scan wants. `minZ` and `maxZ` are nil when the zone is unbounded on that side, which only a polygon with no declared height range can be. This is what makes a polygon affordable: a zone the player is not standing in is rejected by its bounds for about the cost of a sphere (1.25 us against 1.2 us, measured in the client host), so cost tracks the zones that actually overlap the player rather than the number registered.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `zone` | `table (a definition, or a value from Open77.zones.normalize)` | required | — |

Returns: `table { x, y, z, radius, minZ, maxZ } (minZ and maxZ may be nil), or nil, reason`, `reason: the same tokens as Open77.zones.contains`

```lua
local b = assert(Open77.zones.bounds(turf))
print(("turf spans %.1f m around %.1f, %.1f"):format(b.radius, b.x, b.y))
```

[Proximity zones guide](../../01%20Documentation/zones.md)

Registered in `open77_zones.lua` (line 628) as `LuaPrelude`.

## contains

```lua
Open77.zones.contains(zone, point, [options])
```

`SHARED`: Available without a live game instance.

Is a point inside a zone? The one containment implementation both runtimes share.

Available in both runtimes, from the shared prelude scripting/lua/open77_zones.lua. The client poller and the server's Open77.zones.playersIn both call this, which is the point: a client and a server that disagreed at a boundary would leave a place a player can stand where the prompt appears but the grant is refused, or worse the other way round. Shapes: omit `shape` for the cylinder that shipped (planar `radius` around `position` plus a `maxHeight` band, default 6 m); `sphere` is a true 3-D ball and refuses `maxHeight`; `box` takes `position`, `size` (FULL extents) and `rotation` in degrees about the vertical axis; `poly` (alias `polygon`) takes `points` (3..512 planar vertices, concave allowed) with optional `minZ`/`maxZ` -- absent means unbounded on that side; `combo` takes `zones`, nested up to 4 deep. Every shape is CLOSED: a point on an edge, a face or a vertex is inside. `point` is a vector3 or any { x =, y =, z = } table. `options` may carry `grace` (metres to expand the zone outward, with rounded corners -- the server's tolerance for a replicated position, and the client's hysteresis band), `origin` ({ x, y, z } where the zone's anchor actually is right now, which is how an entity-attached zone is tested) and `rotation` (degrees, the zone's heading right now). A definition carrying `attach` REQUIRES `origin`, because resolving an entity is not something pure geometry may do. `false` always means outside and never 'could not tell'. Pure: it reads no world and holds no state. See the Proximity zones guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `zone` | `table (a definition, or a value from Open77.zones.normalize)` | required | — |
| `point` | `table { x, y, z }` | required | — |
| `options` | `table { grace?, origin?, rotation? }` | optional | — |

Returns: `boolean, or nil, reason`, `reason: invalid_definition, unknown_shape, invalid_position, invalid_radius, invalid_max_height, max_height_unsupported, invalid_size, invalid_rotation, invalid_points, too_many_points, degenerate_polygon, invalid_height_range, invalid_zones, too_many_zones, zone_nesting_too_deep, invalid_attach, invalid_point, invalid_options, invalid_grace, invalid_origin, origin_required`

```lua
-- Pure geometry on a point the server already holds: keep the airdrop inside
-- the play area. (For "is this player standing in it", use containsPlayer,
-- which adds the routing-bucket and snapshot-freshness rules.)
local arena = { shape = "poly", minZ = 12.0, maxZ = 20.0, points = {
    { x = -1460.0, y = 90.0 }, { x = -1440.0, y = 90.0 },
    { x = -1440.0, y = 110.0 }, { x = -1460.0, y = 110.0 },
} }
local drop = Open77.vehicles.getPosition(cargoId)
if drop and Open77.zones.contains(arena, drop) then
    Open77.loot.create({ item = "Items.money", quantity = 500, position = drop })
end
```

[Proximity zones guide](../../01%20Documentation/zones.md)

Registered in `open77_zones.lua` (line 626) as `LuaPrelude`.

## containsPlayer

```lua
Open77.zones.containsPlayer(playerId, definition, [options])
```

`NETWORK`: Uses the network backend.

Server only. The single-player containment gate a grant should run before it acts.

Three answers, and they are genuinely different: true, false, and nil with a reason when the answer is not knowable. Refusing on nil is a policy decision the caller has to make, not one this call may make silently. A player in another routing bucket is `false` -- a different bucket is a different world, so they are not standing in this zone by any reading. A player who has never reported a position is nil/position_unknown, which is not 'outside'. Pass `maxAgeMs` when a stale reading must not grant: without it the last known position is used however old it is, matching Open77.players.get and Open77.players.nearby. `grace`, `origin` and `rotation` behave as they do on Open77.zones.playersIn. Use this for one player and playersIn for a sweep. See the Proximity zones guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `playerId` | `number` | required | — |
| `definition` | `table (a zone definition, or a value from Open77.zones.normalize)` | required | — |
| `options` | `table { bucket?, maxAgeMs?, grace?, origin?, rotation? }` | optional | — |

Returns: `boolean, or nil, reason`, `reason: player_not_found, position_unknown, position_stale, invalid_options, invalid_max_age, or any Open77.zones.contains token including origin_required`

```lua
-- The shop only sells to a player really standing inside it
local shop = Open77.zones.normalize({ shape = "box", position = { x = -1460.2, y = 99.9, z = 14.8 }, size = { x = 12, y = 6, z = 4 } })
RegisterNetEvent("shop:buy", function(itemId)
    local inside = Open77.zones.containsPlayer(source, shop, { maxAgeMs = 2000 })
    if not inside then return Open77.chat.send(source, "You are not in the shop.") end
    sell(source, itemId)
end)
```

[Proximity zones guide](../../01%20Documentation/zones.md)

Registered in `LuaResourceRuntime.cs` (line 8493) as `LuaResourceRuntime.Bootstrap`.

## normalize

```lua
Open77.zones.normalize(definition)
```

`SHARED`: Available without a live game instance.

Validates a zone definition once and precomputes everything a containment test needs.

Available in both runtimes, from the shared prelude scripting/lua/open77_zones.lua. Returns a prepared zone -- a plain table, so it still survives json.encode and pairs -- carrying the validated shape, its bounding volume and the derived values a test would otherwise recompute. Open77.zones.contains accepts either form, but on a raw definition it has to redo this work on every single call: measured in the client host, a 32-vertex polygon costs about 9.5 us prepared and roughly eight times that unprepared, because the vertex list is rebuilt and the shoelace area re-run before the test can start. Normalize once, keep the result, test many. Shapes: omit `shape` for the cylinder that shipped (planar `radius` around `position` plus a `maxHeight` band, default 6 m); `sphere` is a true 3-D ball and refuses `maxHeight`; `box` takes `position`, `size` (FULL extents) and `rotation` in degrees about the vertical axis; `poly` (alias `polygon`) takes `points` (3..512 planar vertices, concave allowed) with optional `minZ`/`maxZ` -- absent means unbounded on that side; `combo` takes `zones`, nested up to 4 deep. Every shape is CLOSED: a point on an edge, a face or a vertex is inside.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table` | required | — |

Returns: `table (prepared zone), or nil, reason`, `reason: the same tokens as Open77.zones.contains`

```lua
local shop = assert(Open77.zones.normalize({ shape = "box",
    position = { x = -1460.2, y = 99.9, z = 14.8 },
    size = { x = 12.0, y = 6.0, z = 4.0 }, rotation = 37.0 }))
print(shop.shape, shop.bounds.radius)
```

[Proximity zones guide](../../01%20Documentation/zones.md)

Registered in `open77_zones.lua` (line 627) as `LuaPrelude`.

## playersIn

```lua
Open77.zones.playersIn(definition, [options])
```

`NETWORK`: Uses the network backend.

Server only. Every player inside a zone, nearest to its anchor first, honouring routing buckets.

The authoritative half of zones. The client zone service (open77_zones) is PRESENTATION: it decides whether a prompt draws, it runs on a machine the player owns, and it can say anything. Anything that grants -- a shop selling, a job paying, a door opening, a safe zone suppressing damage -- must re-derive containment here first. The geometry is Open77.zones.contains, the same shared file the client embeds, so a definition means the same thing on both sides. Built on Open77.players.nearby and inheriting its conventions exactly: entries are { playerId, bucket, distance, fresh, ageMs, name, position, heading, speed }, ties break by player id so two calls with the same input answer in the same order, and `distance` is to the zone's ANCHOR, not to its boundary. `bucket` defaults to every bucket -- a zone is a place, not a player, and has no bucket of its own to borrow; pass a number to narrow, or false for every bucket explicitly. `limit` applies after containment, never to the bounding-sphere scan. `grace` is metres of slack, and a validation wants a few: Open77.players.position is a replicated snapshot and an honest player's latest position may not have landed yet, so the check exists to stop a client claiming to be somewhere else entirely, not to arbitrate centimetres. `origin` and `rotation` re-site the whole zone, which is how an entity-attached definition is validated server-side. See the Proximity zones guide.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `definition` | `table (a zone definition, or a value from Open77.zones.normalize)` | required | — |
| `options` | `table { bucket?, maxAgeMs?, limit?, grace?, origin?, rotation? }` | optional | — |

Returns: `table (array of nearby-style entries, possibly empty), or nil, reason`, `reason: invalid_options, invalid_limit, any Open77.zones.contains token including origin_required, or a reason from Open77.players.nearby`

```lua
-- Everyone inside the safe zone is healed every ten seconds
CreateThread(function()
    while true do
        Wait(10000)
        for _, entry in ipairs(Open77.zones.playersIn(safeZone) or {}) do
            Open77.players.heal(entry.playerId, 10)
        end
    end
end)
```

[Proximity zones guide](../../01%20Documentation/zones.md)

Registered in `LuaResourceRuntime.cs` (line 8432) as `LuaResourceRuntime.Bootstrap`.
