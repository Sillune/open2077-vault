# Open77.markers (client runtime)

> Source officielle : [Markdown](https://open2077.net/docs/api/client/open77-markers.md) · [Page web](https://open2077.net/docs/api/client/open77-markers)

Runs inside the game process. Reads and presentation are local; anything that changes shared state has to go through the server.

7 functions.

## clear

```lua
Open77.markers.clear()
```

`GAME`: Requires a live game instance.

Removes every 3D marker owned by the calling resource.

Requires `world.markers`; other resources are unaffected.

Returns: `true on success, otherwise false`, `reason`

```lua
-- The race is over: remove every checkpoint marker
AddEventHandler("race:finished", function()
    Open77.markers.clear()
end)
```

[3D world markers guide](../../01%20Documentation/markers.md)

Registered in `ResourceHost.cpp` (line 25493) as `LuaMarkerClear`.

## create

```lua
Open77.markers.create(options)
```

`GAME`: Requires a live game instance.

Creates a resource-owned 3D world marker.

Requires `world.markers`. Options require `position` and accept `shape`, `style`, `radius`, `height`, `scale = {x,y,z}`, `rotation = {x,y,z}` in degrees, `color = {r,g,b,a}`, `maxDistance`, `minDistance`, and `visible`. Color channels are integers from 0 to 255; omitted alpha defaults to 180. Eight native mesh shapes are listed by `shapes()`. The returned decimal-string handle preserves its full 64-bit identity. Creation is asynchronous: inspect `get(id).failed` and `error` for loading failures. Markers are local, resource-owned and cleaned up when their resource stops; they do not provide collision or gameplay authority.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `options` | `table` | required | — |

Returns: `marker handle string, or nil`, `reason`

```lua
-- A cylinder marker at the delivery point, visible within 80 m
local marker = Open77.markers.create({
    position = { x = -1442.2, y = 127.4, z = 18.0 },
    shape = "cylinder",
    radius = 1.5,
    maxDistance = 80,
})
```

[3D world markers guide](../../01%20Documentation/markers.md)

Registered in `ResourceHost.cpp` (line 25490) as `LuaMarkerCreate`.

## get

```lua
Open77.markers.get(id)
```

`GAME`: Requires a live game instance.

Reads one 3D marker owned by the calling resource.

Requires `world.markers`. Returns the same snapshot fields as `list()`, including height, scale, rotation, color, customColor, rendered, failed and optional error. Unknown or foreign handles return `nil, not_found`; a malformed handle returns `nil, invalid_marker_id`. A valid handle can still be loading: rendered does not certify on-screen visibility.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `marker snapshot, or nil`, `reason`

```lua
local state, reason = Open77.markers.get(marker)
if state and state.failed then
    print(state.error)
end
```

[3D world markers guide](../../01%20Documentation/markers.md)

Registered in `ResourceHost.cpp` (line 25495) as `LuaMarkerGet`.

## list

```lua
Open77.markers.list()
```

`GAME`: Requires a live game instance.

Lists the calling resource's 3D markers.

Requires `world.markers`. Each snapshot includes its requested definition, effective color, and native loading state. `rendered` means attached and enabled, not necessarily on-screen or unoccluded. `failed` and optional `error` expose asynchronous asset or streaming failures.

Returns: `array of { id, shape, style, radius, height, scale, rotation, color, customColor, maxDistance, minDistance, visible, rendered, failed, error?, position }, or nil`, `reason`

```lua
-- Debug: are our markers actually rendered?
for _, marker in ipairs(Open77.markers.list() or {}) do
    print(marker.id, marker.rendered)
end
```

[3D world markers guide](../../01%20Documentation/markers.md)

Registered in `ResourceHost.cpp` (line 25494) as `LuaMarkerList`.

## remove

```lua
Open77.markers.remove(id)
```

`GAME`: Requires a live game instance.

Removes one resource-owned 3D marker.

Requires `world.markers`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- The player reached the checkpoint: remove its marker
Open77.markers.remove(checkpointMarker)
```

[3D world markers guide](../../01%20Documentation/markers.md)

Registered in `ResourceHost.cpp` (line 25492) as `LuaMarkerRemove`.

## shapes

```lua
Open77.markers.shapes()
```

`GAME`: Requires a live game instance.

Lists the eight supported native 3D marker shapes.

Returns ring, cylinder, checkpoint, arrow, chevron, cone, diamond and sphere. These are stable string identifiers, not numeric marker types from another game. Reading this static catalogue needs no permission; creating or inspecting markers requires `world.markers`.

Returns: `array of shape names`

```lua
for _, shape in ipairs(Open77.markers.shapes()) do
    print(shape)
end
```

[3D world markers guide](../../01%20Documentation/markers.md)

Registered in `ResourceHost.cpp` (line 25496) as `LuaMarkerShapes`.

## update

```lua
Open77.markers.update(id, patch)
```

`GAME`: Requires a live game instance.

Patches a resource-owned 3D marker.

Requires `world.markers`. Only supplied fields are changed, and ownership prevents mutation of another resource's marker. Accepts the create options, including height, scale, rotation and RGBA color. `color = false` restores the named style palette; alpha zero hides the marker. Invalid patches are rejected atomically. Color and shape changes replace the native entity asynchronously while preserving the handle; avoid per-frame color updates.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `id` | `string` | required | — |
| `patch` | `table` | required | — |

Returns: `true on success, otherwise false`, `reason`

```lua
-- Move the marker to the next checkpoint
RegisterNetEvent("race:checkpoint", function(position)
    Open77.markers.update(checkpointMarker, { position = position })
end)
```

[3D world markers guide](../../01%20Documentation/markers.md)

Registered in `ResourceHost.cpp` (line 25491) as `LuaMarkerUpdate`.
