# Open77.debugDraw (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-debugdraw.md) · [Page web](https://open2077.net/docs/api/client/open77-debugdraw)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

2 functions.

## clear

```lua
Open77.debugDraw.clear([tag])
```

`GAME`: Requires a live game instance.

Remove one tag's debug geometry, or every batch the calling resource drew.

With a tag, removes that batch; with no argument, removes every batch this resource owns. It never touches another resource's drawings, and it cannot fail: clearing a tag that was never set is a no-op. The runtime does the same on stop and reload, so calling it is only needed to take a drawing down early, before its `ttl` would.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `tag` | `string` | optional | — |

Returns: `true`

```lua
-- Hide the safe-zone outline the moment the admin closes the editor
RegisterCommand("safezone_hide", function()
    Open77.debugDraw.clear("safezone")
end)

-- Or drop everything this resource has drawn
AddEventHandler("onResourceStop", function(name)
    if name == GetCurrentResourceName() then Open77.debugDraw.clear() end
end)
```

Registered in `ResourceHost.cpp` (line 25236) as `LuaDebugGeometryClear`.

## set

```lua
Open77.debugDraw.set(tag, geometry)
```

`GAME`: Requires a live game instance.

Draw resource-owned debug lines and triangles in the world under a tag, replacing that tag atomically.

The overlay PolyZone's editor and `debug` zones use. `tag` (1..96 characters) names a batch owned by the calling resource; setting it again replaces the whole batch. `geometry` is `{ lines = { { a = {x,y,z}, b = {x,y,z}, color = "#RRGGBB" | "#RRGGBBAA" }, ... }, triangles = { { a, b, c, color }, ... }, ttl?, maxDistance?, thickness? }`. Limits: 512 lines and 256 triangles per batch, 32 tags per resource, 128 batches in total, 8192 input vertices in total; coordinates finite and within +/-1,000,000; `ttl` 0.05..10 s (the batch expires on its own, so a resource that stops drawing leaves nothing behind), `maxDistance` 1..5000 m, `thickness` 0.5..8 px. Requires `world.debug`.

Everything a resource drew is released when it stops or reloads. Fails with `invalid_debug_tag`, `invalid_debug_geometry`, `invalid_debug_options`, `invalid_debug_primitives`, `invalid_debug_point`, `invalid_debug_color`, `debug_geometry_quota` or `debug_geometry_unavailable`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `tag` | `string` | required | — |
| `geometry` | `table` | required | — |

Returns: `true, or nil`, `reason on failure`

```lua
-- Outline a 6 m x 6 m safe zone at ground level while an admin tunes it
local cx, cy, cz = Open77.character.position()
local function corner(dx, dy) return { x = cx + dx, y = cy + dy, z = cz + 0.1 } end
local ok, reason = Open77.debugDraw.set("safezone", {
    lines = {
        { a = corner(-3, -3), b = corner(3, -3), color = "#19d5e0" },
        { a = corner(3, -3),  b = corner(3, 3),  color = "#19d5e0" },
        { a = corner(3, 3),   b = corner(-3, 3), color = "#19d5e0" },
        { a = corner(-3, 3),  b = corner(-3, -3), color = "#19d5e0" },
    },
    ttl = 2, maxDistance = 200, thickness = 2,
})
if not ok then print("debugDraw refused: " .. tostring(reason)) end
```

Registered in `ResourceHost.cpp` (line 25235) as `LuaDebugGeometrySet`.
