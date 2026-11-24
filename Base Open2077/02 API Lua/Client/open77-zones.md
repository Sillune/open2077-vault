# Open77.zones (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-zones.md) · [Page web](https://open2077.net/docs/api/client/open77-zones)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

3 functions.

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
local turf = { shape = "poly", minZ = 12.0, maxZ = 20.0, points = {
    { x = -1460.0, y = 90.0 }, { x = -1440.0, y = 90.0 },
    { x = -1440.0, y = 110.0 }, { x = -1460.0, y = 110.0 },
} }
if Open77.zones.contains(turf, Open77.character.state().position) then
    print("standing in the turf")
end
```

[Proximity zones guide](../../01%20Documentation/zones.md)

Registered in `open77_zones.lua` (line 626) as `LuaPrelude`.

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
